- Start Date: 2025-06-21

# Summary
[summary]: #summary

Build to handle classNames consistently.

# Motivation
[motivation]: #motivation

The `compoundVariants` of [rules](./002-css-rules.md) and the isomorphic APIs of [methods](./004-methods.md) are not yet supported by current libraries.

You need to keep the same mental model and make it more intuitive and predictable.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## 1. Name

The API is named `cx()`.

It's concise and easy to type.

Inspired by [Emotion's `cx()`](https://emotion.sh/docs/@emotion/css#cx).

## 2. Conditional class names

Inspired by [clsx](https://github.com/lukeed/clsx).

```typescript
// Strings (variadic)
cx("foo", true && "bar", "baz");
//=> "foo bar baz"

// Objects
cx({ foo: true, bar: false, baz: isTrue() });
//=> 'foo baz'

// Objects (variadic)
cx({ foo: true }, { bar: false }, null, { baz: "hello" });
//=> 'foo baz'

// Arrays
cx(["foo", 0, false, "bar"]);
//=> "foo bar"

// Arrays (variadic)
cx(["foo"], ["", 0, false, "bar"], [["baz", [["hello"], "there"]]]);
//=> "foo bar baz hello there"

// Kitchen sink (with nesting)
cx("foo", [1 && "bar", { baz: false, bat: null }, ["hello", ["world"]]], "cya");
//=> "foo bar hello world cya"
```

## 3. Variants

Inspired by [cva](https://cva.style/docs).

```typescript
const button = cva({
  base: "rounded border font-semibold",
  // **or**
  // base: ["font-semibold", "border", "rounded"],
  variants: {
    intent: {
      primary: "border-transparent bg-blue-500 text-white hover:bg-blue-600",
      // **or**
      // primary: [
      //   "bg-blue-500",
      //   "text-white",
      //   "border-transparent",
      //   "hover:bg-blue-600",
      // ],
      secondary: "border-gray-400 bg-white text-gray-800 hover:bg-gray-100",
    },
    size: {
      small: "px-2 py-1 text-sm",
      medium: "px-4 py-2 text-base",
    },
  },
  compoundVariants: ({ intent, size }) => [
    {
      condition: [intent.primary, size.medium],
      class: "uppercase",
      // **or** if you're a React.js user, `className` may feel more consistent:
      // className: "uppercase"
    },
  ],
  defaultVariants: {
    intent: "primary",
    size: "medium",
  },
});

button();
// => "font-semibold border rounded bg-blue-500 text-white border-transparent hover:bg-blue-600 text-base py-2 px-4 uppercase"

button({ intent: "secondary", size: "small" });
// => "font-semibold border rounded bg-white text-gray-800 border-gray-400 hover:bg-gray-100 text-sm py-1 px-2"
```

You can compose it using `class` or `className` when using it. \
It behaves the same as `clsx`.

```typescript
button({ class: "m-4" });
// => "…buttonClasses m-4"

button({ className: "m-4" });
// => "…buttonClasses m-4"
```

## 4. Raw

Inspired by the methods RFC's `raw()` pattern, we provide type-checked object returns.

### cx.raw ❌

**Not provided.** `cx.raw()` is **not available** because `cx()` itself already returns a plain string without any transformation. The `cx()` function is essentially a raw operation that combines classNames, so an additional `.raw()` method would be redundant.

```typescript
// cx() already returns a raw string
const classes = cx("foo", "bar");
// => "foo bar" (plain string)

// No need for:
// cx.raw("foo", "bar") ❌
```

### cva.raw()

Returns the cva configuration object as-is, with type checking:

```typescript
const buttonConfig = cva.raw({
  base: "rounded border font-semibold",
  variants: {
    intent: {
      primary: "bg-blue-500 text-white",
      secondary: "bg-gray-500 text-white"
    },
    size: {
      small: "px-2 py-1 text-sm",
      medium: "px-4 py-2 text-base"
    }
  },
  defaultVariants: {
    intent: "primary",
    size: "medium"
  }
});

// buttonConfig is a plain object, not a function
// Useful for passing configuration to other utilities or for composition
```

This is useful when you want to:
- Extract and reuse variant configurations
- Compose multiple configurations before creating a cva instance
- Pass configuration objects to framework-specific utilities

## 5. Multiple

Similar to `css.multiple()` and `rules.multiple()` from the methods RFC, we can create multiple className utilities at once.

This is useful when you want to define a set of related className utilities that share a common pattern.

### cx.multiple()

Creates multiple conditional className utilities from a map:

```typescript
const textStyles = cx.multiple({
  primary: ["text-blue-500", "font-bold"],
  secondary: ["text-gray-500", "font-normal"],
  danger: ["text-red-500", "font-semibold"]
});

// Usage
textStyles.primary;
// => "text-blue-500 font-bold"

textStyles.secondary;
// => "text-gray-500 font-normal"
```

You can also pass objects or arrays with conditional logic, just like `cx()`:

```typescript
const isActive = true;
const hasError = false;

const buttonStates = cx.multiple({
  // Objects with conditions
  primary: { 
    "bg-blue-500": true, 
    "text-white": true, 
    "opacity-50": !isActive 
  },
  // Arrays with conditions
  secondary: [
    "bg-gray-500",
    isActive && "hover:bg-gray-600",
    hasError && "border-red-500"
  ],
  // Mixed: combining strings, objects, and arrays
  danger: [
    "bg-red-500",
    { "font-bold": isActive, "italic": hasError },
    !hasError && "shadow-md"
  ]
});

// Usage
buttonStates.primary;
// => "bg-blue-500 text-white"

buttonStates.secondary;
// => "bg-gray-500 hover:bg-gray-600"

buttonStates.danger;
// => "bg-red-500 font-bold shadow-md"
```

### cva.multiple()

Creates multiple variant utilities from a map:

```typescript
const buttons = cva.multiple({
  primary: {
    base: "rounded border",
    variants: {
      size: {
        small: "px-2 py-1 text-sm",
        large: "px-4 py-2 text-lg"
      }
    }
  },
  secondary: {
    base: "rounded border-2",
    variants: {
      size: {
        small: "px-2 py-1 text-sm",
        large: "px-4 py-2 text-lg"
      }
    }
  }
});

// Usage
buttons.primary({ size: "small" });
// => "rounded border px-2 py-1 text-sm"

buttons.secondary({ size: "large" });
// => "rounded border-2 px-4 py-2 text-lg"
```

## 6. With

Similar to `css.with()` and `rules.with()` from the methods RFC, we can constrain the types of classNames that can be used.

### cx.with()

Constrains the allowed className values by defining parameter types:

```typescript
const layoutClasses = cx.with<{
  display: "flex" | "grid" | "block";
  spacing: string;
  rounded?: boolean;
}>();

layoutClasses({
  display: "flex",
  spacing: "p-4 gap-2",
  rounded: true
});
// => "flex p-4 gap-2 rounded"

layoutClasses({
  display: "inline", // Error: Type '"inline"' is not assignable to type '"flex" | "grid" | "block"'
  spacing: "m-4"
});
```

Or use it as a className generator with parameters:

```typescript
const responsiveText = cx.with<{ 
  base: string;
  sm?: string;
  md?: string;
  lg?: string;
}>(({ base, sm, md, lg }) => 
  cx(base, sm && `sm:${sm}`, md && `md:${md}`, lg && `lg:${lg}`)
);

responsiveText({
  base: "text-sm",
  md: "text-base",
  lg: "text-lg"
});
// => "text-sm md:text-base lg:text-lg"
```

### cva.with()

Creates a parameterized variant utility that constrains parameter types and generates cva configuration:

```typescript
const myButton = cva.with<{ bg: string; text: string }>(
  ({ bg, text }) => ({
    base: `rounded border`,
    variants: {
      size: {
        small: "px-2 py-1 text-sm",
        large: "px-4 py-2 text-lg"
      }
    },
    // Dynamic styles based on parameters
    compoundVariants: ({ size }) => [
      {
        condition: [size.small],
        class: `bg-${bg} text-${text}`
      }
    ]
  })
);

// Single usage
const button = myButton({ bg: "blue-500", text: "white" });
button({ size: "small" });
// => "rounded border px-2 py-1 text-sm bg-blue-500 text-white"

// Multiple usage with .multiple()
const buttons = myButton.multiple({
  light: { bg: "white", text: "black" },
  dark: { bg: "gray-900", text: "white" }
});

buttons.light({ size: "small" });
// => "rounded border px-2 py-1 text-sm bg-white text-black"

buttons.dark({ size: "large" });
// => "rounded border px-4 py-2 text-lg bg-gray-900 text-white"
```

This allows you to create reusable variant factories with type-safe parameters, similar to `rules.with()`.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

[clsx](https://github.com/lukeed/clsx)
[classnames](https://github.com/JedWatson/classnames)

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
- If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain?

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

Think about what the natural extension and evolution of your proposal would
be and how it would affect the library and project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project and language in your proposal.
Also consider how this all fits into the roadmap for the project
and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
