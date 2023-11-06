- Start Date: 2023-11-05

# Summary

Suggestions for expressing more natural CSS in Typescript.

# Motivation

- Take advantage of preprocessors
- Finds as many alternatives as possible within the parsing syntax of Javascript
- Be as type-safe and autocomplete as possible

# Guide-level explanation

The following syntax is based on [Vanilla Extract](https://vanilla-extract.style/).

To make the scope of the task more clear, I've written examples of converted code where conversion to `Vanilla Extract` is required.  
I also include the compiled results to help you understand.

## 1. Name

The API is named `css()`.

This is because it's short, easy to type, and tries to present a similar experience to writing regular CSS.

Same as [`style()`](https://vanilla-extract.style/documentation/api/style/) in Vanilla Extract, and returns className.

## 2. Object Styles

As mentioned in [Emotion's best practices](https://emotion.sh/docs/best-practices#use-typescript-and-object-styles), To benefit from proper intelligence and type checking from the editor, it must be object-style based.

```typescript
const myCss = css({
  color: "blue",
  backgroundColor: 100 // Error: Type 'number' is not assignable to type 'Color | undefined'
});
```

## 3. CSS Module

We need to have a hash value to solve the problem of overlapping class names.  
[Vanilla Extract's `style()`](https://vanilla-extract.style/documentation/api/style/) is already doing a good job, so keep it.

**Code:**
```typescript
const myCss = css({
  color: "blue",
  backgroundColor: "#EEEEEE"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  color: blue;
  background-color: #EEEEEE;
}
```

[Identifiers](https://vanilla-extract.style/documentation/integrations/vite/#identifiers) can be changed with settings.

## 4. Unitless Properties

[Unitless Properties](https://vanilla-extract.style/documentation/styling#unitless-properties) is convenient because it reduces unnecessary string representations, so keep it.

**Code:**
```typescript
export const myCss = css({
  // cast to pixels
  padding: 10,
  marginTop: 25,

  // unitless properties
  flexGrow: 1,
  opacity: 0.5
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  padding: 10px;
  margin-top: 25px;

  flex-grow: 1;
  opacity: 0.5;
}
```

## 5. Vendor Prefixes

[Vendor Prefixes](https://vanilla-extract.style/documentation/styling#vendor-prefixes) is convenient because it reduces unnecessary string representations, so keep it.

**Code:**
```typescript
export const myCss = css({
  WebkitTapHighlightColor: "rgba(0, 0, 0, 0)"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}
```

## 6. Fallback Styles

[Fallback Styles](https://vanilla-extract.style/documentation/styling#fallback-styles) is convenient because it reduces unnecessary properties, so keep it.

**Code:**
```typescript
export const myCss = css({
  // In Firefox and IE the "overflow: overlay" will be
  // ignored and the "overflow: auto" will be applied
  overflow: ["auto", "overlay"]
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  overflow: auto;
  overflow: overlay;
}
```

## 7. Merge Values

Inspired by the [Less's Merge properties](https://lesscss.org/features/#merge-feature), this feature allows you to composition long split values.

Only `$` and `_` can use special characters that don't require a string in javascript.  
By convention in the programming world, we use `_` to represent a whitespace, so we do the same.

**Code:**
```typescript
export const myCss = css({
  boxShadow$: ["inset 0 0 10px #555", "0 0 20px black"],
  transform_: ["scale(2)", "rotate(15deg)"]
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  boxShadow: "inset 0 0 10px #555, 0 0 20px black",
  transform: "scale(2) rotate(15deg)"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  boxShadow: inset 0 0 10px #555, 0 0 20px black;
  transform: scale(2) rotate(15deg);
}
```

For use with Fallback Styles, use a double array.  
It's automatically composited.

**Code:**
```typescript
export const myCss = css({
  transform_: [
    // Apply to all
    "scale(2)",

    //  Fallback style
    ["rotate(28.64deg)", "rotate(0.5rad)"]
  ]
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  transform: ["scale(2) rotate(28.64deg)", "scale(2) rotate(0.5rad)"]
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  transform: scale(2) rotate(28.64deg);
  transform: scale(2) rotate(0.5rad);
}
```

## 8. Simply Important

Inspired by the [Tailwind's Important modifier](https://tailwindcss.com/docs/configuration#important-modifier), If `!` is at the end of the value, treat it as `!important`.

**Code:**
```typescript
export const myCss = css({
  color: "red!"
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  color: "red !important"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  color: red !important;
}
```

## 9. CSS Variables

Unlike [Vanilla Extract's CSS Variables](https://vanilla-extract.style/documentation/styling#css-variables), it is supported at the top level.  
Inspired by the [SASS Variable](https://sass-lang.com/documentation/variables/), You can use `$` like you would a variable.

The conversion to prefix and `kebab-case` happens automatically.

**Code:**
```typescript
export const myCss = css({
  $myGlobalVariable: "purple",
  color: "$myGlobalVariable"
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  vars: {
    "--my-global-variable": "purple"
  },
  color: "var(--my-global-variable)"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  --my-global-variable: purple;
  color: var(--my-global-variable);
}
```

## 10. Simple Pseudo Selectors

[Simple Pseudo Selectors](https://vanilla-extract.style/documentation/styling#simple-pseudo-selectors) is convenient because these are the elements you typically use with ["&"](https://sass-lang.com/documentation/style-rules/parent-selector/), so keep it.

Inspired by the [Panda CSS's Conditional Styles](https://panda-css.com/), `_` is used as `:`.  
However, no other classes or attributes are added, it's a simple conversion.  
`camelCase` also convert to `kebab-case`.

**Code:**
```typescript
export const myCss = css({
  _hover: {
    color: "pink"
  },
  _firstOfType: {
    color: "blue"
  },
  __before: {
    content: ""
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  ":hover": {
    color: "pink"
  },
  ":first-of-type": {
    color: "blue"
  },
  "::before": {
    content: ""
  }
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH]:hover {
  color: pink;
}

.[FILE_NAME]_myCSS__[HASH]:first-of-type {
  color: blue;
}

.[FILE_NAME]_myCSS__[HASH]::before {
  content: "";
}
```

## 11. Complex Selectors

Unlike [Vanilla Extract's Complex Selectors](https://vanilla-extract.style/documentation/styling#complex-selectors), it is supported at the top level.  
I want to reduce nesting as much as possible.

Exception values for all properties are treated as complex selectors.

**Code:**
```typescript
export const myCss = css({
 "&:hover:not(:active)": {
    border: "2px solid aquamarine"
  },
  "nav li > &": {
    textDecoration: "underline"
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  selectors: {
   "&:hover:not(:active)": {
      border: "2px solid aquamarine"
    },
    "nav li > &": {
      textDecoration: "underline"
    }
  }
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH]:hover:not(:active) {
  border: 2px solid aquamarine;
}

nav li > .[FILE_NAME]_myCSS__[HASH] {
  text-decoration: underline;
}
```

## 12. At-Rules

Allows nesting, like [Vanilla Extract's Media Queries](https://vanilla-extract.style/documentation/styling#media-queries), and also allows top-levels.

**Code:**
```typescript
export const myCss = css({
  // Nested
  "@media": {
    "screen and (min-width: 768px)": {
      padding: 10
    },
    "(prefers-reduced-motion)": {
      transitionProperty: "color"
    }
  },

  // Top level
  "@supports (display: grid)": {
    display: "grid"
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  "@media": {
    "screen and (min-width: 768px)": {
      padding: 10
    },
    "(prefers-reduced-motion)": {
      transitionProperty: "color"
    }
  },

  "@supports": {
    "(display: grid)": {
      display: "grid"
    }
  }
});
```

**Compiled:**
```css
@media screen and (min-width: 768px) {
  .[FILE_NAME]_myCSS__[HASH] {
    padding: 10px;
  }
}

@media (prefers-reduced-motion) {
  .[FILE_NAME]_myCSS__[HASH] {
    transition-property: color;
  }
}

@supports (display: grid) {
  .[FILE_NAME]_myCSS__[HASH] {
    display: grid;
  }
}
```

## 13. Anonymous At-Rules

Inspired by the [Griffel's Keyframes](https://griffel.js.org/react/api/make-styles#keyframes-animations), Makes [`@keyframes`](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes) or [`@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) writable inline.

`fontFamily$` is used as special case of the `Merge Values` rule.

**Code:**
```typescript
export const myCss = css({
  // Keyframes
  animationName: {
    "0%": { transform: "rotate(0deg)" },
    "100%": { transform: "rotate(360deg)" }
  },
  animationDuration: "3s",

  // Fontface
  fontFamily: {
    src: "local('Comic Sans MS')"
  }
  // Fontface with multiple
  fontfamily$: [{ src: "local('Noto Sans')" }, { src: "local('Gentium')" }]
});
```

**Convert to Vanilla Extract:**
```typescript
const myCssKeyframes = keyframes({
  "0%": { transform: "rotate(0deg)" },
  "100%": { transform: "rotate(360deg)" }
});

const myCssFontFace1 = fontFace({ src: "local('Comic Sans MS')" });
const myCssFontFace2 = fontFace({ src: "local('Noto Sans')" });
const myCssFontFace3 = fontFace({ src: "local('Gentium')" });

export const myCss = style({
  animationName: myCssKeyframes,
  animationDuration: "3s",

  fontFamily: myCssFontFace1,
  fontFamily: `${myCssFontFace2}, ${myCssFontFace3}`
});
```

**Compiled:**
```css
@keyframes [FILE_NAME]_myCSSKeyframes__[HASH] {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@font-face {
  src: local("Comic Sans MS");
  font-family: "[FILE_NAME]_myCSSFontFace1__[HASH]";
}
@font-face {
  src: local("Noto Sans");
  font-family: "[FILE_NAME]_myCSSFontFace2__[HASH]";
}
@font-face {
  src: local("Gentium");
  font-family: "[FILE_NAME]_myCSSFontFace3__[HASH]";
}

.[FILE_NAME]_myCSS__[HASH] {
  animation-name: [FILE_NAME]_myCSSKeyframes__[HASH];
  animation-duration: 3s;

  font-family: [FILE_NAME]_myCSSFontFace1__[HASH];
  font-family: [FILE_NAME]_myCSSFontFace2__[HASH], [FILE_NAME]_myCSSFontFace3__[HASH];
}
```

## 14. CSS Composition

[Vanilla Extract's composition](https://vanilla-extract.style/documentation/style-composition/) is well enough made, so keep it.

**Code:**
```typescript
const base = css({ padding: 12 });
const primary = css([base, { background: "blue" }]);
const secondary = css([base, { background: "aqua" }]);
```

**Compiled:**
```css
.[FILE_NAME]_base__[HASH] {
  padding: 12px;
}

.[FILE_NAME]_base__[HASH] {
  background: blue;
}

.[FILE_NAME]_base__[HASH] {
  background: aqua;
}
```

## 15. Backward compatibility

Backward compatibility with [Vanilla Extract's `style()`](https://vanilla-extract.style/documentation/api/style/) must be preserved.

- `css()` is alised to `style()`
- Should be support [`vars: `](https://vanilla-extract.style/documentation/styling#css-variables) and [`selectors: `](https://vanilla-extract.style/documentation/styling#complex-selectors) properties

# Reference-level explanation

If you make a lot of things top level, it can become difficult to make them type-safe.

However, it is possible to implement it well using [Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) and [Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html).

```typescript
type Rule = "media" | "supports" | "container" | "layer"; // And more..?
type CSSNode = {};
type AtRules = {
  [key in Rule as `@${key}`]: { [key in string]: CSSNode };
} & {
  [key in Rule as `@${key} ${string}`]: CSSNode;
};

const myCss: AtRules = {
  // Nested
  "@media": {
    "screen and (min-width: 768px)": {
      padding: 10
    },
    "(prefers-reduced-motion)": {
      transitionProperty: "color"
    }
  },

  // Top level
  "@supports (display: grid)": {
    display: "grid"
  }
}
```

If it turns out to be difficult, let's go in a different direction.

# Drawbacks

## Object Styles

Unlike template style, object style cannot be pasted from an existing CSS file.  
`kebab-case` is not available and in many cases must be represented using a string of values.

```typescript
// Object Style
const myCss = css({
  display: "inline-block",
  color: "blue",
  borderRadius: "3px",
  width: "10rem"
});

// Template Style
const myCss = css`
  display: inline-block;
  color: blue;
  border-radius: 3px;
  width: 10rem;
`;
```

String represented, which creates three problems.
- Uncomfortable to write
- Doesn't recognize CSS units(px, rem, vw)
- The color picker doesn't automatically appear for color codes either

Fortunately, the color picker extension([VS Code](https://marketplace.visualstudio.com/items?itemName=AntiAntiSepticeye.vscode-color-picker), [Emacs](https://jblevins.org/log/rainbow-mode)) is a bit of a workaround.

## Complex Selectors - Reference constraints

That it inherits all of Vanilla Extract's constraints.
```typescript
const invalid = style({
  // ❌ ERROR: Targetting `a[href]`
  "& a[href]": {...},

  // ❌ ERROR: Targetting `.otherClass`
  "& ~ div > .otherClass": {...}
});

// Also Invalid example:
export const child = style({});
export const parent = style({
  // ❌ ERROR: Targetting `child` from `parent`
  [`& ${child}`]: {...}
});

// Valid example:
export const parent = style({});
export const child = style({
  [`${parent} &`]: {...}
});
```

## Complex Selectors - Circular reference

As above, [Circular reference](https://vanilla-extract.style/documentation/styling/#circular-selectors) is the same.
```typescript
export const child = css({
  background: "blue",
  get selectors() {
    return {
      [`${parent} &`]: {
        color: 'red'
      }
    };
  }
});

export const parent = css({
  background: "yellow",
  selectors: {
    [`&:has(${child})`]: {
      padding: 10
    }
  }
});
```

# Rationale and alternatives

The goal of this RFC is to make writing CSS in Typescript as natural as it was in the first place.  
So [CSS Modules](https://github.com/css-modules/css-modules) or [preprocessors](https://github.com/awesome-css-group/awesome-css#preprocessors-pill) is a very big competitor and substitute.

In the CSS in JS world, The closest to our task is [emotion-to-vanilla-extract](https://github.com/penx/emotion-to-vanilla-extract), but it's mostly different except for the nesting.  
Our solution is much cleaner.

# Unresolved questions

## CSS Variables

Is automatically converting the right design?  
I think the prefix is good, but I'm not sure about the `camelCase` to `kebab-case` conversion.

## Other preprocessors feature

Nested syntax([property](https://sass-lang.com/documentation/style-rules/declarations/#nesting), [selector](https://sass-lang.com/documentation/style-rules/parent-selector/)) in SCSS, referential syntax([property](https://stylus-lang.com/docs/variables.html#property-lookup), [selector](https://stylus-lang.com/docs/selectors.html#partial-reference)) in Stylus, etc. is still powerful.

# Future possibilities

## Nesting and Reference
Nested and referential syntax is intentionally left out because the transform rules are quite complex.  
I'm trying to solve the problem of nesting of properties as well as selectors.

This will be the topic of the next RFC I write.

## Complex Selectors
If you want to circumvent the constraints of Complex Selectors, write a new RFC for the conversion under the name `xxx-complex-selectors.md`.

## Anonymous At-Rules
For now, we only support `@keyframes` and `@font-face`, but the list may grow in the future.
