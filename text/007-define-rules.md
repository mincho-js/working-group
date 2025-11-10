 Start Date: 2025-11-16

# Summary
[summary]: #summary

One paragraph explanation of the feature.

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

    //Entire properties
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
        vars: { [alpha]: '1' },
        background: `rgba(255, 0, 0, ${alpha})`
      }
    },
    backgroundOpacity: {
      1: { vars: { [alpha]: '1' } },
      0.1: { vars: { [alpha]: '0.1' } }
    }
  }
});

// Usage
css({
  background: "red",
  backgroundOpacity: 0.1
});
```

**Compiled:**

```css
.background-red__[HASH] {
  --alpha-[HASH]: 1;
  background: rgba(255, 0, 0, var(--alpha-[HASH]));
}
.backgroundOpacity-0\.1__[HASH] {
  --alpha-[HASH]: 0.1;
}
```

## 4. Functional property

Inspired by [Uno CSS's Dynamic Rules](https://unocss.dev/config/rules#dynamic-rules).

The first level function can return a style object.

```typescript
import { formatHex, parse, lighten, darken, saturate } from 'culori';

const { css } = defineRules({
  properties: {
    background(color: string) {
      const parsed = parse(color);
      return { background: formatHex(parsed) }
    },
    backgroundLight(color: string) {
      const lightened = lighten(parse(color), 0.2);
      return { background: formatHex(lightened) }
    },
    backgroundDark(color: string) {
      const darkened = darken(parse(color), 0.2);
      return { background: formatHex(darkened) }
    },
    backgroundSaturate(color: string) {
      const saturated = saturate(parse(color), 0.3);
      return { background: formatHex(saturated) }
    }
  }
});

// Usage
css({
  background: "blue",           // Normal blue
  backgroundLight: "blue",      // Lightened blue
  backgroundDark: "blue",       // Darkened blue
  backgroundSaturate: "blue"    // More saturated blue
});
```

**Compiled:**

```css
.background-blue__[HASH] { background: #0000ff; }
.backgroundLight-blue__[HASH] { background: #3333ff; }
.backgroundDark-blue__[HASH] { background: #0000cc; }
.backgroundSaturate-blue__[HASH] { background: #0000ff; }
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
      const alignments = value.split('-');
      const [justify, items] = alignments;
      
      return {
        display: 'flex',
        justifyContent: justify,
        alignItems: items || justify
      };
    },

    // Define with fixed value
    inline: { display: "inline" }
  }
});

