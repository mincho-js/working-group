
- Start Date: 2025-02-08

# Summary
[summary]: #summary

Ability to create and use design tokens according to contracts.

# Motivation
[motivation]: #motivation

- Theme contract
- Design tokens

## What is theme?

A theme is the application of styling, such as color or image, according to specific rules.

- [light](https://github.com/mozilla/gecko-dev/blob/91030e74f3e9bb2aac9fe2cbff80734c6fd610b9/browser/themes/addons/light/manifest.json)
- [dark](https://github.com/mozilla/gecko-dev/blob/91030e74f3e9bb2aac9fe2cbff80734c6fd610b9/browser/themes/addons/dark/manifest.json)
- [alpenglow](https://github.com/mozilla/gecko-dev/blob/91030e74f3e9bb2aac9fe2cbff80734c6fd610b9/browser/themes/addons/alpenglow/manifest.json)

```json
{
"name": "Light",
  "description": "A theme with a light color scheme.",
  "author": "Mozilla",
  "version": "1.3",

  "icons": { "32": "icon.svg" },

  "theme": {
    "colors": {
      "tab_background_text": "rgb(21,20,26)",
      "tab_selected": "#fff",
      "tab_text": "rgb(21,20,26)",
      "icons": "rgb(91,91,102)",
      "frame": "rgb(240, 240, 244)",
      "frame_inactive": "rgb(235, 235, 239)",
      "popup": "#fff",
      "popup_text": "rgb(21,20,26)",
      "popup_border": "rgb(240,240,244)",
      "popup_highlight": "#e0e0e6",
      "popup_highlight_text": "#15141a",
      "sidebar": "rgb(255, 255, 255)",
      "sidebar_text": "rgb(21, 20, 26)",
      "sidebar_border": "rgb(240, 240, 244)",
      "tab_line": "transparent",
      "toolbar": "#f9f9fb",
      "toolbar_top_separator": "transparent",
      "toolbar_bottom_separator": "rgb(240, 240, 244)",
      "toolbar_field": "rgba(0, 0, 0, .05)",
      "toolbar_field_text": "rgb(21, 20, 26)",
      "toolbar_field_border": "transparent",
      "toolbar_field_focus": "white",
      "toolbar_text": "rgb(21,20,26)",
      "ntp_background": "#F9F9FB",
      "ntp_text": "rgb(21, 20, 26)",
      "popup_action_color": "rgb(91,91,102)",
      "button": "rgba(207,207,216,.33)",
      "button_hover": "rgba(207,207,216,.66)",
      "button_active": "rgb(207,207,216)",
      "button_primary": "rgb(0, 97, 224)",
      "button_primary_hover": "rgb(2, 80, 187)",
      "button_primary_active": "rgb(5, 62, 148)",
      "button_primary_color": "rgb(251, 251, 254)",
      "input_color": "rgb(21,20,26)",
      "input_background": "rgb(255,255,255)",
      "urlbar_popup_hover": "rgb(240,240,244)",
      "urlbar_popup_separator": "rgb(240,240,244)"
    },
    "properties": {
      "color_scheme": "light",
      "toolbar_field_icon_opacity": "0.72",
      "zap_gradient": "linear-gradient(90deg, #9059FF 0%, #FF4AA2 52.08%, #FFBD4F 100%)"
    }
  }
}
```

### What is a design tokens?

Design tokens store repetitive design decisions as a single source of truth.

Simply put, it's like saving a [color palette](https://dev.epicgames.com/community/learning/tutorials/vrZk/unreal-engine-copy-and-paste-your-color-theme-for-the-color-picker-from-one-project-to-another) from a vast color space and using it consistently across the project.

![Color palette](https://github.com/user-attachments/assets/ca0dec2c-2516-4d45-bf5d-5ce7a94027c2)

However, design tokens are not just simple 'settings'. They need to be set at both the app and component levels, and they function similarly to reactivity.

![Design token](https://github.com/user-attachments/assets/268d0aa4-e677-44fb-9921-22752ffd43e8)

1. Primitive tokens: Define the most basic design values such as colors, fonts, sizes, spacing, etc.
   - These have the lowest level of abstraction and contain specific values.
   - Example: `color-blue-500: #0000FF`, `font-size-16: 16px`
2. Semantic tokens: Semantic tokens are tokens that give meaning based on primitive tokens.
   - They use abstracted names that represent the purpose or intention of the design.
   - They are defined by referencing primitive tokens.
   - Example: `color-primary: $color-blue-500`, `font-size-body: $font-size-16`
3. Component tokens: Component tokens define design values that are applied to specific UI components.
   - These are tokens that are directly applied to actual UI components.
   - They are mainly defined by referencing semantic tokens. (Sometimes primitive tokens can also be directly referenced)
   - Example: `button-primary-color: $color-primary`

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## 1. Name

The API is named `theme()`.

It is a simple and easy to distinguish name.

## 2. Basic Usage

Just define it as the object shape you want.
This will create a [CSS Variable](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties).

**Code:**

```typescript
const [themeClass, vars] = theme({
  spaces: [0, 2, 4, 8, 16, 32, 64],
  colors: {
    "black-100": "#000",
    "white-100": "#fff",
    "blue-500": "#07c"
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-0__[HASH]: 0;
  --spaces-1__[HASH]: 2;
  --spaces-2__[HASH]: 4;
  --spaces-3__[HASH]: 8;
  --spaces-4__[HASH]: 16;
  --spaces-5__[HASH]: 32;
  --spaces-6__[HASH]: 64;
  --colors-black-100__[HASH]: #000;
  --colors-white-100__[HASH]: #fff;
  --colors-blue-500__[HASH]: #07c;
}
```

- `themeClass` contains the class name.
- `vars` has the same structure but includes a CSS Variable.

```typescript
console.log(themeClass);
// "[FILE_NAME]_themeClass__[HASH]"

console.log(vars);
/*
 * {
 *   spaces: [
 *     "var(--spaces-0__[HASH])",
 *     "var(--spaces-1__[HASH])",
 *     "var(--spaces-2__[HASH])",
 *     "var(--spaces-3__[HASH])",
 *     "var(--spaces-4__[HASH])",
 *     "var(--spaces-5__[HASH])",
 *     "var(--spaces-6__[HASH])"
 *   ],
 *   colors: {
 *     "black-100": "var(--colors-black-100__[HASH])",
 *     "white-100": "var(--colors-white-100__[HASH])",
 *     "blue-500": "var(--colors-blue-500__[HASH])"
 *   }
 * }
 */
```

### 3. Token

Inspired by [W3C Token Format](https://tr.designtokens.org/format/), each token can contain more specific information.

If `$value` is included, it is recognized as a Token.

**Code:**

```typescript
const [themeClass, vars] = theme({
  myColor: {
    $description: "My brand color",
    $type: "color",
    $value: "red"
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --myColor__[HASH]: red;
}
```

### 4.Conditional token

If `$base` or `@at-rules` is included, it is recognized as a value.

Not supported with top level.

**Code:**

```typescript
const [themeClass, vars] = theme({
  colors: {
    "black-100": {
      $base: "#000",
      "@media (prefers-color-scheme: dark)": "#111"
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --colors-black-100__[HASH]: #000;
}

@media (prefers-color-scheme: dark) {
  .[FILE_NAME]_themeClass__[HASH] {
    --colors-black-100__[HASH]: #111;
  }
}
```

### 5. Reference

Created as a CSS Variable, but can be referenced.

It receives a theme object as the first argument, and an object containing `raw` and `fallback` functions as the second argument.

**Code:**

```typescript
theme({
  spaces: [0, 2, 4, 8, 16, 32, 64],
  colors: {
    "black-100": "#000",
    "white-100": "#fff",
    "blue-500": "#07c"
  },

  $refer: (theme) => ({
    spaces: {
      medium: spaces[4]
    },
    colors: {
      primary: theme.colors["blue-500"]
    }
  })
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-0__[HASH]: 0;
  --spaces-1__[HASH]: 2;
  --spaces-2__[HASH]: 4;
  --spaces-3__[HASH]: 8;
  --spaces-4__[HASH]: 16;
  --spaces-5__[HASH]: 32;
  --spaces-6__[HASH]: 64;
  --colors-black-100__[HASH]: #000;
  --colors-white-100__[HASH]: #fff;
  --colors-blue-500__[HASH]: #07c;

  /* New Style */
  --spaces-medium__[HASH]: var(--spaces-4__[HASH]);
  --colors-primary__[HASH]: var(--colors-blue-500__[HASH]);
}
```

### 6. Aliased

Aliased only on Javascript objects.

It receives a theme object as the first argument, and an object containing `raw` and `fallback` functions as the second argument.

**Code:**

```typescript
theme({
  spaces: [0, 2, 4, 8, 16, 32, 64],
  colors: {
    "black-100": "#000",
    "white-100": "#fff",
    "blue-500": "#07c"
  },

  $alias: (theme) => ({
    spaces: {
      medium: spaces[4]
    },
    colors: {
      primary: theme.colors["blue-500"]
    }
  })
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-0__[HASH]: 0;
  --spaces-1__[HASH]: 2;
  --spaces-2__[HASH]: 4;
  --spaces-3__[HASH]: 8;
  --spaces-4__[HASH]: 16;
  --spaces-5__[HASH]: 32;
  --spaces-6__[HASH]: 64;
  --colors-black-100__[HASH]: #000;
  --colors-white-100__[HASH]: #fff;
  --colors-blue-500__[HASH]: #07c;

  /* New Style is None */
}
```

### 7. Raw

It returns the actual value, not a CSS Var.

**Code:**

```typescript
theme({
  spaces: [0, 2, 4, 8, 16, 32, 64],
  colors: {
    "black-100": "#000",
    "white-100": "#fff",
    "blue-500": "#07c"
  },

  $refer: (theme, { raw }) => ({
    spaces: {
      medium: raw(spaces[4])
    },
    colors: {
      primary: raw(theme.colors["blue-500"])
    }
  })
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-0__[HASH]: 0;
  --spaces-1__[HASH]: 2;
  --spaces-2__[HASH]: 4;
  --spaces-3__[HASH]: 8;
  --spaces-4__[HASH]: 16;
  --spaces-5__[HASH]: 32;
  --spaces-6__[HASH]: 64;
  --colors-black-100__[HASH]: #000;
  --colors-white-100__[HASH]: #fff;
  --colors-blue-500__[HASH]: #07c;

  /* New Style */
  --spaces-medium__[HASH]: 16;
  --colors-primary__[HASH]: #07c;
}
```

### 8. Fallback

Fallback is easy to do.

```typescript
theme({
  spaces: [0, 2, 4, 8, 16, 32, 64],
  colors: {
    "black-100": "#000",
    "white-100": "#fff",
    "blue-500": "#07c"
  },

  $refer: (theme, { fallback }) => ({
    spaces: {
      medium: fallback(spaces[4], 12)
    },
    colors: {
      primary: fallback(theme.colors["blue-500"], "blue")
    }
  })
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-0__[HASH]: 0;
  --spaces-1__[HASH]: 2;
  --spaces-2__[HASH]: 4;
  --spaces-3__[HASH]: 8;
  --spaces-4__[HASH]: 16;
  --spaces-5__[HASH]: 32;
  --spaces-6__[HASH]: 64;
  --colors-black-100__[HASH]: #000;
  --colors-white-100__[HASH]: #fff;
  --colors-blue-500__[HASH]: #07c;

  /* New Style */
  --spaces-medium__[HASH]: var(var(--spaces-4__[HASH]), 12);
  --colors-primary__[HASH]: var(var(--colors-blue-500__[HASH]), blue);
}
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Hybrid merge

How should I use it when an array and an object exist at the same time as shown below?

```typescript
theme({
  spaces: [0, 2, 4, 8, 16, 32, 64],

  $refer: (theme, { raw }) => ({
    spaces: {
      medium: raw(spaces[4])
    }
  })
});
```

Usually you will want to use them simultaneously, like `spaces[3]` and `spaces["medium"]`.

To achieve this we need to use a proxy.

```javascript
function createHybridData(array, object) {
  const data = {
    array,
    object
  };

  return new Proxy(data, {
    get(target, prop) {
      if (!isNaN(prop)) {
        return target.array[prop];
      }
      return target.object[prop];
    },

    set(target, prop, value) {
      if (!isNaN(prop)) {
        target.array[prop] = value;
      } else {
        target.object[prop] = value;
      }
      return true;
    },


    ownKeys(target) {
      return [
        ...Object.keys(target.array),
        ...Object.keys(target.object)
      ];
    },

    getOwnPropertyDescriptor(target, prop) {
      if (!isNaN(prop)) {
        return {
          value: target.array[prop],
          writable: true,
          enumerable: true,
          configurable: true
        };
      }
      return {
        value: target.object[prop],
        writable: true,
        enumerable: true,
        configurable: true
      };
    }
  });
}
```

Now you can use it like this:

```javascript
const arr = ['a', 'b', 'c'];
const obj = { x: 1, y: 2 };
const result = createHybridData(arr, obj);

// hybrid data
result[0]; //'a'
result["x"]; //1

// result proxy object
Proxy [
  { array: [ 'a', 'b', 'c' ], object: { x: 1, y: 2 } },
  {
    get: [Function: get],
    set: [Function: set],
    ownKeys: [Function: ownKeys],
    getOwnPropertyDescriptor: [Function: getOwnPropertyDescriptor]
  }
]
```

I expect customization based on [fastify/deepmerge](https://github.com/fastify/deepmerge).

# Drawbacks
[drawbacks]: #drawbacks

To keep the toplevel as much as possible, I used special attributes starting with `$`.

Since it is somewhat heterogeneous, I am concerned about whether it is the right direction.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

## Provider

We could supply theme values ​​as Context like in [emotion](https://emotion.sh/docs/theming), but it is not an option because it is Near zero runtime.

## Config

Like [pandacss](https://panda-css.com/docs/theming/tokens), theme values ​​can be defined as Config, but it is oriented toward a general function.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

## Composite token

There is a simple way to define [W3C composite token](https://tr.designtokens.org/format/#composite-types), but I can't think of a simple way to use it.

The definition only needs to allow composite tokens within `$value`.

```typescript
theme({
  shadow-token: {
    $type: "shadow",
    $value: {
      color: "#00000080",
      offsetX: { value: 0.5, unit: "rem" },
      offsetY: { value: 0.5, unit: "rem" },
      blur: { value: 1.5, unit: "rem" },
      spread: { value: 0, unit: "rem" }
    }
  }
});
```

## Constants

There may be values ​​that exist only in Javascript and not in CSS.

```typescript
// No create CSS
theme({
  $constants: {
    spaces: [0, 2, 4, 8, 16, 32, 64],
    colors: {
      "black-100": "#000",
      "white-100": "#fff",
      "blue-500": "#07c"
    },
  }
});
```

# Future possibilities
[future-possibilities]: #future-possibilities

## API Isomorphism

We will need to add a few more features for API isomorphism.

- Composition
- Variants

I will likely add to this RFC.

I am thinking about a simple API to use with [`createThemeContract`](https://vanilla-extract.style/documentation/api/create-theme-contract/).
