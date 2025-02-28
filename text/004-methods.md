- Start Date: 2025-02-15

# Summary
[summary]: #summary

Designed to reduce API surface area and achieve isomorphism.

# Motivation
[motivation]: #motivation

We attach great importance to isomorphism in our APIs.  
This is because we fundamentally believe that it is possible, and because the repetitive structure greatly increases user predictability.

Some of the repeating structures are:
```typescript
// cssVariants
const background = cssVariants({
  primary: { background: "blue" },
  secondary: { background: "aqua" }
});

// rulesVariants
const contents = rulesVariants({
  text: {
    fontWeight: "bold",
    toggles: {
      accent: { textDecoration: "underline" }
    }
  },
  image: {
    width: "100%",
    border: "1px solid #ccc",
    borderRadius: "5px",

    variants: {
      style: {
        thumbnail: {
          width: "50px"
        },
        detail: {
          width: "80%",
          marginBottom: "10px"
        }
      }
    }
  }
});
```

In the above example, the `variants` is repeated several times, causing users to worry about whether it is appropriate for use.

We want to prevent confusion between terms that appear similar.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## 1. Raw

Inspired by the [PandaCSS's `raw()`](https://panda-css.com/docs/concepts/merging-styles#merging-css-objects)

It only checks the type and returns the Object as is.

- Targets: `css`, `rules`, `theme`

```typescript
const myCssObj = css.raw({
  color: "blue",
  backgroundColor: "#EEEEEE"
});

const myRuleObj = rules.raw({
  borderRadius: 8,
  props: ["color", "background"],
  toggles: {
    rounded: { borderRadius: 999 }
  }
});
```

If co-location is used, semantics such as PandaCSS may be added.

## 2. With

Inspired by the [TSS-React's `withParams()`](https://docs.tss-react.dev/api/tss-usestyles#tss.withparams), [SCSS's @mixin](https://sass-lang.com/documentation/at-rules/mixin/).

It constrains the type of params.

- Targets: `css`, `rules`, `theme`

```typescript
const myCss = css.with<{ color: true, background: "blue" | "grey" }>();
myCss({
  color: "red", // Allow all properties
  background: "blue" // Only some properties are allowed
  border: "none" // Error!!
});
```

### Mixin

It can also act as a mixin.  
If the type is difficult to implement, it can be separated and explicitly called a mixin.
```typescript
const myCss = css.with<{ size: number, radius: number }>(({ size, radius = 0 }) => {
  const styles = {
    width: size,
    height: size
  };

  if (radius !== 0) {
    styles.borderRadius = radius;
  }

  return styles;
});
```

### Type

We can also provide type tools like stylex's [`StyleXStyles`](https://stylexjs.com/docs/learn/static-types/#constraining-accepted-styles).

```typescript
type RestrictedCSSRule = CSSRuleWith<{
  // 1. Allowed Props ((required unless marked optional)
  paddingTop: true; // All value allowed; required
  marginTop: 0 | 4 | 8 | 16; // Only these values allowed; required
  paddingBottom?: true; // All value allowed; optional
  marginBottom?: 0 | 4 | 8 | 16; // Only these values allowed; optional

  // 2. Forbidden Props
  position: false; // Forbidden prop
}>;
```

### Theme

Theme should be implemented similarly to [Vanilla Extract's `createThemeContract()`](https://vanilla-extract.style/documentation/api/create-theme-contract/).

```typescript
const myTheme = theme.with<{
  color: {
    brand: string;
  },
  font: {
    body: string;
  }
}>();

const themeA = myTheme({
  color: {
    brand: "blue"
  },
  font: {
    body: "arial"
  }
});
```

### Methods

The results can utilize methods such as `raw()` and `multiple()`.

```typescript
const myCss = css.with<{ color: true, background: "blue" | "grey" }>();
myCss.raw({
  color: "red", // Allow all properties
  background: "blue" // Only some properties are allowed
});
```

## 3. Multiple

As mentioned in Motivation, the `variants` function is being repeated.

The biggest problem is that the names overlap with the Rule Variants, causing confusion.

Among these, `cssVariants` and `rulesVariants` are both methods of creating multiple rules and styles at once, so how about `multiple()`.

- Targets: `css`, `rules`, `theme`

```typescript
css.multiple();
rules.multiple();
theme.multiple();
```

## 4. Compatibility

Separate Vanilla Extract's API.  
This is a way to reduce confusion while providing a more consistent and simple API.

### Path

Export to `/compat`.

### Aliased

The following are comparable APIs:

1. `css()`: `style()`
2. `css.multiple()`: `styleVariants()`
3. `rules()`: `recipe()`
4. `theme()`: `createTheme()`
5. `theme.with()`: `createThemeContract()`

APIs that are currently simply aliased will be included in `/compat` and deleted.

### Breakings

Generally, we just need to be aliased, but there may be breaking changes in some APIs.

- rules: Unlike recipe, `condition` is used.
```typescript
// Before
const button = rules({
  compoundVariants: [
    {
      variants: {
        color: "brand",
        size: "small",
      },
      style: {
        fontSize: "16px"
      }
    }
  ]
});

// After
const button = rules({
  compoundVariants: [
    {
      condition: {
        color: "brand",
        size: "small",
      },
      style: {
        fontSize: "16px"
      }
    }
  ]
});
```

- theme, theme.with: The way you use it changes.
```typescript
// Before
const vars = createThemeContract({
  color: {
    brand: null
  },
  font: {
    body: null
  }
});

const themeA = createTheme(vars, {
  color: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});

// After
const myTheme = theme.with<{
  color: {
    brand: string;
  },
  font: {
    body: string;
  }
}>();

const [themeClass, vars] = myTheme({
  color: {
    brand: "blue"
  },
  font: {
    body: "arial"
  }
});
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Path

Additional `package.json` and bundler settings are required.

## With method

If you want to have both a function and a method exist at the same time, you can use [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign).

```typescript
const css = Object.assign(cssImpl, {
  raw: cssRaw,
  with: cssWith,
  multiple: cssMultiple
});
```

# Drawbacks
[drawbacks]: #drawbacks

Drop in may be inconvenient for users who want to replace it.  
However, it is clear that this approach is more correct over time.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Ensure that the current API is backwards compatible with the Vanilla Extract API.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

There may be some problems on the theme side.

# Future possibilities
[future-possibilities]: #future-possibilities

I will revisit this after creating the `defineRules` RFC.