// Usage
css(["inline", { px: 8 }]);         // { display: 'inline', paddingLeft: 8, paddingRight: 8 }
css({ align: "center" });           // { display: 'flex', justifyContent: 'center', alignItems: 'center' }
css({ align: "center-start" });     // { display: 'flex', justifyContent: 'center', alignItems: 'start' }
css({ align: "space-between-center" }); // { display: 'flex', justifyContent: 'space-between', alignItems: 'center' }
```

**Compiled:**

```css
.display-inline__[HASH] { display: inline; }
.paddingLeft-8__[HASH] { padding-left: 8px; }
.paddingRight-8__[HASH] { padding-right: 8px; }
.display-flex__[HASH] { display: flex; }
.justifyContent-center__[HASH] { justify-content: center; }
.alignItems-center__[HASH] { align-items: center; }
.justifyContent-space\-between__[HASH] { justify-content: space-between; }
```

## 6. Conditions

Inspired by [Vanilla Extract's Conditions](https://vanilla-extract.style/documentation/packages/sprinkles/#conditions).

As with other conditionals, it starts with `_` and is used.  
Both nested and property conditions can be used, as can general conditions.

```typescript
const { css } = defineRules({
  conditions: {
    mobile: {},
    // **or**
    // mobile: {},
    tablet: "@media screen and (min-width: 768px)",
    // **or**
    // tablet: { "@media": "screen and (min-width: 768px)" },
    desktop: "@media screen and (min-width: 1024px)",
    // **or**
    // desktop: { "@media": "screen and (min-width: 1024px)" }
  },
  defaultCondition: 'mobile'
  // etc.
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
  },

  // **or**
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

You can also override Selector like [Panda CSS's Conditional Styles](https://panda-css.com/docs/concepts/conditional-styles).  
`@layer`, `@container`, `@media`, `@supports` and selector should all be enabled.

```typescript
const { css } = defineRules({
  conditions: {
    hover: { selector: "&:is(:hover, [data-hover])": {} },
    focus: { selector: "&:is(:focus, [data-focus])": {} },
    checked: { selector: "&:is(:checked, [data-checked], [aria-checked=true], [data-state='checked'])": {} }
  }
});
```

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

For interaction design, mobile (coarse pointer) users often rely on tap/active and focus-visible states, whereas desktop (fine pointer) users expect hover and focus-within feedback. You can encode those differences declaratively.

**Code:**

```typescript
const { css } = defineRules({
  conditions: {
    mobile: { "@media": "(pointer: coarse)" },
    desktop: { "@media": "(pointer: fine)" },
    hover: { selector: "&:hover" },
    active: { selector: "&:active" },
    focusVisible: { selector: "&:focus-visible" },
    focusWithin: { selector: "&:focus-within" },
    reducedMotion: { "@media": "(prefers-reduced-motion: reduce)" },
    highContrast: { "@media": "(prefers-contrast: more)" },
    forcedColors: { "@media": "(forced-colors: active)" }
  },
  compoundConditions: [
    // Mobile press feedback, but only when the active state and reduced motion preference are both present
    {
      when: ["mobile", "active", "reducedMotion"],
      condition: { "@supports": "(touch-action: manipulation)" }
    },

    // Mobile focus-visible rings that respect high-contrast users
    {
      when: ["mobile", "focusVisible", "highContrast"],
      condition: { selector: "&:not([data-focus-visible=false])" }
    },

    // Desktop hover states that should disappear when the user disables hover hints or prefers reduced motion
    {
      when: ["desktop", "hover", "reducedMotion"],
      condition: { selector: "&:not([data-hover-disable])" }
    },

    // Desktop focus-within outlines inside forced colors mode
    {
      when: ["desktop", "focusWithin", "forcedColors"],
      condition: { selector: "&:not([data-focus-ring=hidden])" }
    },

    // Desktop focus-within rings tuned for high-contrast users
    {
      when: ["desktop", "focusWithin", "highContrast"],
      condition: { "@supports": "(focus-visible: auto)" }
    }
  ]
});
```

**Usage:**

```typescript
css({
  fontSize: 14,
  color: "#1f2329",
  transition: "color 120ms ease, background 120ms ease",

  // Mobile prioritises tap + focus-visible feedback
  _mobile: {
    fontSize: 15,
    padding: 12,

    // Compound condition: mobile + active + reducedMotion
    _active: {
      _reducedMotion: {
        background: "#0f172a",
        color: "#f8fafc",
        transition: "none"
      }
    },

    // Compound condition: mobile + focus-visible + highContrast
    _focusVisible: {
      outline: "2px solid currentColor",
      outlineOffset: 4,

      _highContrast: {
        outlineWidth: 3,
        outlineStyle: "double"
      }
    }
  },

  // Desktop can leverage hover + focus-within
  _desktop: {
    fontSize: 16,
    padding: 10,

    // Compound condition: desktop + hover + reducedMotion
    _hover: {
      background: "#111827",
      color: "#f8fafc",

      _reducedMotion: {
        transition: "none"
      }
    },

    // Compound condition: desktop + focus-within + forced/high contrast
    _focusWithin: {
      boxShadow: "0 0 0 3px rgba(59,130,246,0.5)",

      _forcedColors: {
        outline: "2px solid ButtonText",
        boxShadow: "none"
      },

      _highContrast: {
        boxShadow: "0 0 0 4px currentColor"
      }
    }
  }
});
```

**Compiled:**

```css
.fontSize-14__[HASH] {
  font-size: 14px;
}
.color-\#1f2329__[HASH] {
  color: #1f2329;
}
.transition-color_120ms\ ease\,\ background_120ms\ ease__[HASH] {
  transition: color 120ms ease, background 120ms ease;
}

@media (pointer: coarse) {
  .fontSize-15_mobile__[HASH] {
    font-size: 15px;
  }
  .padding-12_mobile__[HASH] {
    padding: 12px;
  }

  /* Base mobile focus-visible */
  .outline-2px_solid_currentColor_mobile_focusVisible__[HASH]:focus-visible {
    outline: 2px solid currentColor;
  }
  .outlineOffset-4_mobile_focusVisible__[HASH]:focus-visible {
    outline-offset: 4px;
  }
}

/* Compound: mobile + active + reducedMotion */
@supports (touch-action: manipulation) {
  @media (pointer: coarse) and (prefers-reduced-motion: reduce) {
    .background-\#0f172a_mobile_active_reducedMotion__[HASH]:active {
      background: #0f172a;
    }
    .color-\#f8fafc_mobile_active_reducedMotion__[HASH]:active {
      color: #f8fafc;
    }
    .transition-none_mobile_active_reducedMotion__[HASH]:active {
      transition: none;
    }
  }
}

/* Compound: mobile + focus-visible + highContrast */
@media (pointer: coarse) and (prefers-contrast: more) {
  .outline-2px_solid_currentColor_mobile_focusVisible_highContrast__[HASH]:focus-visible:not([data-focus-visible=false]) {
    outline: 2px solid currentColor;
  }
  .outlineWidth-3_mobile_focusVisible_highContrast__[HASH]:focus-visible {
    outline-width: 3px;
  }
  .outlineStyle-double_mobile_focusVisible_highContrast__[HASH]:focus-visible {
    outline-style: double;
  }
}

@media (pointer: fine) {
  .fontSize-16_desktop__[HASH] {
    font-size: 16px;
  }
  .padding-10_desktop__[HASH] {
    padding: 10px;
  }

  /* Base hover/focus-within */
  .background-\#111827_desktop_hover__[HASH]:hover {
    background: #111827;
  }
  .color-\#f8fafc_desktop_hover__[HASH]:hover {
    color: #f8fafc;
  }
  .boxShadow-focusRing_desktop_focusWithin__[HASH]:focus-within {
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.5);
  }
}

/* Compound: desktop + hover + reducedMotion */
@media (pointer: fine) and (prefers-reduced-motion: reduce) {
  .background-\#111827_desktop_hover_reducedMotion__[HASH]:hover:not([data-hover-disable]) {
    background: #111827;
  }
  .color-\#f8fafc_desktop_hover_reducedMotion__[HASH]:hover:not([data-hover-disable]) {
    color: #f8fafc;
  }
  .transition-none_desktop_hover_reducedMotion__[HASH]:hover {
    transition: none;
  }
}

/* Compound: desktop + focus-within + forced colors */
@media (pointer: fine) and (forced-colors: active) {
  .outline-2px_solid_ButtonText_desktop_focusWithin_forced__[HASH]:focus-within:not([data-focus-ring=hidden]) {
    outline: 2px solid ButtonText;
  }
  .boxShadow-none_desktop_focusWithin_forced__[HASH]:focus-within {
    box-shadow: none;
  }
}

/* Compound: desktop + focus-within + high contrast */
@supports (focus-visible: auto) {
  @media (pointer: fine) and (prefers-contrast: more) {
    .boxShadow-0_0_0_4px_desktop_focusWithin_highContrast__[HASH]:focus-within {
      box-shadow: 0 0 0 4px currentColor;
    }
  }
}
```

This allows you to reuse complex condition combinations throughout your styles without repeating the logic, and extend them with additional conditions as needed.

## 9. Class Name Merge

Inspired by [tailwind-merge](https://github.com/dcastil/tailwind-merge).

`defineRules()` ships with a `cx()` helper that understands the metadata produced by the ruleset. Instead of concatenating raw strings, it canonicalises every generated class into a `{ property, condition, slot }` identity and rewrites the list so that the last declaration for a given identity wins. Directional shorthands, compound conditions, and shortcut expansions are all mapped back to the same identity, so `px-8` and `paddingLeft-8` are treated as the same group.

- **Conflict groups from rules.** The merge trie is generated from the declared `properties`, `rules`, and `shortcuts`, so any preset automatically inherits the correct override behavior.
- **Condition awareness.** `_desktop:margin-8` only competes with other `_desktop` margin tokens, while `_mobile` or `hover` variants remain isolated.
- **Zero-cost falsy filtering.** Arrays, objects, and boolean branches follow the same ergonomics as `clsx`, while still taking advantage of the property-aware merging.

The merge pass follows the same $\mathcal{O}(n)$ algorithm as tailwind-merge and caches the canonical token lookups. We reuse their [performance tricks](https://github.com/dcastil/tailwind-merge/blob/v3.4.0/docs/features.md#performance) (prebuilt tries + memoised keys) and extend the schema so that custom presets can register new groups without rebuilding the whole trie at runtime.

```typescript
const { cx, css, rules } = defineRules({
  properties: {
    margin: [0, 4, 8],
    paddingLeft: [8, 16],
    paddingRight: [8, 16],
    color: {
      primary: "#1d4ed8",
      inverse: "#f8fafc"
    }
  },
  shortcuts: {
    px: ["paddingLeft", "paddingRight"]
  }
});

const button = rules({
  intent: {
    primary: css({ background: "#1d4ed8", color: "inverse" }),
    ghost: css({ color: "primary" })
  },
  spacing: {
    sm: "px-8",
    lg: "px-16"
  }
});

const className = cx(
  "margin-8",
  button({ intent: "primary", spacing: "sm" }),
  { "margin-0": true }
);

// => margin-0 wins over margin-8, and px-8 resolves to paddingLeft/paddingRight while deduping duplicates.
```

Because the merge config is derived from the ruleset, preset authors can serialise it alongside their precompiled CSS. Downstream packages can extend the conflict groups (icon utilities, animation helpers, etc.) while reusing the same cache, keeping class composition predictable across package boundaries.

## 10. Class Name Register

Presets can be defined and CSS extraction can occur at the library level.

```typescript
// Package A: @design-system/core
export const presets = defineRules({
  /* ... */
});
```

```typescript
// Package B: @design-system/components
import { presets } from "@design-system/core";

export const componentStyle = presets.css({
  /* ... */
});
```

```typescript
// Application
import { presets } from "@design-system/core";
import { MyComponent } from "@design-system/components";
import "@design-system/components/styles.css"; // Precompiled CSS

const appStyle = presets.css({
  /* ... */
});
```

To avoid creating duplicate class names and styles, you need to register class names with Vanilla Extract, and provide the store objects created so far to the preset object.

```typescript
import { presets } from "@design-system/core";
import { MyComponent } from "@design-system/components";
import "@design-system/components/styles.css"; // Precompiled CSS

presets.registerClassNames();
```

## 11. Theme Specification

Inspired by [Theme UI Specification](https://theme-ui.com/theme-spec).

Design tokens only help when rules know which CSS properties they can satisfy. `defineRules.themeSpec()` takes the object returned from `theme()` and emits a property constraint map that slots neatly into `defineRules({ properties })`.

**Code:**

```ts
const [themeClass, themeVars] = theme({
  colors: {
    text: { default: "#0f172a", muted: "#475569" },
    surface: { canvas: "#ffffff", brand: "#2563eb" }
  },
  space: [0, 4, 8, 12, 16, 20]
});

const customSpacingVariable = createVar();

const tokens = defineRules.themeSpec([
  // The values ​​of the sub-objects of the theme are registered.
  { condition: [themeVars.colors], properties: ["color", "backgroundColor", "borderColor"] },
  // Custom Variables are also possible.
  { condition: [themeVars.space, customSpacingVariable], properties: ["gap", "paddingInlineStart", "paddingInlineEnd"] }
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

For this to work, theme vars and createVar must be of type brand.

## 12. Theme context

`defineRules()` can hydrate the theme bag returned from Vanilla Extract's `theme()` helper and thread it through every helper it creates. Once the `theme` option is provided, `css`, `rules`, `cva`, and even shortcut handlers can accept a function instead of a plain object. Each helper receives the strongly typed `themeVars` tree exactly once—`rules` and `cva` do this at the root of their configuration, so every nested branch can close over the same token vocabulary without re-invoking the callback.

**Code:**

```ts
import { theme } from "@vanilla-extract/css";

const [themeClass, themeVars] = theme({
  colors: {
    text: { base: "#0f172a", inverse: "#f8fafc" },
    surface: { canvas: "#ffffff", brand: "#2563eb" }
  },
  space: { xs: "0.375rem", sm: "0.75rem", md: "1rem" },
  radii: { sm: "8px", pill: "999px" }
});

const { css, rules, cx } = defineRules({
  theme: themeVars,
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

export const cardClass = cx(
  themeClass,
  card,
  button(["pill", { intent: "primary", spacing: "relaxed" }])
);
```

## 13. Shortcuts Spec

Shortcuts can be packaged once and re-used across projects. `defineRules.shortcutPreset()` collects a dictionary of shortcut handlers, tags them with metadata, and exposes a typed object that can be spread into any ruleset. The helper is generic over the shape of `properties`; if you omit the generic, it defaults to `CSSProperties`, so the handlers always know which CSS props are safe to emit.

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

  // Functional shortcut -> can read the declared properties
  gutter(value: number | string) {
    const size = Number(value);
    return {
      gap: properties.gap ?? size,
      paddingInlineStart: size,
      paddingInlineEnd: size
    } satisfies SpacingProps;
  }
});

const motionShortcuts = defineRules.shortcutSpec({
  transition(value: string) {
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
  { transition: "background 160ms ease" }
]);
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

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
  defaultCondition: "mobile"
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
  defaultCondition: "mobile",
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
