- Start Date: 2024-01-27

# Summary
Suggestions for better nesting and referencing.

# Motivation
- Take advantage of preprocessors
- Finds as many alternatives as possible within the parsing syntax of Javascript
- Keep related properties or rules close by to make it easier to understand the context

# Guide-level explanation
1. Maintain a locality of thinking.
2. Deeper nesting is bad and should be encouraged to be reduced.
3. With nesting, you can isolate or reuse contexts.
4. Nesting should be automatically hosted.
5. Properties and variants can be referenced.

## 1. Simple Attribute Selectors
Allow toplevel [`attribute selector`](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) to prevent deep nesting.
It would be nice to be able to autocomplete [HTML attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes).

If the start is `[` without `&` treat it as `attribute selectors`.
It is a continuation of [Simple Pseudo Selectors](./000-css-literals.md#10-simple-pseudo-selectors).

**Code:**
```typescript
export const myCss = css({
  "[disabled]": {
    color: "red"
  },
  `[href^="https://"][href$=".org"]`: {
    color: "blue"
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  selectors: {
    "&[disabled]": {
      color: "red"
    },
    `&[href^="https://"][href$=".org"]`: {
      color: "blue"
    }
  }
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH][disabled] {
  color: red;
}

.[FILE_NAME]_myCSS__[HASH][href^="https://"][href$=".org"] {
  color: blue;
}
```

## 2. Nested Properties
Inspired by the [SCSS's nested properties](https://sass-lang.com/documentation/style-rules/declarations/#nesting), this feature allows nesting for property names.

Reduce redundancy and make your context stand out. 

Uppercase it to distinguish it from `Property based condition`.
[`Vendor Prefixes`](./000-css-literals.md#5-vendor-prefixes) exists only in Top level, while `Nested Properties` exists only in nesting, so you can tell them apart.

**Code:**
```typescript
export const myCss = css({
  transition: {
    Property: "font-size",
    Duration: "4s",
    Delay: "2s"
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  transitionProperty: "font-size",
  transitionDuration: "4s",
  transitionDelay: "2s"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  transition-property: font-size;
  transition-duration: 4s;
  transition-delay: 2s;
}
```

## 3. Property based condition
Inspired by the [Panda CSS](https://panda-css.com/docs/concepts/conditional-styles#property-based-condition), You can apply properties based on selectors or at-rules.

The default properties refer to `base`.

```typescript
export const myCss = css({
  color: {
    base: "red",
    _hover: "green",
    "[disabled]": "blue",
    "nav li > &": "black",
    "@media (prefers-color-scheme: dark)": "white"
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  color: "red",
  ":hover": {
    color: "green"
  },
  selectors: {
    "&[disabled]": {
      color: "blue"
    },
    "nav li > &": {
      color: "black"
    }
  },
  "@media": {
    "(prefers-color-scheme: dark)": {
      color: "white"
    }
  }
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  color: red;
}

.[FILE_NAME]_myCSS__[HASH]:hover {
  color: green;
}

.[FILE_NAME]_myCSS__[HASH][disabled] {
  color: blue;
}

nav li > .[FILE_NAME]_myCSS__[HASH] {
  color: black;
}

@media (prefers-color-scheme: dark) {
  .[FILE_NAME]_myCSS__[HASH] {
    color: red;
  }
}
```

## 4. Nested Selectors
Inspired by the [SCSS's nested selectors](https://sass-lang.com/documentation/style-rules/parent-selector/), this feature allows nesting for selectors.

It works with [Simple Pseudo Selectors](./000-css-literals.md#10-simple-pseudo-selectors) and [Complex Selectors](./000-css-literals.md#11-complex-selectors).

```typescript
export const myCss = css({
  "nav li > &": {
    color: "red",
    _hover: {
      color: "green"
    },
    "&:hover:not(:active)": {
      color: "blue"
    },
    ":root[dir=rtl] &": {
      color: "black"
    }
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  selectors: {
    "nav li > &": {
      color: "red"
    },
    "nav li > &:hover": {
      color: "green"
    },
    "nav li > &:hover:not(:active)": {
      color: "blue"
    },
    ":root[dir=rtl] nav li > &": {
      color: "black"
    }
  }
});
```

**Compiled:**
```css
nav li > .[FILE_NAME]_myCSS__[HASH] {
  color: red;
}

nav li > .[FILE_NAME]_myCSS__[HASH]:hover {
  color: green;
}

nav li > .[FILE_NAME]_myCSS__[HASH][disabled]:hover:not(:active) {
  color: blue;
}

:root[dir=rtl] nav li > .[FILE_NAME]_myCSS__[HASH] {
  color: black;
}
```

## 5. Nested At-Rules
Like `Nested Selectors`, but they are hoisted and combined into a `AND` rule.

Depending on the `Ar-Rules` keyword, the combining syntax is slightly different.  
(Unlike [`@media`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media), [`@supports`](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports), and [`@container`](https://developer.mozilla.org/en-US/docs/Web/CSS/@container), [`@layer`](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer) is displayed like `parent.child`.)

**Code:**
```typescript
export const myCss = css({
  "nav li > &": {
    color: "red",

    "@media (prefers-color-scheme: dark)": {
      "@media": {
        "(prefers-reduced-motion)": {
          color: "green"
        },
        "(min-width: 900px)": {
          color: "blue"
        }
      }
    },

    "@layer framework": {
      "@layer": {
        "layout": {
          color: "black"
        },
        "utilities": {
          color: "white"
        }
      }
    }
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  selectors: {
    "nav li > &": {
      color: "red"
    }
  },

  "@media": {
    "(prefers-color-scheme: dark) and (prefers-reduced-motion)": {
      selectors: {
        "nav li > &": {
          color: "green"
        }
      },
    },
    "(prefers-color-scheme: dark) and (min-width: 900px)": {
      selectors: {
        "nav li > &": {
          color: "blue"
        }
      },
    }
  },
  
  "@layer": {
    "framework.layout": {
      selectors: {
        "nav li > &": {
          color: "black"
        }
      },
    },
    "framework.utilities": {
      selectors: {
        "nav li > &": {
          color: "white"
        }
      },
    }
  }
});
```

**Compiled:**
```css
nav li > .[FILE_NAME]_myCSS__[HASH] {
  color: red;
}

@media (prefers-color-scheme: dark) and (prefers-reduced-motion) {
  nav li > .[FILE_NAME]_myCSS__[HASH] {
    color: green;
  }
}

@media (prefers-color-scheme: dark) and (min-width: 900px) {
  nav li > .[FILE_NAME]_myCSS__[HASH] {
    color: blue;
  }
}


@layer framework.layout {
  nav li > .[FILE_NAME]_myCSS__[HASH] {
    color: blue;
  }
}

@layer framework.utilities {
  nav li > .[FILE_NAME]_myCSS__[HASH] {
    color: blue;
  }
}
```

It can be used with `Property based condition`.

**Code:**
```typescript
export const myCss = css({
  "nav li > &": {
    color: {
      base: "red",
      "@media (prefers-color-scheme: dark)": {
        "@media (prefers-reduced-motion)": "green",
        "@media (min-width: 900px)": "blue"
      },
      "@layer framework": {
        "@layer": {
          "layout": "black",
          "utilities": "white"
        }
      }
    }
  }
});
```

## 6. Property Reference
Inspired by the [Stylus's property lookup](https://stylus-lang.com/docs/variables.html#property-lookup), this feature can be used to refer to a property value.

**Code:**
```typescript
export const myCss = css({
  width: "50px",
  height: "@width",
  margin: "calc(@width / 2)"
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  width: "50px",
  height: "50px",
  margin: "calc(50px / 2)"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS__[HASH] {
  width: 50px
  height: 50px;
  margin: calc(50px / 2);
}
```

## 7. Variants Reference
Inspired by the [JSS plugin nested](https://cssinjs.org/jss-plugin-nested?v=v10.10.0#use-rulename-to-reference-a-local-rule-within-the-same-style-sheet), this feature can be reference a local rule.

Use the `%` symbol.

**Code:**
```typescript
export const myCss = cssVariant({
  primary: {
    color: "red",
    ":has(%secondary)": {
      color: "blue"
    }
  },
  secondary: {
    color: "black",
    "%primary &":{
      color: "white"
    }
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = styleVariant({
  primary: {
    color: "red"
  },
  secondary: {
    color: "black"
  }
});
globalCSS(`${myCss["primary"]}:has(${myCss["secondary"]})`, {
  color: "blue"
});
globalCSS(`${myCss["primary"]} ${myCss["secondary"]}`, {
  color: "white"
});
```

**Compiled:**
```css
.[FILE_NAME]_myCSS_primary__[HASH] {
  color: red;
}

.[FILE_NAME]_myCSS_primary__[HASH]:has(.[FILE_NAME]_myCSS_secondary__[HASH]) {
  color: blue;
}

.[FILE_NAME]_myCSS_secondary__[HASH] {
  color: black;
}

.[FILE_NAME]_myCSS_primary__[HASH] .[FILE_NAME]_myCSS_secondary__[HASH] {
  color: white;
}
```

# Reference-level explanation
It's more complicated, but can be implemented almost exactly like [CSS Literals](./000-css-literals.md).

## Stabilize nesting order

To resolve complex nested structures, it is recommended to order them when nested.

This isn't a problem if each element appears only once, for example.

**Code:**
```typescript
export const myCss1 = css({
  "nav li > &": {
    "@media (prefers-color-scheme: dark)": {
      background: "red"
    }
  }
});

const myCss2 = css({
  "nav li > &": {
    "@media (prefers-color-scheme: dark)": {
      _hover: {
        background: "red"
      }
    }
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss1 = style({
  selectors: {
    "nav li > &": {
      "@media": {
        "(prefers-color-scheme: dark)": {
          background: "red"
        }
      }
    }
  }
});

export const myCss2 = style({
  "@media": {
    "(prefers-color-scheme: dark)": {
      selectors: {
        "nav li > &": {
          background: "red"
        }
      }
    }
  }
});
```

But what about the following cases?

**Code:**
```typescript
const myCss = css({
  "nav li > &": {
    "@media (prefers-color-scheme: dark)": {
      _hover: {
        background: "red"
      }
    }
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  // ???
});
```

Therefore, we need to force a specific order to resolve them.

### Targets

At-Rules(`@media`, `@supports`, `@container`, `@layer`) and selectors.

1. `@media`: Apply styles based on screen size or device characteristics.
2. `@supports:` Check for support for specific features first.
3. `@container`: Apply styles based on the size or characteristics of a specific container.
4. `@layer`: Define the priority of the style.
5. selectors: Apply the actual style.

Nested properties are not a problem because they are automatically transformed to have a flattened structure.

### Considerations

Think about performance and readability.  
Overall, it makes sense to gradually narrow the scope and conditions for styles.

- Performance: It is advantageous to have conditions that do not change at low levels and conditions that change frequently at deep levels.
- Readability: The more similar contexts they have, the more they should be at the same level.

**Nesting for performance**  
This is a generalized approach, and we ***haven't done any specific benchmarks yet***.
1. `@layer`: It defines the entire style sheet structure, so it helps the browser prioritize it when it's on the outermost part.
2. `@supports`: If it's on top, browser can avoid unnecessary style calculations.
3. `@media`: Apply styles based on screen size or device characteristics.
4. `@container`: It's in `@media` because it depends on the size and characteristics of the specific element, not the whole.
5. selectors: It is evaluated most frequently, so it should be the innermost.

However, the order of `@media` and `@container` can change depending on your project.  
Since it applies styles based on the size of a particular element, it can be placed at the top to reduce unnecessary media query calculations.  
If container queries are used more frequently, and the size of the container changes frequently independent of the viewport, it may be advantageous to have `@container` on top of `@media`.

Also, even if `@supports` is inside, it may not have a significant impact on performance.  
In fact, for Firefox, the representation inside `@supports` is [not re-evaluated unless](https://github.com/black7375/Firefox-UI-Fix/blob/master/docs/Restrictions.md#supports) it is reloaded.
(Only need to check once on page load)

**Nesting for readability**
1. `@layer`: Since this is the structure of the entire style sheet, it makes sense that it is declared first.
2. `@container`: Since it is applied based on the conditions of a specific element, it is better to be on the outside.
3. `@media`: It is recommended that styles branch based on conditions inside the container.
4. `@supports`: It checks for support of certain CSS features, so it should be inside. Near `@supports`, there might be a style for fallback.
5. selectors: This is the most specific collection of CSS properties and values and should be the innermost.

### Decision

Follow order of `Nesting for performance`.  
This is because we think it's better to consider performance as the library responsible for the overall CSS output.  
I also don't think it's in any order that is significantly less readable.

This decision can be overturned later based on benchmark results, ease of debugging, etc.

### Examples

You can now answer the question.

**Code:**
```typescript
const myCss = css({
  "nav li > &": {
    "@media (prefers-color-scheme: dark)": {
      _hover: {
        background: "red"
      }
    }
  }
});
```

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  "@media": {
    "(prefers-color-scheme: dark)": {
      selectors: {
        "nav li > &:hover": {
          background: "red"
        }
      }
    }
  }
});
```

It sounds complicated, but it's always transpiled with the following structure:

**Convert to Vanilla Extract:**
```typescript
export const myCss = style({
  // Level1: @layer
  "@layer": {
    "framework.layout": {

      // Level2: @supports
      "@supports": {
        "gap: 1rem": {

          // Level3: @media
          "@media": {
            "(prefers-color-scheme: dark) and (prefers-reduced-motion)": {

              // Level4: @container
              "@container": {
                "(min-width: 500px)": {

                  // Level5: selectors
                  "selectors": {
                    "nav li > &:hover": {
                      background: "red"
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
});
```

# Drawbacks
Nested syntax reduces duplication, but if abused, it can be hard to know the context.

# Rationale and alternatives
The syntax for literal CSS is now on the same level as the preprocessors.
The selector stuff is definitely [Stylus](https://stylus-lang.com/docs/selectors.html#partial-reference)-rich, but it's generally okay.

## Partial Reference

Using the `^[N]`, `^[N..M]` syntax, You can reference the parent, but it's confusing.
It's easy to make a mistake, and it's easy to encourage nesting.

If you need them in the future, you should add them very carefully.

**SCSS like Syntax:**
```scss
.foo {
  &__bar {
    &__baz {
      width: 10px;

      ^[0]:hover & {
        width: 20px;
      }
      ^[1]:hover & {
        width: 30px;
      }
      ^[2]:hover & {
        width: 40px;
      }
      ^[-1]:hover & {
        width: 50px;
      }
      ^[-2]:hover & {
        width: 60px;
      }
      ^[1..-1]:hover & {
        width: 70px
      }
    }
  }
}
```

**Compiled:**
```css
.foo__bar_baz {
  width: 10px;
}

.foo:hover .foo__bar_baz {
  width: 20px;
}

.foo__bar:hover .foo__bar_baz {
  width: 30px;
}

.foo__bar_baz:hover .foo__bar_baz {
  width: 40px;
}

.foo__bar:hover .foo__bar_baz {
  width: 50px;
}

.foo:hover .foo__bar_baz {
  width: 60px;
}

__bar__baz:hover .foo__bar__baz {
  width: 70px;
}
```

## Path-like Reference

Using the `/`, `~`, `../` syntax, which looks like it's used in paths, is less confusing, but still complicated.

**SCSS like Syntax:**
```scss
.block {
  &__element {
    /.root {
      color: red;
    }

    ~/:hover & {
      color: green;
    }

    ../::before {
      color: blue;
    }
  }
}
```

**Compiled:**
```css
.root {
  color: red;
}

.block:hover .block__element {
  color: green;
}

.block::before {
  color: blue;
}
```

# Unresolved questions
Implementing complex nested syntax type-safely can be quite tricky.
You can give it a try and abandon some syntax if it's really hard.

# Future possibilities
## Rules
Literals are fine for static cases, but it doesn't yet cover cases that require props.
It will be partial coverage of [`@mixin` in SCSS](https://sass-lang.com/documentation/at-rules/mixin/) and [Variants in Stitches](https://stitches.dev/docs/variants).

This will be the topic of the next RFC I write.
