- Start Date: 2025-10-19

# Summary

[summary]: #summary

This RFC proposes co-located React styling through three increasingly convenient
authoring layers:

- Mincho authoring exports such as `css()`, `rules()`, `theme()`, and the
  compatibility subpath APIs are lowered from the component module into
  build-time CSS artifacts;
- `styled()` binds those rule semantics to a React element or component;
- the JSX `css` prop attaches one-off styles directly to the element that uses
  them.

All three surfaces lower authoring-time style objects into Vanilla Extract
artifacts. Static CSS rules are generated during the build. Runtime code may
select pre-generated class names and assign values to pre-generated CSS custom
properties, but it does not serialize style objects, generate CSS rules, or
insert stylesheets while rendering.

# Motivation

[motivation]: #motivation

Traditional Vanilla Extract authoring places styles in a `.css.ts` module and
imports the resulting class names into a component module. This boundary is
explicit and statically analyzable, but it also separates a component from the
styles that only make sense for that component. Authors must name the style,
switch files, export it, import it, and manually connect it through `className`.

Co-location became popular because it removed that plumbing.

## Why styled-components was convenient

[styled-components](https://styled-components.com/docs/basics) made an element
and its styles one React component. Authors could name `Button` once, keep its
CSS beside its behavior, forward ordinary props, and express visual variants
through component props. Call sites consumed a semantic component instead of a
class-name protocol.

That convenience is independent of its runtime implementation. The design
lesson is the component boundary: styling should be able to produce a reusable
React component with typed semantic props.

## Why Emotion's CSS prop was convenient

[Emotion's CSS prop](https://emotion.sh/docs/css-prop) removed even the named
wrapper when a style belonged to one JSX site. An author could attach object
styles to an element, compose them with existing classes, and keep a small
layout adjustment beside the markup it affects.

The design lesson is the call-site boundary: not every local style deserves a
new component or exported style name.

styled-components and Emotion provide these conveniences through runtime CSS
serialization and stylesheet insertion in their standard React paths. Mincho
adopts the authoring boundaries, not the runtime CSS engine.

## Build-time prior art

Several projects demonstrate ways to retain co-location while constraining
styles for static extraction.

### Macaron

[Macaron](https://macaron.js.org/docs/working/) is the closest lowering prior
art. It finds co-located style and styled-component calls, moves CSS-producing
expressions into an extracted module, evaluates that module with Vanilla
Extract, and leaves runtime code that selects pre-generated classes. It shows
that `.css.ts` remains a build artifact without being a manual authoring
boundary.

### StyleX

[StyleX](https://stylexjs.com/docs/learn/styling-ui/using-styles) uses typed
object definitions, explicit composition, and compiler-generated atomic CSS.
Its local and cross-module behavior also demonstrates that build-time CSS does
not necessarily remove all runtime class selection. The relevant lesson is to
define "zero runtime" as the absence of runtime CSS generation, not the absence
of runtime JavaScript.

### WyW-in-JS

[WyW-in-JS](https://wyw-in-js.dev/how-it-works) provides a build-time processing
substrate for co-located CSS-in-JS libraries. A processor identifies its own
calls, evaluates the required JavaScript during the build, and emits
processor-defined CSS and JavaScript artifacts. The relevant lesson is to make
authoring-function discovery and the sidecar artifact contract extensible rather
than coupling lowering to one API name.

### Panda CSS

[Panda CSS](https://panda-css.com/docs/concepts/style-props) combines static
extraction with recipes, patterns, and generated JSX style props. Its compiler
must recognize the styling syntax statically, so convenience APIs remain within
an analyzable contract. The relevant lesson is that prop-based ergonomics must
fail clearly when the build cannot determine the resulting rule shape.

### Kuma UI

[Kuma UI](https://www.kuma-ui.com/docs/Concepts/Hybrid) combines familiar
`styled`, style-prop, and `css` APIs with separate static and dynamic paths. It
makes the trade-off visible: static values can be extracted, while truly dynamic
rules require runtime work. Mincho takes the stricter boundary and permits
runtime values only through pre-generated CSS custom properties.

### Tamagui

[Tamagui](https://tamagui.dev/docs/intro/compiler-install) applies an optional
compiler to co-located JSX and styled components. It partially evaluates style
props, hoists static work, emits atomic CSS for web, and can flatten safe
component boundaries while retaining runtime behavior when optimization does
not apply. The relevant lesson is to keep authoring semantics valid without an
optimization and treat compiler rewriting as a separate, provable layer.

## Mincho's direction

Mincho combines these lessons in three steps:

1. Macaron-like lowering lets Mincho's CSS, rules, theme, scoped authoring, and
   compatibility APIs stay beside a component while producing Vanilla Extract
   artifacts during the build.
2. `styled()` adds the component ergonomics popularized by styled-components
   without adding runtime CSS generation.
3. the JSX `css` prop adds Emotion-like locality for one-off styles while
   retaining a statically bounded rule language.

This preserves the reusable visual layer provided by `css()` and
[`rules()`](./002-css-rules.md), then adds semantic components and call-site
styles without requiring authors to maintain a separate `.css.ts` file.

## Goals

- Preserve type inference for intrinsic elements, components, variants, and
  toggles.
- Keep reusable and one-off styles close to their React usage.
- Reuse `rules()` as the semantic styling model instead of creating a second
  variant system.
- Extract static rule shapes during the build.
- Keep runtime behavior limited to component rendering, class-name selection,
  prop forwarding, and CSS custom-property assignment.
- Fail during transformation when a CSS rule shape cannot be determined.

## Non-goals

- Emotion-compatible runtime serialization or stylesheet insertion.
- Elimination of all runtime JavaScript.
- Automatic removal of individual unused variant values.
- Compiler-backend, CSS-output, and delivery optimizations.
- A framework-independent component runtime. Framework-specific extensions are
  discussed under Future possibilities.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

## 1. Co-located build-time lowering

The lowest co-location layer keeps Mincho authoring calls in the component
module. This follows Macaron's central design: authoring stays in `.tsx`, while
the build creates the extracted style module that Vanilla Extract needs.

```tsx
import { css } from '@mincho-js/css';

const panel = css({
	padding: '20px',
	backgroundColor: 'white',
});

export function Panel() {
	return <section className={panel}>Content</section>;
}
```

Conceptually, the transform moves the CSS-producing call into a generated
sidecar and replaces the authoring call with an imported class name.

```text
component.tsx with css(...)
  -> transformed component JavaScript
  -> generated .css.ts sidecar
  -> Vanilla Extract CSS artifact
```

The sidecar preserves the `.css.ts` build boundary without requiring authors to
create, name, export, and import that file manually.

### Build setup

Co-located authoring calls require the generic Mincho Babel transform and a
bundler adapter that compiles the generated sidecar. No React-specific JSX
configuration is part of this layer.

```typescript
// vite.config.ts
import { minchoVitePlugin } from '@mincho-js/vite';
import { defineConfig } from 'vite';

export default defineConfig({
	plugins: [minchoVitePlugin()],
});
```

The equivalent esbuild integration includes both Mincho extraction and Vanilla
Extract CSS processing.

```typescript
// esbuild.config.ts
import { minchoEsbuildPlugins } from '@mincho-js/esbuild';

export default {
	plugins: minchoEsbuildPlugins(),
};
```

At the Babel layer, `minchoBabelPlugin()` performs generic authoring-call
extraction. Its generated sidecar result still needs an integration or bundler
adapter, so enabling that low-level plugin alone is not a complete build setup.

### Authoring API coverage

Lowering is not specific to `css()` or `rules()`. Any public Mincho authoring
API that would otherwise need to execute in `.css.ts` should be eligible when
its package identity and member path are statically known.

```tsx
import {
	css,
	defineRules,
	rules,
	theme,
} from '@mincho-js/css';
import {
	recipe,
	styleVariants,
} from '@mincho-js/css/compat';

const [themeClass, vars] = theme({
	color: {
		text: '#111',
		accent: 'rebeccapurple',
	},
});

const card = css({ color: vars.color.text });

const tones = css.multiple({
	neutral: { color: vars.color.text },
	accent: { color: vars.color.accent },
});

const button = rules({
	variants: {
		tone: {
			neutral: { color: vars.color.text },
			accent: { color: vars.color.accent },
		},
	},
});

const { css: tokenCss } = defineRules({
	properties: { color: [vars.color.text, vars.color.accent] },
});

const label = tokenCss({ color: vars.color.accent });

const compatTones = styleVariants({
	neutral: { color: vars.color.text },
	accent: { color: vars.color.accent },
});

const compatButton = recipe({
	variants: {
		tone: {
			neutral: { color: vars.color.text },
			accent: { color: vars.color.accent },
		},
	},
});
```

The same rule applies to statically named functions attached with
`Object.assign`, including `css.raw`, `css.multiple`, `css.with`, `rules.raw`,
`rules.multiple`, `rules.with`, `theme.extends`, and `theme.with`. It also
applies to declared authoring subpaths such as `@mincho-js/css/compat`, rather
than treating only the root module as extractable.

Named imports, renamed imports, namespace imports, and static re-exports refer
to the same authoring identity. Runtime-computed property names and arbitrary
local wrapper functions are outside this contract because the build cannot
prove which authoring function they call.

Lowering preserves each API's result interface. A class remains a string, a
variant collection remains a class map, `theme()` retains its class, variables,
and contract tuple, and `rules()` or `recipe()` retains its callable selection
interface. Co-location changes where the function is evaluated, not what its
consumer receives.

## 2. Styled components

Co-located `css()` and `rules()` still require authors to name a class-producing
value, call it, and wire the result through `className`. `styled()` removes that
plumbing by binding the same rules semantics to an intrinsic element or React
component.

### Build setup

Styled lowering adds one Babel stage before generic authoring-call extraction:

```text
minchoStyledComponentPlugin()
  -> minchoBabelPlugin()
```

The Vite adapter provides both stages. It should run before the React plugin so
the Mincho syntax is normalized while JSX and TypeScript source are still
available.

```typescript
// vite.config.ts
import { minchoVitePlugin } from '@mincho-js/vite';
import react from '@vitejs/plugin-react';
import { defineConfig } from 'vite';

export default defineConfig({
	plugins: [minchoVitePlugin(), react()],
});
```

`minchoEsbuildPlugins()` provides the corresponding styled and generic lowering
plus Vanilla Extract processing for esbuild. The styled binding uses the
application's normal React JSX configuration.

The public `styled` export is intentionally a transform-required placeholder.
If a `styled()` call reaches runtime without lowering, it throws a build
configuration error instead of creating styles at runtime. There is no runtime
fallback.

```tsx
import { styled } from '@mincho-js/react';

const Button = styled('button', {
	paddingBlock: '10px',
	paddingInline: '20px',
	borderRadius: '4px',
	fontWeight: 'bold',
});

<Button type="button">Save</Button>;
```

HTML and SVG tags also have property accessors.

```tsx
const Button = styled.button({
	paddingBlock: '10px',
	paddingInline: '20px',
	borderRadius: '4px',
});
```

Both public forms lower to the same component and rule boundary. The accessor
form provides a concise intrinsic-element API; the call form also accepts
custom components.

A custom target must accept and forward `className` to the element that owns
the generated styles. If callers provide inline `style` assignments, including
values produced by a standalone `rules().props()` call, the target must forward
`style` to that same element.

### Rules binding

`styled(target, options)` does not define another styling language. It binds a
React target to the result of `rules(options)`.

```text
styled(target, options)
  -> rules(options)
  -> bind target + rules result
  -> StyledComponent
```

Base declarations, variants, toggles, defaults, compound variants, and their
class-selection order remain the contract of
[CSS Rules](./002-css-rules.md). The styled layer only exposes those selections
as component props and connects the selected classes to the target.

```tsx
const Button = styled.button({
	paddingBlock: '10px',
	paddingInline: '20px',
	variants: {
		tone: {
			primary: { backgroundColor: 'blue', color: 'white' },
			danger: { backgroundColor: 'red', color: 'white' },
		},
	},
	toggles: {
		outlined: { border: '1px solid currentcolor' },
	},
});

<Button type="button" tone="danger" outlined>
	Delete
</Button>;
```

### Returned component interface

The returned component combines:

- ordinary props from the bound element or component, such as `type` and
  `onClick` for a button;
- variant and toggle selections inferred from the rules result;
- caller-provided `className`;
- the polymorphic `as` prop and a forwarded ref.

At runtime, variant and toggle props are consumed by the rules result instead
of being forwarded to the DOM. Other props are forwarded, and the caller's
`className` is appended after the generated classes.

The binding does not automatically call `rules().props()` or create inline
custom-property assignments. Dynamic rule props remain explicit through
`style={rule.props(values)}` until a separate binding contract is accepted.

### Component targets and composition

An existing styled component can be used as the target of another styled
component.

```tsx
const Base = styled.div({
	fontWeight: 'bold',
	variants: {
		tone: {
			neutral: { color: 'gray' },
			accent: { color: 'rebeccapurple' },
		},
	},
});

const Card = styled(Base, {
	padding: '20px',
	backgroundColor: 'white',
	variants: {
		rounded: {
			true: { borderRadius: '8px' },
		},
	},
});

<Card tone="accent" rounded>
	Content
</Card>;
```

Composition remains component composition. Each styled layer contributes its
class name through normal `className` forwarding. This RFC does not flatten
nested component layers.

### Polymorphic rendering

The `as` prop changes the rendered element or component at runtime.

```tsx
const Text = styled.span({
	fontFamily: 'sans-serif',
	variants: {
		size: {
			small: { fontSize: '12px' },
			large: { fontSize: '18px' },
		},
	},
});

<Text size="large">Span</Text>;
<Text as="p" size="large">Paragraph</Text>;
<Text as="h1" size="large">Heading</Text>;
```

The runtime forwards refs and ordinary props to the selected target. Precise
ref and event-target narrowing across every `as` value remains an unresolved
type-system question.

## 3. JSX CSS prop

`styled()` is convenient when a style represents a reusable component, but a
one-off style should not require another component name. The `css` prop places
that style directly on the JSX element that owns it.

### Build setup

The CSS prop is opt-in. The Mincho adapter must enable `jsxCssProp: true` and run
before React transforms JSX.

```typescript
// vite.config.ts
import { minchoVitePlugin } from '@mincho-js/vite';
import react from '@vitejs/plugin-react';
import { defineConfig } from 'vite';

export default defineConfig({
	plugins: [minchoVitePlugin({ jsxCssProp: true }), react()],
});
```

esbuild uses the same option through
`minchoEsbuildPlugins({ jsxCssProp: true })`.

TypeScript must use Mincho's scoped automatic JSX runtime for the `css` prop
type and missed-transform guard.

```json
{
	"compilerOptions": {
		"jsx": "react-jsx",
		"jsxImportSource": "@mincho-js/react"
	}
}
```

If an own `css` prop survives until `@mincho-js/react/jsx-runtime` or
`jsx-dev-runtime`, the runtime throws a configuration error. It does not attempt
runtime CSS generation.

```tsx
function Notice() {
	return (
		<div css={{ padding: '20px', color: 'blue' }}>
			Notice
		</div>
	);
}
```

Existing class names are preserved and appear before the generated CSS class.

```tsx
<div className="layout" css={{ padding: '20px' }} />
```

An existing Mincho class can also be passed as a class value.

```tsx
import { css } from '@mincho-js/css';

const panel = css({ padding: '20px' });

<div css={panel} />;
```

Static CSS arrays remain CSS-rule composition.

```tsx
const base = { display: 'flex' } as const;

<div css={[base, { gap: '8px' }]} />;
```

### Dynamic declaration values

A declaration value may be dynamic when its surrounding CSS rule shape is
static.

```tsx
interface MeterProps {
	color: string;
	gap: number;
}

function Meter(props: MeterProps) {
	return (
		<div
			css={{
				color: props.color,
				marginTop: `${props.gap}px`,
			}}
		/>
	);
}
```

The build generates a class whose declarations read CSS custom properties.
Runtime code writes only the property values through `style`. It does not
generate a selector or declaration at runtime.

Dynamic keys, runtime object spreads, function-valued CSS, and other values
that can change the rule shape are rejected during transformation.

### Custom component forwarding

A custom JSX target must forward both `className` and `style` to the element
that receives the generated styles.

```tsx
import type { CSSProperties, ReactNode } from 'react';

interface PanelProps {
	className?: string;
	style?: CSSProperties;
	children?: ReactNode;
}

function Panel({ className, style, children }: PanelProps) {
	return (
		<section className={className} style={style}>
			{children}
		</section>
	);
}

<Panel css={{ color: 'blue' }}>Content</Panel>;
```

Forwarding `className` applies extracted classes. Forwarding `style` allows
dynamic declaration values to reach the same element.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## Meaning of zero-runtime CSS

In this RFC, **zero-runtime CSS** means that rendering does not:

- serialize style objects;
- generate selectors, declarations, or at-rules;
- insert or update stylesheet rules;
- maintain a runtime stylesheet cache.

It does not mean that styling has no runtime JavaScript. The React runtime
still renders components, separates variant props, selects pre-generated class
names, merges `className`, handles `as`, and forwards refs. Dynamic declaration
values are written to pre-generated CSS custom properties through inline
`style` values.

The more precise description is therefore **build-time CSS rule generation
with runtime class selection and variable assignment**.

## Lowering overview

```text
author .ts/.tsx
  -> Mincho transform
       -> transformed JavaScript
       -> extracted_<hash>.css.ts sidecar
  -> bundler adapter compiles the sidecar
  -> Vanilla Extract registers CSS
       -> .vanilla.css module
  -> bundler emits JavaScript and CSS assets
  -> React runtime selects classes and assigns dynamic values
```

The sidecar and virtual-module names describe the current artifact boundary;
they are not a public naming API. The design requirement is that authoring
syntax is removed before React rendering and that final CSS is emitted through
the bundler.

## Authoring API lowering

Generic lowering identifies an authoring call by its resolved package identity,
not by the local variable name:

```text
module source + exported name + static member path
```

For example, all of the following retain an authoring identity after imports are
resolved:

```tsx
import { css as makeCss, theme } from '@mincho-js/css';
import * as mincho from '@mincho-js/css';
import { recipe } from '@mincho-js/css/compat';

const card = makeCss({ color: 'red' });
const variants = mincho.css.multiple({
	primary: { color: 'blue' },
});
const button = recipe({ base: { color: 'red' } });
```

The CSS package declares which root exports, subpath exports, and static member
paths are authoring functions. This includes functions attached with
`Object.assign`; the transform recognizes the declared public path such as
`css.multiple` or `theme.with` and does not need to execute the `Object.assign`
expression that created the API object.

The initial authoring surface includes:

| Module | Authoring families |
| --- | --- |
| `@mincho-js/css` | `css`, `globalCss`, `rules`, `theme`, `globalTheme`, `defineRules`, and their declared static members |
| `@mincho-js/css/compat` | `style`, `globalStyle`, `styleVariants`, and `recipe` |

Aliases, namespace imports, and static re-exports preserve identity. A runtime
computed property path or an arbitrary local wrapper does not, so it is not
treated as an authoring call.

The transform moves the call and the static bindings it depends on into the
generated sidecar, then imports its result into the component module. It
preserves the public result shape: strings, class maps, theme tuples, contracts,
and callable rules or recipe interfaces cross the sidecar boundary unchanged.

Every declared authoring path requires a conformance fixture. Adding a root
export, assigned member, or authoring subpath without adding its lowering
fixture is an incomplete API change.

## Styled binding

The styled transform recognizes `styled` imported from `@mincho-js/react` and
normalizes component syntax into a target plus `rules(options)`.

```tsx
// Author source
const Button = styled('button', {
	color: 'red',
});
```

The complete lowering is conceptually:

```text
styled('button', options)
  -> $$styled('button', rules(options))
  -> extract rules(options) into the generated sidecar
  -> $$styled('button', importedRulesResult)
```

`rules(options)` is therefore not a special CSS implementation inside React. It
uses the same authoring API lowering contract as a standalone `rules()` call.
The React-specific result binds the imported callable rules interface to the
target component.

The property-accessor form only supplies the intrinsic target:

```tsx
styled.button({ color: 'red' });
// -> bind "button" to the extracted rules result
```

The public `styled` function is a transform-required placeholder. If an
untransformed call executes, it throws rather than generating CSS at runtime.

## CSS prop lowering routes

The CSS prop transform chooses the narrowest route whose behavior can be
determined safely.

### Static rule reduction

Inline objects and arrays, immutable local values, supported imported values,
static computed keys, and static spreads are reduced to a rule in the generated
sidecar.

### Sidecar rule delegation

A whole rule expression that can be preserved safely may be moved into the
generated sidecar and evaluated there by the build. Babel does not execute user
modules or factories to discover their returned styles.

### Dynamic declaration lowering

When keys and rule structure are static but declaration leaves are dynamic,
the transform generates CSS custom properties and rewrites the JSX `style`
value to assign them. Supported unit templates retain their unit suffix.

### Class-value lowering

Values that are already class names remain class values and are composed with
`cx()`. Conditional and logical class expressions preserve JavaScript
short-circuit behavior.

### Rejection

If a value would require runtime discovery of a selector, at-rule, property
name, object spread, or complete rule object, transformation fails. There is no
runtime CSS engine fallback.

## Class and style merge order

For an element without JSX spreads, an existing `className` is applied before
the class produced by `css`.

For explicit CSS props mixed with spreads, the order is:

1. class name from spreads before `css`;
2. class name produced by the explicit `css` prop;
3. class name from spreads after `css`.

Only an explicit `css` attribute activates this lowering. A `css` value that
exists only inside a spread is not transformed and is rejected by the scoped
JSX runtime guard. Explicit `key` or `ref` attributes on an element that also
requires CSS-prop spread aggregation are not currently supported.

Generated CSS custom-property assignments are merged after explicit and
spread-provided style objects so that transformed dynamic declarations receive
their generated property keys.

## Styled result interface

`$$styled` receives a target component and a `rules()` runtime function. It:

1. records the variant keys;
2. separates variant selections from forwarded props;
3. asks the rules runtime for pre-generated class names;
4. appends the caller's `className`;
5. renders the `as` target or the original target;
6. forwards ordinary props and the ref.

The public component type mirrors that binding. It combines:

- props from the target intrinsic element or React component;
- selections exposed by the callable rules result;
- the polymorphic `as` prop;
- React ref attributes.

Property accessors bind a finite HTML or SVG intrinsic target. The function form
can bind a custom component or another styled result. Component composition
therefore remains ordinary React composition connected through `className`, not
compiler flattening.

Precise event and ref narrowing after polymorphic substitution must be verified
before it is described as a guarantee.

## JSX runtime guard

The scoped `jsx-runtime` and `jsx-dev-runtime` do not process styles. They only
detect an own `css` prop that survived transformation and throw a configuration
error before delegating to React.

## Build adapters and artifacts

The Babel package owns syntax lowering. Shared integration code owns source
resolution, static-evaluation metadata, dependencies, and sidecar artifacts.

Vite compiles generated sidecars, exposes virtual Vanilla Extract CSS modules,
handles HMR invalidation, and ensures library entry chunks load their CSS
assets. The esbuild adapter resolves and loads generated sidecars, then hands
final CSS processing to the Vanilla Extract esbuild plugin.

Vite and esbuild are the adapter contracts covered by this RFC. A React SWC
application may run after the Mincho Vite pre-transform, but this does not imply
a native Mincho SWC transform. Other bundlers require their own integration.

## Illustrative CSS output

Generated identifiers are owned by Vanilla Extract and build configuration.
The following output is illustrative, not a naming guarantee:

```css
.Button_base__hash {
	padding-block: 10px;
	padding-inline: 20px;
}

.Button_tone_primary__hash {
	background-color: blue;
	color: white;
}
```

The stable contract is the relationship among base, variant, compound, and
caller-provided classes, not their literal names.

# Drawbacks

[drawbacks]: #drawbacks

## Build configuration is required

All co-located authoring calls require a transform and a sidecar-aware bundler
adapter. Styled adds its normalization stage before generic extraction. The CSS
prop additionally requires the scoped JSX runtime, an explicit transform option,
and ordering before React lowers JSX.

## Static rule-shape restrictions

The CSS prop cannot accept arbitrary runtime objects. Dynamic keys, spreads,
selectors, and at-rules would require runtime CSS generation and are rejected.

## Runtime component work remains

Styled components still perform variant selection, prop separation, class-name
composition, polymorphic target selection, and ref forwarding during render.
The proposal removes runtime CSS generation, not component runtime cost.

## Adapter complexity

Generated sidecars, virtual CSS modules, dependency tracking, HMR invalidation,
and final CSS asset delivery require adapter-specific integration. Supporting a
new bundler is more than adding a Babel transform.

## Source maps

The current lowering pipeline does not preserve meaningful source maps through
all generated JavaScript and CSS artifacts. Generated output and transformation
errors must provide enough context until that support is designed.

## Type-system complexity

Combining intrinsic props, custom component props, variants, refs, and `as`
substitution creates expensive and subtle TypeScript types. Runtime behavior can
be correct while event or ref narrowing remains less precise than expected.

## More than one authoring surface

`styled`, `css`, `rules`, and compat `recipe` serve different reuse levels but
increase the learning surface. Documentation must state which layer owns each
syntax and avoid presenting compatibility syntax as core syntax.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

## Why build-time lowering?

Runtime CSS-in-JS can accept arbitrary runtime style objects, but it must also
serialize declarations and manage stylesheet insertion while the application
runs. Mincho instead accepts a statically bounded rule language and emits CSS
through the build.

CSS Modules and handwritten Vanilla Extract files already provide build-time
CSS. The co-location transform adds component semantics and JSX-local authoring
without requiring every style to live in a separate file.

## Why reuse rules?

Binding `styled()` to `rules(options)` gives Mincho one definition of variants,
toggles, defaults, compound conditions, and class selection. Dynamic
declaration props remain an explicit rules result until their component binding
is specified. A separate React-only variant engine would duplicate both types
and runtime behavior.

## Why object syntax?

Object syntax is compatible with Mincho's CSS preprocessing and TypeScript
property checking. Template literals would require another parser and would not
reuse the existing CSS rule types directly.

## Why keep styled and the CSS prop?

They express different reuse boundaries:

- use `styled` when an element has component semantics and reusable props;
- use the CSS prop when a style belongs to one JSX site;
- use `rules()` or compat `recipe()` when reusable styles should remain plain
  class-name functions.

Removing any one surface forces its use cases into a less precise abstraction.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- How should CSS packages declare their authoring exports, assigned member
  paths, and authoring subpaths so transforms do not rely on a hard-coded list?
- Should styled components automatically consume `rules().props()` and merge
  dynamic values into `style`, or should dynamic rule props remain explicit?
- Should a pre-built `rules()` or compat `recipe()` function be accepted by
  `styled()` without wrapping it in another `rules()` call?
- What ref and event-target precision must polymorphic `as` guarantee?
- Which generated artifact boundaries need stable source-map support?
- Which additional bundler adapters are required before stabilization?
- Which CSS prop expression forms are part of the stable language, and which
  remain implementation-specific conveniences?
- Should absent caller `className` values be normalized so generated class
  strings never contain trailing whitespace?

# Future possibilities

[future-possibilities]: #future-possibilities

## Other framework runtimes

`rules()` and the CSS artifacts are framework-independent, so other component
runtimes can reuse the same build-time rule generation.

A SolidJS binding could provide a Solid-specific component wrapper and reactive
class selection while retaining the same rule functions. React Native would
require a different lowering target because it uses `StyleSheet` rather than
CSS classes, selectors, and stylesheets. Each runtime requires its own RFC for
props, refs, reactivity, SSR, and platform semantics.

## Optimization

Possible optimizations include an OXC transform backend, unused-variant
analysis, safe component flattening, atomic CSS deduplication, minification,
critical CSS extraction, and route-aware CSS splitting. These ideas do not
change the co-location authoring contract and belong in a separate optimization
RFC.
