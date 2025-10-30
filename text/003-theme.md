
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

## 3. Structural Token

Inspired by [W3C Token Format](https://tr.designtokens.org/format/), each token can contain more specific information.

If `$value` is included, it is recognized as a Token.

**Code:**

```typescript
const [themeClass, vars] = theme({
  myColor: {
    $description: "My brand color",
    $type: "color",
    $value: {
      colorSpace: "srgb",
      components: [1, 0, 1],
      alpha: 1,
      hex: "#ff00ff"
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --myColor__[HASH]: #ff00ff;
}
```

## 4. Reference

Created as a CSS Variable, but can be referenced.

You can easily reference CSS Variables using [`get`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) and [`this`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this).

Limitation: TypeScript's inference does not work as expected with [`ThisType`](https://www.typescriptlang.org/docs/handbook/utility-types.html#thistypetype), but instead infers it to `any`, so you must specify the return type. 
- See: [TypeScript#47150](https://github.com/microsoft/TypeScript/issues/47150), [TypeScript#52969](https://github.com/microsoft/TypeScript/issues/52969)

**Code:**

```typescript
theme({
  spaces: {
    base: [0, 2, 4, 8, 16, 32, 64],
    semantic: {
      get medium(): string {
        return this.spaces.base[4]; // var(--spaces-base-4__[HASH])
      }
    }
  },
  colors: {
    base: {
      "black-100": "#000",
      "white-100": "#fff",
      "blue-500": "#07c"
    },
    semantic: {
      get primary(): string {
        return this.colors.base["blue-500"]; // var(--colors-base-blue-500__[HASH])
      } 
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-base-0__[HASH]: 0;
  --spaces-base-1__[HASH]: 2;
  --spaces-base-2__[HASH]: 4;
  --spaces-base-3__[HASH]: 8;
  --spaces-base-4__[HASH]: 16;
  --spaces-base-5__[HASH]: 32;
  --spaces-base-6__[HASH]: 64;
  --colors-base-black-100__[HASH]: #000;
  --colors-base-white-100__[HASH]: #fff;
  --colors-base-blue-500__[HASH]: #07c;

  /* New Style */
  --spaces-semantic-medium__[HASH]: var(--spaces-base-4__[HASH]);
  --colors-semantic-primary__[HASH]: var(--colors-base-blue-500__[HASH]);
}
```

## 5. Composite Token

Inspired by [W3C composite token](https://tr.designtokens.org/format/#composite-types), but simplified.

**Code:**

```typescript
theme({
  shadow: {
    light: {
      get resolved(): string {
        return `${this.shadow.light.color} ${this.shadow.light.offsetX} ${this.shadow.light.offsetY} ${this.shadow.light.blur}`;
      },
      color: "#00000080",
      offsetX: { value: 0.5, unit: "rem" },
      offsetY: { value: 0.5, unit: "rem" },
      blur: { value: 1.5, unit: "rem" }
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --shadow-light__[HASH]: var(--shadow-light-color__[HASH]) var(--shadow-light-offsetX__[HASH]) var(--shadow-light-offsetY__[HASH]) var(--shadow-light-blur__[HASH]);
  --shadow-light-color__[HASH]: #00000080;
  --shadow-light-offsetX__[HASH]: "0.5rem";
  --shadow-light-offsetY__[HASH]: "0.5rem";
  --shadow-light-blur__[HASH]: "1.5rem";
}
```

**Code:**

```typescript
theme({
  shadow: {
    light: theme.compositeValue({
      get resolved(): string {
        return `${this.color} ${this.offsetX} ${this.offsetY} ${this.blur}`;
      },
      color: "#00000080",
      offsetX: { value: 0.5, unit: "rem" },
      offsetY: { value: 0.5, unit: "rem" },
      blur: { value: 1.5, unit: "rem" }
    })
  }
});
```


## 6. Aliased

Aliased only on Javascript objects.

This only references another Variable without creating a CSS Variable.

**Code:**

```typescript
theme({
  spaces: {
    base: [0, 2, 4, 8, 16, 32, 64],
    semantic: {
      get medium(): string {
        return this.alias(this.spaces.base[4]); // var(--spaces-base-4__[HASH])
      }
    }
  },
  colors: {
    base: {
      "black-100": "#000",
      "white-100": "#fff",
      "blue-500": "#07c"
    },
    semantic: {
      get primary(): string {
        return this.alias(this.colors.base["blue-500"]); // var(--colors-base-blue-500__[HASH])
      } 
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-base-0__[HASH]: 0;
  --spaces-base-1__[HASH]: 2;
  --spaces-base-2__[HASH]: 4;
  --spaces-base-3__[HASH]: 8;
  --spaces-base-4__[HASH]: 16;
  --spaces-base-5__[HASH]: 32;
  --spaces-base-6__[HASH]: 64;
  --colors-base-black-100__[HASH]: #000;
  --colors-base-white-100__[HASH]: #fff;
  --colors-base-blue-500__[HASH]: #07c;

  /* New Style is None */
}
```

## 7. Raw

It returns the actual value, not a CSS Var.

**Code:**

```typescript
theme({
  spaces: {
    base: [0, 2, 4, 8, 16, 32, 64],
    semantic: {
      get medium(): number {
        return this.raw(this.spaces.base[4]); // 16
      }
    }
  },
  colors: {
    base: {
      "black-100": "#000",
      "white-100": "#fff",
      "blue-500": "#07c"
    },
    semantic: {
      get primary(): string {
        return this.raw(this.colors.base["blue-500"]); // #07c
      } 
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-base-0__[HASH]: 0;
  --spaces-base-1__[HASH]: 2;
  --spaces-base-2__[HASH]: 4;
  --spaces-base-3__[HASH]: 8;
  --spaces-base-4__[HASH]: 16;
  --spaces-base-5__[HASH]: 32;
  --spaces-base-6__[HASH]: 64;
  --colors-base-black-100__[HASH]: #000;
  --colors-base-white-100__[HASH]: #fff;
  --colors-base-blue-500__[HASH]: #07c;

  /* New Style */
  --spaces-semantic-medium__[HASH]: 16;
  --colors-semantic-primary__[HASH]: #07c;
}
```

## 8. Fallback

Fallback is easy to do.

**Code:**

```typescript
theme({
  spaces: {
    base: [0, 2, 4, 8, 16, 32, 64],
    semantic: {
      get medium(): string {
        return this.fallbackVar(this.spaces.base[4], 12); // var(var(--spaces-base-4__[HASH]), 12)
      }
    }
  },
  colors: {
    base: {
      "black-100": "#000",
      "white-100": "#fff",
      "blue-500": "#07c"
    },
    semantic: {
      get primary(): string {
        return this.fallbackVar(this.colors.base["blue-500"]); // var(var(--colors-base-blue-500__[HASH]), "blue")
      } 
    }
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --spaces-base-0__[HASH]: 0;
  --spaces-base-1__[HASH]: 2;
  --spaces-base-2__[HASH]: 4;
  --spaces-base-3__[HASH]: 8;
  --spaces-base-4__[HASH]: 16;
  --spaces-base-5__[HASH]: 32;
  --spaces-base-6__[HASH]: 64;
  --colors-base-black-100__[HASH]: #000;
  --colors-base-white-100__[HASH]: #fff;
  --colors-base-blue-500__[HASH]: #07c;

  /* New Style */
  --spaces-semantic-medium__[HASH]: var(var(--spaces-base-4__[HASH]), 12);
  --colors-semantic-primary__[HASH]: var(var(--colors-base-blue-500__[HASH]), blue);
}
```

## 9. Contract

Create derived themes according to the contract.
[Vanilla Extract's `createTheme()`](https://vanilla-extract.style/documentation/theming#code-splitting-themes) is already doing a good job, so keep it.

**Code:**

```typescript
const [themeClass, vars] = theme({
  colors: {
    brand: "blue"
  },
  font: {
    body: "arial"
  }
});

const otherThemeClass = theme(vars, {
  colors: {
    brand: "red"
  },
  font: {
    body: "helvetica"
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_themeClass__[HASH] {
  --colors-brand__[HASH]: blue;
  --font-body__[HASH]: "arial";
}

.[FILE_NAME]_otherThemeClass__[HASH] {
  --colors-brand__[HASH]: red;
  --font-body__[HASH]: "helvetica";
}
```

Can just set the contract and postpone CSS generation.

```typescript
const themeContract = themeContract({
  colors: {
    brand: ''
  },
  font: {
    body: ''
  }
});

const blueThemeClass = theme(vars, {
  colors: {
    brand: 'blue'
  },
  font: {
    body: 'arial'
  }
});
```

**Compiled:**

```css
.[FILE_NAME]_blueThemeClass__[HASH] {
  --colors-brand__[HASH]: blue;
  --font-body__[HASH]: "arial";
}
```

## 10. Global Theme

Like other APIs, there is a Global API.

**Code:**

```typescript
export const vars = globalTheme(':root', {
  colors: {
    brand: "blue"
  },
  font: {
    body: "arial"
  }
});
```

**Compiled:**

```css
:root {
  --colors-brand__[HASH]: blue;
  --font-body__[HASH]: "arial";
}
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Getters and Methods

You can use [getters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) and [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) to dynamically calculate values ​​from properties, or use methods. \
TypeScript's [ThisType](https://www.typescriptlang.org/docs/handbook/utility-types.html#thistypetype) allows you to control the type inferred from this.

```typescript
interface GreetingMixin {
  greeting(name: string, age: number): string;
}

// `withGreeting()` injects the greeting method into the target object and returns the injected type.
function withGreeting<TTarget extends object>(
  target: TTarget & ThisType<TTarget & GreetingMixin>
): TTarget & GreetingMixin {
  const enhancedTarget = target as TTarget & GreetingMixin;
  
  enhancedTarget.greeting = function (name: string, age: number): string {
    return `Hello, I'm ${name} and I'm ${age} years old!`;
  };
  
  return enhancedTarget;
}

// Example usage
const personWithGreeting = withGreeting({
  name: "John",
  age: 30,
  otherInfo: {
    hobby: "reading",
    city: "New York"
  },
  get casualGreeting() {
    console.log(this.otherInfo.city);
    // Thanks to ThisType, this here is inferred to be TTarget & GreetingMixin
    return this.greeting(this.name, this.age);
  }
});

console.log(personWithGreeting.casualGreeting); // "Hello, I'm John and I'm 30 years old!"
```

# Drawbacks
[drawbacks]: #drawbacks

To keep the top-level as much as possible, I used special attributes starting with `$`.

Since it is somewhat heterogeneous, I am concerned about whether it is the right direction.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

## Provider

We could supply theme values ​​as Context like in [emotion](https://emotion.sh/docs/theming), but it is not an option because it is Near zero runtime.

## Config

Like [pandacss](https://panda-css.com/docs/theming/tokens), theme values ​​can be defined as Config, but it is oriented toward a general function.

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

# Unresolved questions
[unresolved-questions]: #unresolved-questions

## Conditional token

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
