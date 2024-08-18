- Start Date: 2024-02-24

# Summary

Styling for cases where props are required.  
It will be partial coverage of [`@mixin` in SCSS](https://sass-lang.com/documentation/at-rules/mixin/) and [Variants in Stitches](https://stitches.dev/docs/variants).

# Motivation

- Complementing composition
- Dynamic styling
- Take advantage of preprocessors
- Variants that correspond to semantic hierarchies
- Improve [Vanilla Extract's recipes](https://vanilla-extract.style/documentation/packages/recipes/)
- Make your BEM expressive

## Composition

[css literals RFC](./000-css-literals.md#2-object-styles) is used to create each class.

It's more of a feature for a single class.

```tsx
const mySample = css({ color: "red", padding: 3 });

function Sample() {
  return <div className={mySample}>contents...</div>;
}
```

In reality, you'll likely use multiple classes.

```tsx
function Sample() {
  const activate = useState(true);
  return (
    <div className={`class1 class2 ${activate && "class3"}`}>contents...</div>
  );
}
```

This is where using [clsx](https://github.com/lukeed/clsx) looks better.

```tsx
function Sample() {
  const activate = useState(true);
  return (
    <div className={clsx(["class1", "class2", { class3: activate }])}>
      contents...
    </div>
  );
}
```

For the static case, you can leverage formal [compositing to reuse styles](./000-css-literals.md#14-css-composition) for optimization.

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

.[FILE_NAME]_primary__[HASH] {
  background: blue;
}

.[FILE_NAME]_secondary__[HASH] {
  background: aqua;
}
```

## Dynamic Styling

It's clear that composition alone can't handle all dynamic cases.  
What about dynamic cases?

The solution is known as [`UI = f(state)`](https://docs.flutter.dev/data-and-backend/state-mgmt/declarative) and it's simple.

If so, is there any reason for Style to not be a function?  
This is the same as [fela's approach](https://fela.js.org/docs/latest/intro/principles).

```typescript
const rule = (props) => ({
  fontSize: props.size + "px",
  color: "blue",
  lineHeight: props.lineHeight,
  display: "flex"
});
```

In some cases, a property can also be a function.

```typescript
const rule = {
  fontSize: (props) => props.size + "px",
  color: "blue",
  lineHeight: (props) => props.lineHeight,
  display: "flex"
};
```

Let's apply it to an existing `css()`?

```typescript
// Functions are arguments, so type inference is possible
const sample1 = css(({ color }) => ({
  color
}));

// Type inference is tricky because properties are functions
const sample2 = css({
  color: ({ color }) => color
});

function Sample({ color }) {
  return <div className={sample1({ color })}>contents...</div>;
}
```

While it's not impossible to integrate dynamic features, I realized that it would introduce a lot of confusion in real-world use and should be separated.

```typescript
// Always returns a string(class name)
const static: string = css({
  color: "red"
});

// Always return a function that returns a string(class name)
const dynamic: <T>(props: T) => string = something(({ color }) => ({
  color,
  background: ({ bg } = { bg: "blue" }) => bg
}));
```

## Mixin

Mixins in SCSS are already partially replaceable using compositing and merging.

```scss
@mixin square($size, $radius: 0) {
  width: $size;
  height: $size;

  @if $radius != 0 {
    border-radius: $radius;
  }
}

.avatar {
  border: none;
  @include square(100px, $radius: 4px);
}
```

In javascript, this would be code like this.

```typescript
function square(size: number, radius = 0) {
  const styles = {
    width: size,
    height: size
  };

  if (radius !== 0) {
    styles.borderRadius = radius;
  }

  return styles;
}

const avatar = css([{ border: "none" }, square(100, 4)]);
```

Control flow is very powerful, but it's also very easy to make mistakes.  
It leaves the way out open, but we need to find a way to make it as declarative as possible and as simple as possible.

## Variants

[Variants](https://stitches.dev/docs/variants), made famous by [stitches](https://stitches.dev/), makes some control flows declarative.

The Vanilla extract, which this project is based on, is called [recipe](https://vanilla-extract.style/documentation/packages/recipes/).

```typescript
const avatar = recipe({
  base: {
    border: "none"
  },

  defaultVariants: {
    size: "medium",
    rounded: true
  },
  variants: {
    size: {
      small: {
        width: 12,
        height: 12
      },
      medium: {
        width: 16,
        height: 16
      },
      large: {
        width: 24,
        height: 24
      }
    },
    rounded: {
      true: {
        borderRadius: 999
      }
    }
  }
});
```

At first glance, it seems like it's only useful in static cases.  
However, it does provide pattern matching to compensate for dynamic cases, and always allows for intentional design.

```typescript
const avatar = recipe({
  base: {
    /* ... */
  },
  variants: {
    /* ... */
  },
  compoundVariants: [
    {
      variants: {
        size: "small",
        rounded: true
      },
      style: {
        background: "red"
      }
    }
  ]
});
```

Variants are an example of declarative dynamic styling.

## BEM

[BEM](https://getbem.com) is a popular CSS management methodology.

- Block: Standalone entity that is meaningful on its own.

This can be expressed by the use of the generic `recipe`.

```typescript
const block = recipe({
  base: {
    /* ... */
  }
});
```

- Element: A part of a block that has no standalone meaning and is semantically tied to its block.
  (e.g. menu item, list item, checkbox caption, header title)

This can be expressed as `recipe` being in the same file as `Block`.

```typescript
const block = recipe({
  /* ... */
});

const element = recipe({
  base: {
    /* ... */
  }
});
```

- Modifier: A flag on a block or element. Use them to change appearance or behavior.
  (e.g. disabled, highlighted, checked, fixed, size big, color yellow)

This can be represented as a variant of a `recipe`.

```typescript
const block = recipe({
  base: {
    /* ... */
  },
  variants: {
    /* ... */
  }
});

const element = recipe({
  base: {
    /* ... */
  },
  variants: {
    /* ... */
  }
});
```

Now you can make your components the BEM way.

```tsx
import blockName from "./block.css";

function Block() {
  return (
    <div
      className={blockName.block({
        /* with Modifier */
      })}
    >
      <div
        className={blockName.element({
          /* with Modifier */
        })}
      ></div>
    </div>
  );
}
```

# Guide-level explanation

1. Provide and extend the variant features of [Stitches's variants](https://stitches.dev/docs/variants) and [Vanilla Extract's recipes](https://vanilla-extract.style/documentation/packages/recipes/).
2. Provides syntax sugar for a few special cases like props, toggles.
3. It should be available for use with `cssVariant`.

## 1. Name

The API is named `rules()`.

This is to reduce confusion with `cssVariant` and to go beyond variant feature like props.

Inspired by the [Fela rules](https://fela.js.org/docs/latest/basics/rules) and [UnoCSS rules](https://unocss.dev/config/rules).

## 2. Basic usage

Define it as an object style, similar to [css literals RFC](./000-css-literals.md#2-object-styles).

**Code:**

```typescript
const myRule = rules({
  color: "blue",
  backgroundColor: "red"
});
```

**Compiled:**

```css
.[FILE_NAME]_myRule__[HASH] {
  color: blue;
  background-color: red;
}
```

However, it is returned as a function, so you need to run it to use it.

**Usage:**

```tsx
function MyComponent() {
  return <div className={myCSS()}></div>;
}
```

## 3. Props

AST traversal is required for the implementation of functional rules when static extraction is required.

```typescript
inteface MyRulesProps {
  color: string;
  background: string;
  size: number;
}

const myRule = rules<MyRuleProps>(({ color, size }) => ({
  color,
  background: (value = "blue") => value,
  padding: size
}));
```

This complicates the implementation and should be avoided.

Fortunately, in many cases, props are used as is.  
You can remove unnecessary boilerplate code and group properties that need to do similar things together.

**Code:**

```typescript
const myRule = rules({
  props: ["color", "background"]
});
```

**Compiled:**

```css
.[FILE_NAME]_myRule__[HASH] {
  color: var(--[FILE_NAME]_myRule_color__[HASH]);
  background: var(--[FILE_NAME]_myCSS_background__[HASH]);
}
```

You may also need aliased.

**Code:**

```typescript
const myRule = rules({
  props: ["color", "background", { size: ["padding", "margin"] }]
});
```

**Compiled:**

```css
.[FILE_NAME]_myRule__[HASH] {
  color: var(--[FILE_NAME]_myRule_color__[HASH]);
  background: var(--[FILE_NAME]_myRule_background__[HASH]);
  padding: var(--[FILE_NAME]_myRule_size__[HASH]);
  margin: var(--[FILE_NAME]_myRule_size__[HASH]);
}
```

You can also set a default value.

**Code:**

```typescript
const myRule = rules({
  props: [
    "color",
    {
      background: { base: "red" },
      size: { base: "3px", targets: ["padding", "margin"] }
    }
  ]
});
```

**Compiled:**

```css
.[FILE_NAME]_myRule__[HASH] {
  color: var(--[FILE_NAME]_myRule_color__[HASH]);
  background: var(--[FILE_NAME]_myRule_background__[HASH], red);
  padding: var(--[FILE_NAME]_myRule_size__[HASH], 3px);
  margin: var(--[FILE_NAME]_myRule_size__[HASH], 3px);
}
```

You can think of use cases as those that are statically extracted and those that are dynamically assigned.

**Static Usage:**

```typescript
const myCSS = css([
  myRule({ color: "red", background: "blue", size: "5px" }),
  { borderRadis: 999 }
]);
```

**Compiled:**

```css
.[FILE_NAME]_myCSS__[HASH] {
  --myCSS_color__[HASH]: red;
  --myCSS_background__[HASH]: blue;
  --myCSS_size__[HASH]: 5px;
  border-radius: 999px;
}
```

If dynamic case, it is assigned as an inline style, like [Vanilla Extract's `assignInlineVars()`](https://vanilla-extract.style/documentation/packages/dynamic/#assigninlinevars).

**Dynamic Usage:**

```tsx
import { myRule } from "sample.css";

function Sample({ color }) {
  return <div style={myRule.props({ color })}>contents...</div>;
}
```

## 4. Variants

[Stitches's `variants`](https://stitches.dev/docs/variants#adding-variants) is well enough made, so keep it.

**Code:**

```typescript
const button = rules({
  color: "black",
  backgroundColor: "white",
  borderRadius: 6,

  variants: {
    color: {
      brand: {
        color: "#FFFFA0",
        backgroundColor: "blueviolet"
      },
      accent: {
        color: "#FFE4B5",
        backgroundColor: "slateblue"
      }
    },
    size: {
      small: { padding: 12 },
      medium: { padding: 16 },
      large: { padding: 24 }
    },
    rounded: {
      true: { borderRadius: 999 }
    }
  }
});
```

You can use it as if you were using `css`.

**Usage:**

```typescript
button({
  color: "accent",
  size: "large",
  rounded: true
});
```

**Compiled:**

```css
.[FILE_NAME]_button__[HASH] {
  color: black;
  background-color: white;
  border-radius: 6px;
}

.[FILE_NAME]_button_color_brand__[HASH] {
  color: #ffffa0;
  background-color: blueviolet;
}
.[FILE_NAME]_button_color_accent__[HASH] {
  color: #ffe4b5;
  background-color: slateblue;
}

.[FILE_NAME]_button_size_small__[HASH] {
  padding: 12px;
}
.[FILE_NAME]_button_size_medium__[HASH] {
  padding: 16px;
}
.[FILE_NAME]_button_size_large__[HASH] {
  padding: 24px;
}
```

## 5. Toggle Variants

[Stitches's `boolean variants`](https://stitches.dev/docs/variants#boolean-variants) are a special case, but the syntax for defining them is awkward.

Therefore, we introduce a specialized syntax.

**Code Before:**

```typescript
const button = rules({
  // base styles

  variants: {
    // common variants
    rounded: {
      true: { borderRadius: 999 }
    }
  }
});
```

**Code After:**

```typescript
const button = rules({
  // base styles

  toggles: {
    rounded: { borderRadius: 999 }
  }

  variants: {
    // common variants
  }
});
```

It seems also awkward to use `true` even when using.

It can be represented through an array.  
This is also a representation isomorphism with `css()`.

**Usage Before:**

```typescript
button({
  rounded: true,
  color: "accent",
  size: "large"
});
```

**Usage After:**

```typescript
button([
  "rounded",
  {
    color: "accent",
    size: "large"
  }
]);
```

## 6. Compound Variants

[Stitches's `Compound Variants`](https://stitches.dev/docs/variants#compound-variants) is an effective way to set up additional css by leveraging the combination of variations you have already set up.

However, the method of writing the conditions seems quite inconvenient when conditions are complicated.  
So we want to improve the UX in this area.

**Code Before:**

```typescript
const button = rules({
  // base styles

  variants: {
    color: {
      brand: { color: "#FFFFA0" },
      accent: { color: "#FFE4B5" }
    },
    size: {
      small: { padding: 12 },
      medium: { padding: 16 },
      large: { padding: 24 }
    }
  },
  compoundVariants: [
    {
      color: brand,
      size: small,
      style: {
        fontSize: "16px"
      }
    }
  ]
});
```

It doesn't seem uncomfortable when the conditions are not as demanding as they are now.  
But if the conditions become complicated, it will be inconvenient to fill out.

**Code After**

```typescript
const button = rules({
  // base styles

  variants: {
    color: {
      brand: { color: "#FFFFA0" },
      accent: { color: "#FFE4B5" }
    },
    size: {
      small: { padding: 12 },
      medium: { padding: 16 },
      large: { padding: 24 }
    }
  },
  // method 1
  compoundVariants: [
    {
      color: brand,
      size: small,
      style: {
        fontSize: "16px"
      }
    }
  ],
  // method 2
  compoundVariants: ({ color, size }) => [
    {
      condition: [color.brand, size.small],
      style: {
        fontSize: "16px"
      }
    }
  ]
});
```

The first method is the original method in stitches's compound variants, and it is still a good way for it.  
So we keep it.

And the second method we present will provide a more comfortable way to write the conditions.  
This feature checks existing variants and provides automatic completion.

**Compiled:**

```css
.[FILE_NAME]_button_compound_0__[HASH] {
  font-size: '16px'
}
.[FILE_NAME]_button_compound_1__[HASH] {
  font-size: '24px',
  font-weight: bold
}
```

**Usage:**

If you use the already set combination, it will be applied automatically.

```typescript
button({
  color: "accent",
  size: "large"
});
```

## 7. Default Variants

The way of [Stitches's Default Variants](https://stitches.dev/docs/variants#default-variants) is already good to use, so we keep this method in ours.

**Code:**

```typescript
const button = rules({
  // base styles

  variants: {
    color: {
      brand: {
        color: "#FFFFA0",
        backgroundColor: "blueviolet"
      },
      accent: {
        color: "#FFE4B5",
        backgroundColor: "slateblue"
      }
    }
  },
  defaultVariants: {
    color: "brand"
  }
});
```

## 8. Composition

**Code:**

[Vanilla Extract's Composition](https://vanilla-extract.style/documentation/style-composition/) is enough to use already. So we maintain this basically.  
And It gonna inherit base rules like [Stitches's Composing Components](https://stitches.dev/docs/composing-components).

```typescript
const baseRule = rules({
  padding: "12px",

  variants: {
    color: {
      brand: { color: "#FFFFA0" },
      accent: { color: "#FFE4B5" }
    },
    size: {
      small: { fontSize: "11px" },
      large: { fontSize: "20px" }
    }
  }
});

const myRule = rules([
  baseRule,

  {
    color: "blue",
    marginTop: "10px",

    toggles: {
      rounded: { borderRadius: 999 }
    }
  }
]);
```

**Compiled:**

```css
.[FILE_NAME]_baseRule__[HASH] {
  padding: 12px;
}

.[FILE_NAME]_baseRule_color_brand__[HASH] {
  color: "#FFFFA0";
}
.[FILE_NAME]_baseRule_color_accent__[HASH] {
  color: "#FFE4B5";
}
.[FILE_NAME]_baseRule_size_small__[HASH] {
  font-size: 11px;
}
.[FILE_NAME]_baseRule_size_large__[HASH] {
  font-size: 20px;
}

.[FILE_NAME]_myRule__[HASH] {
  color: "blue";
  margin-top: "10px";
}
.[FILE_NAME]_myRule_toggle_rounded__[HASH] {
  border-radius: 999;
}
```

**Usage**

```ts
myRule(["rounded", { color: "brand", size: "small" }]);
```

## 9. rulesVarints

Inspired by the [PandaCSS's `defineSlotRecipe()`](https://panda-css.com/docs/concepts/slot-recipes#config-slot-recipe).  
And at the same time, We refer to [Vanilla Extract's `styleVariants`](https://vanilla-extract.style/documentation/api/style-variants/) for code style for now. It could be change to [sub functions](https://github.com/mincho-js/mincho/issues/59).

This is a combination of `rules()` and `cssVariants()`.

**Code:**

```typescript
const contents = rulesVariants({
  text: {
    fontWeight: "bold",

    variants: {
      color: {
        main: { color: "#0078e5" },
        sub: { color: "#fff7ed" }
      },
      size: {
        large: { fontSize: "24px" },
        medium: { fontSize: "18px" }
        small: { fontSize: "12px" }
      }
    },

    toggles: {
      accent: { textDecoration: "underline" }
    }
  },
  image: {
    width: '100%',
    border: "1px solid #ccc",
    borderRadius: "5px",

    variants: {
      style: {
        thumbnail: {
          width: "50px"
        },
        detail: {
          width: '80%';
          marginBottom: '10px';
        }
      }
    }
  }
});
```

It helps that we write multiple `rules` at once.

**Compiled:**

```css
.[FILE_NAME]_text__[HASH] {
  font-weight: "bold";
}
.[FILE_NAME]_text_color_main__[HASH] {
  color: "#0078e5";
}
.[FILE_NAME]_text_color_sub__[HASH] {
  color: "#fff7ed";
}
.[FILE_NAME]_text_size_large__[HASH] {
  font-size: "24px";
}
.[FILE_NAME]_text_size_medium__[HASH] {
  font-size: "18px";
}
.[FILE_NAME]_text_size_small__[HASH] {
  font-size: "12px";
}
.[FILE_NAME]_text_toggle_accent__[HASH] {
  text-decoration: "underline";
}

.[FILE_NAME]_image__[HASH] {
  width: "100%";
  border: "1px solid #ccc";
  border-radius: "5px";
}
.[FILE_NAME]_image_style_thumbnail__[HASH] {
  width: "50px";
}
.[FILE_NAME]_image_style_detail__[HASH] {
  width: "80%";
  margin-bottom: "10px";
}
```

**Usage:**

```typescript
// case 1
contents.text([
  "accent",
  {
    color: "sub",
    size: "medium"
  }
]);
// case 2
contents.image({
  style: "thumbnail"
});
```

## 10. With `css()`

Automatically evaluated at composition in `css()`.

```typescript
const base = css({ color: red });
const myRule = rules({
  background: blue,
  props: ["padding"],
  toggles: {
    rounded: { borderRadius: 999 }
  }
});

const myCss = css([base, myRules]); // Same as [base, myRules()]
```

# Reference-level explanation

The default implementation starts with the [Vainilla Extracts's recipes package](https://vanilla-extract.style/documentation/packages/recipes/) [code](https://github.com/vanilla-extract-css/vanilla-extract/tree/master/packages/recipes).

See also [Rainbow Sprinkles](https://github.com/wayfair/rainbow-sprinkles).

Actively use [CSS Variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) for the sake of `props`.

# Drawbacks

Because they must be statically extracted, there are a number of [limitations](https://github.com/mui/pigment-css?tab=readme-ov-file#how-to-guides).

Some restrictions may be lifted in the future when functional `rules()` is implemented.

## Fixed set of styles

Conditional styling with Props is not available.

```typescript
const Flex = rules((props) => ({
  display: "flex",
  ...(props.vertical // ❌ ERROR: Can't evaluate
    ? {
        flexDirection: "column",
        paddingBlock: "1rem"
      }
    : {
        paddingInline: "1rem"
      })
}));
```

Use variants instead.

```typescript
const Flex = rules((props) => ({
  display: "flex",
  variants: {
    vertical: {
      true: {
        flexDirection: "column",
        paddingBlock: "1rem"
      },
      false: {
        paddingInline: "1rem"
      }
    }
  }
}));
```

## Programatically generated styles

You might have styles that are generated at runtime.

```tsx
function randomBetween(min: number, max: number) {
  return Math.floor(Math.random() * (max - min + 1) + min);
}
function generateBubbleVars() {
  return `
    --x: ${randomBetween(10, 90)}%;
    --y: ${randomBetween(15, 85)}%;
  `;
}

function App() {
  return (
    <div>
      {[...Array(10)].map((_, index) => (
        <div
          key={index}
          className={css`
            ${generateBubbleVars()} width: var(--x);
            height: var(--y);
          `}
        />
      ))}
    </div>
  );
}
```

In this case, utilize `props`.

```tsx
// app.css.ts
export const myRule = rules({
  props: {
    x: "width",
    y: "height"
  }
});

// App.tsx
import { myRule } from "app.css";

function randomBetween(min: number, max: number) {
  return Math.floor(Math.random() * (max - min + 1) + min);
}

function App() {
  return (
    <div>
      {[...Array(10)].map((_, index) => (
        <div
          key={index}
          style={myRule.props({
            x: randomBetween(10, 90),
            y: randomBetween(15, 85)
          })}
        />
      ))}
    </div>
  );
}
```

See also [PandaCSS recipes limitations](https://panda-css.com/docs/concepts/recipes#limitations), [Chat about the future for stitches](https://github.com/stitchesjs/stitches/discussions/1149)

# Rationale and alternatives

PandaCSS is implementing very powerful variants feature([Recipes](https://panda-css.com/docs/concepts/recipes), [Slot Recipes](https://panda-css.com/docs/concepts/slot-recipes)).

For runtime props interpolation on the client side, see Fela([`renderRule`](https://fela.js.org/docs/latest/basics/renderer#renderrule), [Combined Rules](https://fela.js.org/docs/latest/advanced/combined-rules)).

# Unresolved questions

While this should be theoretically possible to implement in practice, it can be more time-consuming than you might think.  
If a feature is overly difficult to implement, you may want to implement another RFC first and then try again.

# Future possibilities

## Functional `rules()`

As drafted, functional `rules()` should be possible.

```typescript
inteface MyRulesProps {
  color: string;
  background: string;
  size: number;
}

const myRule = rules<MyRuleProps>(({ color, size }) => ({
  color,
  background: (value = "blue") => value,
  padding: size
}));
```

To achieve this, you might need an AST traversal and a runtime styling.

For runtime style, check the accesses in Kuma UI([1](https://www.kuma-ui.com/docs/Concepts/Hybrid), [2](https://github.com/kuma-ui/kuma-ui/discussions/409)).

## Build time evaluate

Like [Linaria](https://github.com/callstack/linaria/blob/master/docs/HOW_IT_WORKS.md), [Pigment CSS](https://github.com/mui/pigment-css/blob/master/HOW_PIGMENT_CSS_WORKS.md), you can utilize [WyW-in-JS](https://wyw-in-js.dev/) to evaluate javascript at build time.

See also [macaron](https://github.com/macaron-css/macaron).

## Styled Components

It would be nice to be able to generate Styled Components from rules, like [Attributify Mode in Windi CSS](https://windicss.org/features/attributify.html) or [Stitches](https://stitches.dev/docs/api#styled) does.

See also [Dessert Box](https://github.com/TheMightyPenguin/dessert-box), [styled-extract](https://github.com/Slooowpoke/styled-extract), [react-styled-factory](https://github.com/kasper573/react-styled-factory).

## Atomic Styles

This library was born for the combination of `Atomic CSS` and `CSS Variants`.
This will be the next RFC.
