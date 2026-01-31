- Start Date: 2025-11-16

# Summary
[summary]: #summary

`defineRules()` creates a typed styling API from property constraints, shortcuts, and named conditions.

Its `css()` helper generates atomic CSS on demand at build time, while `rules()` defines reusable recipes and scoped `cx()` resolves conflicts between registered atomic classes.

Presets share generated declarations across packages, and `propertyValues()` maps shared values to CSS properties.

The browser loads the extracted CSS and selects or merges class names.

# Motivation
[motivation]: #motivation

## Atomic CSS

The basic idea behind [sprinkles](https://vanilla-extract.style/documentation/packages/sprinkles/) is good.

But various issues([#91](https://github.com/vanilla-extract-css/vanilla-extract/discussions/91), [#992](https://github.com/vanilla-extract-css/vanilla-extract/discussions/992), [#1132](https://github.com/vanilla-extract-css/vanilla-extract/discussions/1132), [#1237](https://github.com/vanilla-extract-css/vanilla-extract/discussions/1237)) with Vanilla Extract, it doesn't seem easy.

![sprinkle generated css](https://github.com/user-attachments/assets/70e1c800-913f-48d6-bb22-b929c80c25be)

Solving this problem requires an [on-demand](https://antfu.me/posts/reimagine-atomic-css) approach like uno css.

## Style for design system

Consistent styles created for a design system should be easily reusable across each application.

[Tailwind](https://tailwindcss.com/) is gaining huge popularity by building reusable and collocated design system presets.

It should provide a more reusable style system, and be structured to work well across different packages.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## 1. Name

The API is named `defineRules()`.

High order definitions for `cx`, `cva`, `css`, and `rules`.

## 2. Property constraints

Inspired by [Vanilla Extract's Sprinkles](https://vanilla-extract.style/documentation/packages/sprinkles/).

Constrains the CSS properties and values ​​that can be used.  

**Code:**

```typescript
const { cx, cva, css, rules } = defineRules({
  properties: {
    // Allow only values in arrays
    display: ["none", "inline"],
    paddingLeft: [0, 2, 4, 8, 16, 32, 64],
    paddingRight: [0, 2, 4, 8, 16, 32, 64],

    // Allow only values in objects
    color: { "indigo-800": "rgb(55, 48, 163)" },

    // Entire properties
    background: true,
    border: false
  }
});

// Usage
css({
  display: "inline", // Only these values allowed
  paddingLeft: 8,    // Only these values allowed
  color: "indigo-800", // Only these values allowed
  background: "red"   // All values allowed
});
```

Each property generates its own class name.

**Compiled:**

```css
.display-inline__[HASH] { display: inline; }
.paddingLeft-8__[HASH] { padding-left: 8px; }
.color-indigo-800__[HASH] { color: rgb(55, 48, 163); }
.background-red__[HASH] { background: red; }
```

## 3. Style object property

Inspired by [Vanilla Extract's Sprinkles](https://vanilla-extract.style/documentation/packages/sprinkles/#properties).

The second level object represents a style object.

```typescript
const alpha = createVar();

const { css } = defineRules({
  properties: {
    background: {
      red: {
        vars: { [alpha]: "1" },
        background: `rgba(255, 0, 0, ${alpha})`
      }
    },
    backgroundOpacity: {
      1: { vars: { [alpha]: "1" } },
      0.1: { vars: { [alpha]: "0.1" } }
    }
  }
});

// Usage
css({
  background: "red",
  backgroundOpacity: 0.1
});
```

Both inputs write the same custom property. Its last value, `0.1`, wins, while the background declaration remains. Each becomes a separate atomic class.

**Compiled:**

```css
.background-red__[HASH] {
  background: rgba(255, 0, 0, var(--alpha-[HASH]));
}
.alpha__[HASH] {
  --alpha-[HASH]: 0.1;
}
```

## 4. Functional property

Inspired by [Uno CSS's Dynamic Rules](https://unocss.dev/config/rules#dynamic-rules).

The first level function can return a style object.

```typescript
import { colord } from "colord";

const { css } = defineRules({
  properties: {
    background(color: string) {
      return { background: colord(color).toHex() };
    },
    backgroundLight(color: string) {
      return { background: colord(color).lighten(0.2).toHex() };
    },
    backgroundDark(color: string) {
      return { background: colord(color).darken(0.2).toHex() };
    },
    backgroundSaturate(color: string) {
      return { background: colord(color).saturate(0.3).toHex() };
    }
  }
});

// Usage
css({ background: "#6699cc" });         // Normal blue
css({ backgroundLight: "#6699cc" });    // Lightened blue
css({ backgroundDark: "#6699cc" });     // Darkened blue
css({ backgroundSaturate: "#6699cc" }); // More saturated blue
```

These calls use the same base color but compute different `background` values.

**Compiled:**

```css
.background-\#6699cc__[HASH] { background: #6699cc; }
.backgroundLight-\#6699cc__[HASH] { background: #b3cce6; }
.backgroundDark-\#6699cc__[HASH] { background: #336699; }
.backgroundSaturate-\#6699cc__[HASH] { background: #4799eb; }
```

## 5. Shortcuts

Inspired by [Vanilla Extract's Shorthands](https://vanilla-extract.style/documentation/packages/sprinkles/#shorthands), [Uno CSS's Shortcuts](https://unocss.dev/config/shortcuts).

Using shortcuts automatically derives properties, making Atomic CSS easy to achieve.
It provides dynamic rules like uno as well as fixed shortcuts, giving you an experience similar to writing tailwind.

```typescript
const { cx, css, rules } = defineRules({
  properties: {
    display: ["none", "inline", "flex"],
    paddingLeft: [0, 2, 4, 8, 16, 32, 64],
    paddingRight: [0, 2, 4, 8, 16, 32, 64],
    justifyContent: true,
    alignItems: true
  },
  shortcuts: {
    // Define only property
    pl: "paddingLeft",
    pr: "paddingRight",

    // Define multiple properties
    px: ["pl", "pr"],

    // Define with functional value
    align(value: string) {
      const alignments = value.match(
        /^(space-between|center|start|end)(?:-(center|start|end))?$/
      );
      if (!alignments) throw new Error(`Invalid alignment: ${value}`);
      const [, justify, items] = alignments;

      return {
        display: "flex",
        justifyContent: justify,
        alignItems: items ?? (justify === "space-between" ? "center" : justify)
      } as const;
    },

    // Define with fixed value
    inline: { display: "inline" }
  }
});

// Usage (comments show expanded declarations)
css(["inline", { px: 8 }]);         // { display: "inline", paddingLeft: 8, paddingRight: 8 }
css({ align: "center" });           // { display: "flex", justifyContent: "center", alignItems: "center" }
css({ align: "center-start" });     // { display: "flex", justifyContent: "center", alignItems: "start" }
css({ align: "space-between-center" }); // { display: "flex", justifyContent: "space-between", alignItems: "center" }
```

The functional shortcut parses Tailwind-like syntax such as `center-start` and `space-between-center` into CSS properties.

**Compiled:**

```css
.display-inline__[HASH] { display: inline; }
.paddingLeft-8__[HASH] { padding-left: 8px; }
.paddingRight-8__[HASH] { padding-right: 8px; }
.display-flex__[HASH] { display: flex; }
.justifyContent-center__[HASH] { justify-content: center; }
.alignItems-center__[HASH] { align-items: center; }
.alignItems-start__[HASH] { align-items: start; }
.justifyContent-space\-between__[HASH] { justify-content: space-between; }
```

## 6. Conditions

Inspired by [Vanilla Extract's Conditions](https://vanilla-extract.style/documentation/packages/sprinkles/#conditions).

As with other conditionals, it starts with `_` and is used.  
Both nested and property conditions can be used, as can general conditions.
An empty condition object names the base style without adding a media query.

```typescript
const { css } = defineRules({
  conditions: {
    mobile: {},
    tablet: "@media screen and (min-width: 768px)",
    // **or**
    // tablet: { "@media": "screen and (min-width: 768px)" },
    desktop: "@media screen and (min-width: 1024px)"
    // **or**
    // desktop: { "@media": "screen and (min-width: 1024px)" }
  },
  properties: {
    fontSize: [12, 16, 20]
  }
});

// Usage
css({
  _mobile: {
    fontSize: 12
  },
  _tablet: {
    fontSize: 16
  },
  _desktop: {
    fontSize: 20
  }
});

// Equivalent property-level form; reuses the same atomic classes.
css({
  fontSize: {
    _mobile: 12,
    _tablet: 16,
    _desktop: 20
  }
});
```

**Compiled:**

```css
.fontSize-12_mobile__[HASH] { font-size: 12px; }

@media screen and (min-width: 768px) {
  .fontSize-16_tablet__[HASH] { font-size: 16px; }
}

@media screen and (min-width: 1024px) {
  .fontSize-20_desktop__[HASH] { font-size: 20px; }
}
```

You can also define selector conditions like [Panda CSS's Conditional Styles](https://panda-css.com/docs/concepts/conditional-styles).

Condition objects support `@layer`, `@container`, `@media`, `@supports`, and `selector`.  
Use a distinct name when a built-in alias such as `_hover` already exists.

```typescript
const { css } = defineRules({
  conditions: {
    interactiveHover: { selector: "&:is(:hover, [data-hover])" },
    interactiveFocus: { selector: "&:is(:focus, [data-focus])" },
    checked: { selector: "&:is(:checked, [data-checked], [aria-checked=true], [data-state='checked'])" }
  }
});
```

### Default Condition

`defaultCondition` selects the condition for declarations written without an explicit condition.  
A print-specific ruleset can serve reports, invoices, and toolbars without repeating `_print` in every style.

```typescript
const printRules = defineRules({
  conditions: {
    print: { "@media": "print" }
  },
  defaultCondition: "print",
  properties: {
    color: true,
    display: ["none", "block"]
  }
});

const reportText = printRules.css({ color: "black" });
const toolbar = printRules.css({ display: "none" });
```

**Compiled:**

```css
@media print {
  .color-black_print__[HASH] { color: black; }
  .display-none_print__[HASH] { display: none; }
}
```

Both calls produce print-only declarations.  
Explicitly named conditions keep their own target.

The same pattern can provide a default scope for theme-specific styles or motion styles under `prefers-reduced-motion: no-preference`.

## 7. Composite Condition

You can also create complex conditions.

You can use multiple conditions at once or use composite rules.

```typescript
const { css } = defineRules({
  conditions: {
    // Multiple conditions
    dark: {
      selector: ".dark &",
      "@media": "(forced-colors: active)"
    },

    // Composite conditions
    highContrast: {
      "@media": ["(prefers-contrast: more)", "(forced-colors: active)"]
    }
  }
});
```

## 8. Compound Condition

Similar to [compound variants in rules](./002-css-rules.md#6-compound-variants), you can create new conditions by combining existing conditions.  
This is useful when you need to define a condition that applies only when multiple conditions are met simultaneously.

`compoundConditions` adds its condition when the named conditions appear together in a style's condition path.

For example, buttons and menu items can share a policy that enables hover feedback only on enabled elements when the primary pointer supports hover and fine pointing.  
Each component still chooses its own feedback styles.

**Code:**

```typescript
const interaction = defineRules({
  properties: {
    backgroundColor: true,
    outline: true
  },
  conditions: {
    enabled: { selector: "&:not(:disabled):not([aria-disabled=true])" },
    interactiveHover: { selector: "&:hover" },
    focusVisible: { selector: "&:focus-visible" }
  },
  compoundConditions: [
    {
      when: ["enabled", "interactiveHover"],
      condition: {
        "@media": "(hover: hover) and (pointer: fine)"
      }
    }
  ]
});
```

**Usage:**

```typescript
const button = interaction.css({
  _enabled: {
    _focusVisible: {
      outline: "2px solid #2563eb"
    },
    _interactiveHover: {
      backgroundColor: "#1d4ed8"
    }
  }
});

const menuItem = interaction.css({
  _enabled: {
    _interactiveHover: {
      backgroundColor: "#e2e8f0"
    }
  }
});
```

**Compiled:**

```css
.buttonFocus__[HASH]:not(:disabled):not([aria-disabled=true]):focus-visible {
  outline: 2px solid #2563eb;
}

@media (hover: hover) and (pointer: fine) {
  .buttonHover__[HASH]:not(:disabled):not([aria-disabled=true]):hover {
    background-color: #1d4ed8;
  }
  .menuItemHover__[HASH]:not(:disabled):not([aria-disabled=true]):hover {
    background-color: #e2e8f0;
  }
}
```

The extra media query applies to declarations under both `enabled` and `interactiveHover`.  
The focus branch uses a different combination, so its outline remains available across pointer types.

Declarations in parent conditions stay under their original conditions.

A shared ruleset can update this hover policy for every matching style without editing each component.  
Nested or composite conditions remain sufficient for combinations that do not need an automatically applied policy.

## 9. Class Name Merge

Inspired by [tailwind-merge](https://github.com/dcastil/tailwind-merge).

`defineRules()` returns a scoped `cx()` that uses metadata recorded for generated atomic classes.

When two registered classes write the same property under the same normalized conditions, the last input wins.  
Shortcuts are expanded before registration, so each resulting property can be overridden independently.

```typescript
const { cx, css, rules } = defineRules({
  properties: {
    paddingLeft: [8, 16],
    paddingRight: [8, 16],
    backgroundColor: true,
    color: {
      primary: "#1d4ed8",
      inverse: "#f8fafc"
    }
  },
  shortcuts: {
    px: ["paddingLeft", "paddingRight"]
  },
  conditions: {
    desktop: { "@media": "(min-width: 1024px)" }
  }
});

const button = rules({
  variants: {
    intent: {
      primary: { backgroundColor: "#1d4ed8", color: "inverse" },
      ghost: { color: "primary" }
    }
  }
});

const base = css({ px: 8 });
const widerLeft = css({ paddingLeft: 16 });
const desktopLeft = css({ _desktop: { paddingLeft: 16 } });

const className = cx(
  button({ intent: "primary" }),
  base,
  [widerLeft, false],
  desktopLeft,
  { external: true }
);

// Keeps paddingLeft: 16 and paddingRight: 8 in the base condition,
// the separate desktop declaration, the recipe classes, and the external class.

cx(widerLeft, base); // Reversing the inputs keeps paddingLeft: 8.
```

Inputs follow `clsx` conventions for arrays, conditional objects, and falsy values.

Unknown classes are preserved, including their repetitions and relative order.  
Writing a string such as `"paddingLeft-8"` does not register it.

Grouped recipe classes without atomic metadata are also preserved as opaque tokens.

Conditions match exactly after normalization of `layer`, `supports`, `media`, `container`, and `selector`.  
The merge does not infer overlapping media queries, selector equivalence, or precedence between layers.

It also treats CSS shorthands and longhands, such as `padding` and `paddingLeft`, as different properties, and does not give `!important` special priority.

The root `cx` export joins inputs without this scoped conflict resolution.  
Compiled scoped `cx()` calls use class-write metadata; they do not generate CSS at runtime.

## 10. Class Name Register

Scoped `css()` records each emitted atomic class.  
The `.preset` property captures an immutable snapshot of those declarations and inherited presets.

Another `defineRules()` instance can consume it through `presets` to reuse classes for matching properties, values, and conditions, and to include their metadata in its scoped `cx()`.

Generate shared styles before exporting the snapshot.  
The consumer still declares its own property constraints, shortcuts, and condition names.

```typescript
// Package A: shared.css.ts, exporting sharedPreset through ./preset
import { defineRules } from "@mincho-js/css";

const shared = defineRules({
  properties: { color: true }
});

const brandText = shared.css({ color: "#1d4ed8" });
export const sharedPreset = shared.preset;
```

```typescript
// Package B: button.css.ts, exporting componentPreset through ./preset
import { defineRules } from "@mincho-js/css";
import { sharedPreset } from "@design-system/core/preset";

const component = defineRules({
  properties: { color: true, paddingLeft: [8, 16] },
  presets: sharedPreset
});

const buttonClass = component.css({
  color: "#1d4ed8", // Reuses Package A's class and CSS.
  paddingLeft: 8    // Emits a new class in Package B.
});

export const componentPreset = component.preset;
```

Reading `.preset`, including through destructuring, captures the current state.  
Later styles in the producer do not update an existing snapshot or consumer.

To consume those additions, construct a new ruleset with the newer snapshot.  
Combining different revisions of the same producer is rejected.

Multiple presets can be supplied as an array.  
Their ancestors are visited first, in parent order, and each shared ancestor is applied once.

When independently generated classes represent the same declaration, the later visited producer supplies the class reused by subsequent authoring.  
Existing classes remain valid and registered for scoped `cx()`.

This preset order is separate from the input order used by `cx()` to resolve conflicting values.

Registration with Vanilla Extract happens during CSS extraction.

Published packages expose runtime exports through `.`, authoring snapshots through `./preset`, and precompiled CSS through `./style.css`.  
Package authors configure these exports and preserve stylesheet imports with `sideEffects`.

```typescript
// Application entry
import "@design-system/core/style.css";
import "@design-system/components/style.css";
import { Button } from "@design-system/components";
```

Preset composition reuses known declarations in later builds; it does not remove CSS already emitted independently by other packages.  
Load the stylesheets of every producer whose classes are used, and publish matching JavaScript, preset, and CSS outputs together.

## 11. Property Values

Inspired by [Theme UI Specification](https://theme-ui.com/theme-spec).

`defineRules.propertyValues()` assigns one value source to several CSS properties.

Each entry has a `source` and a list of target `properties`; the result can be passed to `defineRules({ properties })` or spread alongside other constraints.  
It works with ordinary values as well as theme variables.

**Code:**

```ts
import { createVar, defineRules, theme } from "@mincho-js/css";

const [themeClass, themeVars] = theme({
  colors: {
    text: { default: "#0f172a", muted: "#475569" },
    surface: { canvas: "#ffffff", brand: "#2563eb" }
  },
  space: ["0px", "4px", "8px", "12px", "16px", "20px"]
});

const customSpacingVariable = createVar();

const tokens = defineRules.propertyValues([
  // The selected values of the theme are registered.
  {
    source: [
      themeVars.colors.text.default,
      themeVars.colors.text.muted,
      themeVars.colors.surface.canvas,
      themeVars.colors.surface.brand
    ],
    properties: ["color", "backgroundColor", "borderColor"]
  },
  // Custom variables are also possible.
  {
    source: [...themeVars.space, customSpacingVariable],
    properties: ["gap", "paddingInlineStart", "paddingInlineEnd"]
  }
]);

const { css } = defineRules({
  conditions: {
    mobile: { "@media": "(max-width: 768px)" },
    desktop: { "@media": "(min-width: 769px)" }
  },
  properties: {
    display: ["inline-flex", "flex"],
    ...tokens
  }
});

const button = css({
  color: themeVars.colors.text.muted,
  backgroundColor: themeVars.colors.surface.brand,
  gap: themeVars.space[1],
  _mobile: {
    paddingInlineStart: themeVars.space[3],
    paddingInlineEnd: themeVars.space[2]
  }
});
```

The `tokens` object shape:

```typescript
{
  color: [themeVars.colors.text.default, themeVars.colors.text.muted, themeVars.colors.surface.canvas, themeVars.colors.surface.brand],
  backgroundColor: [themeVars.colors.text.default, themeVars.colors.text.muted, themeVars.colors.surface.canvas, themeVars.colors.surface.brand],
  borderColor: [themeVars.colors.text.default, themeVars.colors.text.muted, themeVars.colors.surface.canvas, themeVars.colors.surface.brand],
  gap: [themeVars.space[0], themeVars.space[1], themeVars.space[2], themeVars.space[3], themeVars.space[4], themeVars.space[5], customSpacingVariable],
  paddingInlineStart: [themeVars.space[0], themeVars.space[1], themeVars.space[2], themeVars.space[3], themeVars.space[4], themeVars.space[5], customSpacingVariable],
  paddingInlineEnd: [themeVars.space[0], themeVars.space[1], themeVars.space[2], themeVars.space[3], themeVars.space[4], themeVars.space[5], customSpacingVariable]
}
```

An array source accepts the listed values directly, as in the example above.  
An object source instead maps names to values:

```typescript
const { css: namedCss } = defineRules({
  properties: defineRules.propertyValues([
    {
      source: { muted: themeVars.colors.text.muted },
      properties: ["color", "backgroundColor"]
    }
  ])
});

namedCss({ color: "muted" });
```

The helper assigns each source as-is.  
It does not recursively flatten a theme tree or infer target properties from token names.

Select nested leaves explicitly, and use `[...themeVars.space, customSpacingVariable]` to combine array values.

If several entries target the same property, the last source replaces the earlier source.

## 12. Theme context

`defineRules()` can hydrate the theme bag returned from the `theme()` helper and thread it through every helper it creates.

Once the `context` option is provided, `css`, `rules`, `cva`, and even shortcut handlers can accept a function instead of a plain object.

Each helper receives the strongly typed `themeVars` tree exactly once—`rules` and `cva` do this at the root of their configuration, so every nested branch can close over the same token vocabulary without re-invoking the callback.

**Code:**

```ts
import { theme } from "@mincho-js/css";

const [themeClass, themeVars] = theme({
  colors: {
    text: { base: "#0f172a", inverse: "#f8fafc" },
    surface: { canvas: "#ffffff", brand: "#2563eb" }
  },
  space: { xs: "0.375rem", sm: "0.75rem", md: "1rem" },
  radii: { sm: "8px", pill: "999px" }
});

const { css, rules, cx } = defineRules({
  context: themeVars,
  conditions: {
    mobile: { "@media": "(max-width: 767px)" },
    desktop: { "@media": "(min-width: 768px)" }
  },
  properties: {
    color: true,
    backgroundColor: true,
    gap: true,
    borderRadius: true,
    transition: true
  }
});

const card = css((theme) => ({
  color: theme.colors.text.base,
  backgroundColor: theme.colors.surface.canvas,
  borderRadius: theme.radii.sm,
  gap: theme.space.xs,
  transition: "color 150ms ease, background 150ms ease",
  _desktop: {
    gap: theme.space.md,
    backgroundColor: theme.colors.surface.brand,
    color: theme.colors.text.inverse
  }
}));

const button = rules((theme) => ({
  base: {
    borderRadius: theme.radii.sm,
    transition: "background 160ms ease, color 160ms ease"
  },
  variants: {
    intent: {
      primary: {
        color: theme.colors.text.inverse,
        backgroundColor: theme.colors.surface.brand
      },
      subtle: {
        color: theme.colors.surface.brand
      }
    },
    spacing: {
      snug: { gap: theme.space.xs },
      relaxed: { gap: theme.space.sm }
    }
  },
  toggles: {
    pill: { borderRadius: theme.radii.pill }
  }
}));

const cardClass = cx(
  themeClass,
  card,
  button(["pill", { intent: "primary", spacing: "relaxed" }])
);
```

## 13. Shortcuts Spec

Shortcuts can be packaged once and re-used across projects.

`defineRules.shortcutSpec()` collects a dictionary of shortcut handlers, tags them with metadata, and exposes a typed object that can be spread into any ruleset.

The helper is generic over the shape of `properties`; if you omit the generic, it defaults to `CSSProperties`, so the handlers always know which CSS props are safe to emit.

**Code:**

```typescript
import type { CSSProperties } from "@mincho-js/css";

type SpacingProps = Pick<CSSProperties, "gap" | "paddingInlineStart" | "paddingInlineEnd">;

const spacingShortcuts = defineRules.shortcutSpec<SpacingProps>({
  // Plain alias -> property
  pl: "paddingInlineStart",
  pr: "paddingInlineEnd",

  // Multi-alias -> expands recursively (px -> pl/pr)
  px: ["pl", "pr"],

  // Functional shortcut -> computes several property values
  gutter(value: number | string) {
    const size = Number(value);
    return {
      gap: size,
      paddingInlineStart: size,
      paddingInlineEnd: size
    } satisfies SpacingProps;
  }
});

const motionShortcuts = defineRules.shortcutSpec({
  motion(value: string) {
    return { transition: value || "color 150ms ease" } satisfies CSSProperties;
  },
  focusRing(color: string) {
    return {
      outline: `2px solid ${color}`,
      outlineOffset: 4
    } satisfies Pick<CSSProperties, "outline" | "outlineOffset">;
  }
});

const { css } = defineRules({
  properties: {
    gap: [0, 4, 8, 12, 16],
    paddingInlineStart: [0, 4, 8, 12, 16],
    paddingInlineEnd: [0, 4, 8, 12, 16],
    transition: true,
    outline: true,
    outlineOffset: true
  },
  shortcuts: {
    ...spacingShortcuts,
    ...motionShortcuts,
    inline: { display: "inline-flex", alignItems: "center" }
  }
});

const pill = css([
  "inline",
  { px: 8 },
  { gutter: "12" },
  { focusRing: "#2563eb" },
  { motion: "background 160ms ease" }
]);
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Input resolution

`defineRules()` stores property constraints, shortcuts, conditions, and optional authoring context.

Arrays describe allowed property values; object maps translate input keys to values or style objects.  
Functions can compute those values during authoring.

`propertyValues()` only builds this constraint map; it emits no CSS.

Style inputs are resolved in array order and object property order.  
Shortcuts expand recursively, property resolvers produce CSS declarations, and named conditions add their media queries or selectors.

Cyclic shortcuts and invalid or reserved condition names are errors.

`css.raw()` returns the resolved style object; `css()` generates classes for its declarations.

## Atomic output and merging

A conflict is identified by the normalized condition tuple—layer, supports, media, container, and selector—and the expanded CSS property.  
The last write to that pair survives within a call.

Custom properties are separate declarations, so the background-opacity example keeps the background and only the final alpha value.

CSS shorthand relationships and overlapping media queries are left to the browser.

Reuse additionally includes the declaration's value: an equivalent declaration reuses an existing atomic class.  
Earlier calls may already have emitted other CSS; a later override does not remove that stylesheet output.

Scoped `cx()` uses the same conflict identity to merge registered classes.  
Grouped recipe classes from `rules()` remain opaque to this atomic merge.

## Build and runtime boundary

Authoring runs in a `.css.ts` file or through the [co-location extraction flow](./005-co-location.md).  
The build evaluates style callbacks and emits CSS, class names, and the metadata needed by scoped helpers.

The browser loads that CSS and runs class selection or merging without evaluating the property resolvers again.

Local authoring can use function-valued properties and shortcuts.  
Exporting an authoring factory through extraction also requires serializable configuration and context; those function-valued definitions cannot cross that boundary.

A `preset` snapshot contains generated atomic metadata, so consumers still declare their own property constraints and shortcuts.

# Drawbacks
[drawbacks]: #drawbacks

This requires a really **complex type definition** that goes beyond what has been required so far.

If implementation is not possible, it may need to be removed from the existing design.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

## Condition ClassNames

Class names can be generated in two ways depending on conditions.

### Variable ClassNames (Default)

Each condition generates a unique class name. This approach is similar to Atomic CSS frameworks.

**Code:**

```typescript
const { css } = defineRules({
  conditions: {
    mobile: {},
    tablet: "@media screen and (min-width: 768px)",
    desktop: "@media screen and (min-width: 1024px)"
  },
  properties: {
    fontSize: [12, 16, 20]
  }
});

css({
  fontSize: {
    _mobile: 12,
    _tablet: 16,
    _desktop: 20
  }
});
```

**Compiled:**

```css
.fontSize-12_mobile__[HASH] { font-size: 12px; }

@media screen and (min-width: 768px) {
  .fontSize-16_tablet__[HASH] { font-size: 16px; }
}

@media screen and (min-width: 1024px) {
  .fontSize-20_desktop__[HASH] { font-size: 20px; }
}
```

**HTML:**

```html
<div class="fontSize-12_mobile__[HASH] fontSize-16_tablet__[HASH] fontSize-20_desktop__[HASH]">
```

This is the most atomic and cacheable approach. Each value gets its own class.

### Stable ClassNames

All conditions share the same class name. This approach reduces class name duplication in HTML.

**Code:**

```typescript
const { css } = defineRules({
  conditions: {
    mobile: {},
    tablet: "@media screen and (min-width: 768px)",
    desktop: "@media screen and (min-width: 1024px)"
  },
  properties: {
    fontSize: [12, 16, 20]
  },
  // Proposed alternative to the atomic output above.
  stableClassNames: true
});

css({
  fontSize: {
    _mobile: 12,
    _tablet: 16,
    _desktop: 20
  }
});
```

**Compiled:**

```css
.fontSize__[HASH] {
  font-size: 12px;
}

@media screen and (min-width: 768px) {
  .fontSize__[HASH] {
    font-size: 16px;
  }
}

@media screen and (min-width: 1024px) {
  .fontSize__[HASH] {
    font-size: 20px;
  }
}
```

**HTML:**

```html
<div class="fontSize__[HASH]">
```

### Comparison with Tailwind

Tailwind uses prefixed class names for each condition:

```html
<!-- Tailwind -->
<div class="text-xs md:text-base lg:text-xl">
```

Each responsive variant is a separate class. This is similar to our variable class name approach.

Our stable class name approach differs by bundling all conditions into one class, reducing HTML size but limiting atomic reusability.

### Trade-offs

**Variable ClassNames:**
- ✅ Maximum atomicity and cache efficiency
- ✅ Can reuse individual condition classes across components
- ❌ More class names in HTML

**Stable ClassNames:**
- ✅ Cleaner HTML with fewer class names
- ✅ Smaller HTML payload
- ❌ Less atomic, harder to share partial styles
- ❌ Generates more CSS when the same property has different condition combinations

Choose variable class names for design systems with high reusability. Choose stable class names for applications prioritizing HTML size.


# Unresolved questions
[unresolved-questions]: #unresolved-questions

## Complex Type Definition

The type definition required to implement this is quite complex.

If implementation is not possible, it may need to be removed from the existing design.

# Future possibilities
[future-possibilities]: #future-possibilities

## Presets

First, we port the presets for commonly used elements.
- [Tailwind CSS](https://tailwindcss.com/)
- [DaisyUI](https://daisyui.com/)

At this point, it may be necessary to implement a Theme spec to facilitate customization.
- [Theme UI Specification](https://theme-ui.com/theme-spec)
- [System UI Theme Specification](https://github.com/system-ui/theme-specification)

Then port the presets for special cases like [Uno's icons](https://unocss.dev/presets/icons) or compose presets with better values.

- [AdorableCSS](https://developer-1px.github.io/adorable-css/): Preset that matches Figma's mental model and is more intuitive.
- [Elm UI](https://package.elm-lang.org/packages/mdgriffith/elm-ui/latest/): Presets designed to suppress CSS layout errors.
