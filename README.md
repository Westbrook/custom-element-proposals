# Declarative Custom Elements
### A discussion...

## Problem
Imperative JS code is required to unlock the power of [Custom Elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements), [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM), and other "web components" APIs. This is particularly true when these APIs are used in concert with each other. A declarative API would open the door for things like:

- non-JS developers being given the power of custom elements
- having custom element definitions available as parse time without requireing the jump over to the JS thread early in the document parse
- others?

*Questions:* Have we come to a future where "web components" amount to _just_ Custom Elements and Shadow DOM? Could we benefit from including other specs in this conversation? Possibly:

- [constructible stylesheets](https://web.dev/constructable-stylesheets/)
- [adoptedStyleSheets](https://wicg.github.io/construct-stylesheets/#using-constructed-stylesheets)
- [aria delegation](https://leobalter.github.io/cross-root-aria-delegation/) (possibly implied but also not currently "spec", only proposal)
- [scoped registries](https://github.com/WICG/webcomponents/issues/716)
- others? see [current Web Components Community Group priorities](https://w3c.github.io/webcomponents-cg/)

## Existing research
*NOTE:* It would be great to get someone with specific experience with these technologies to write up a demo outlining the declarative nature of components in their context.

- Vue
- Svelte
- [Declarative Shadow DOM](https://web.dev/declarative-shadow-dom/)
- [Declarative CSS modules](https://github.com/WICG/webcomponents/issues/939)
- [x-element](https://www.npmjs.com/package/astro-xelement) from Astro
- [decorate element](https://matthewphillips.info/programming/posts/decorate-element/)
  - cons: much like DSD does not allow for scripting.
- others?

Previous conversation in this space have halted at "do we have enough coverage of exisiting paradigms in the room?", let's address that early and often!

## Dreams
I would could this as a smashing success if we were able to do something _like_ the following, but understand this is more of a starting place than necessarily the API that we should drive towards.

1. *External "module"*
  ```html
    <import rel="customelement" href="my-element.html" tagname="my-element" />
    <!-- ... -->
    <my-element></my-element>
  ```
2. *Inline "module"*
  ```
    <customelement tagname="my-element">
      <!-- ... -->
    </customelement>
    <my-element></my-element>
  ```

## Related research

### Import assertions
Can we assert that a file we import ins a"custom element" rather than JSON or CSS or in the future plain HTML? The graph created here may be secondary to a declarative "element" that encapsulated the definition and registration of a custom element.

### Declarative Shadow DOM
This technique upgrades an existing element, `<template>`, to have the side effect of attaching a shadow root the parent element within which is lives. Would we benefit from following this style of approach when addressing inline API as it seems to build on the inert nature of the `<template>` tag to begin with? This might presuppose that instead of a `<customelement>` style registration we coudl leverag something along the lines of `<template custom-element="my-tag">` for these properties when addressing a new specification.

### Module boundaries and scope
When leveraging the JS module graph either as `import ...` or as `<scropt type="module">` we are provided a new scope in which the code available therein is separate from teh rest of the JS scope. Would it make sense to allow for a custom element registered via either via import mechanism or via attribute rules to have such a boundary? In this way we could allow for techniques like scoped registries to be available as if for free with the new scope being create at these boundaries by default...

### Inline modules (CSS or JS or CEs) tagged as URL
Declarative CSS module research has included inestigation into a sharing mechanism to [prevent multiple copies of the same CSS module](https://github.com/WICG/webcomponents/issues/939). The ability, and possible need, to do so would prevent duplication in a "bundled" context; whether that bundle was manual or built by tooling. ID references do not cross Shadow DOM boundaries, so a technique to cache and reference this content in those situations would be productive.

## Conciderations

### Polyfill/tooling to aid adoption
Any solution put forward here would benefit greatly by being paired as soon as possible with a bundler plug-in or polyfill that would make it possible for consumers to leverage the technique _today_. Previous attempts at larger scale quality of life improvements in this area have suffered greatly from low x-browser investment that has made developer exploration, let along developer adoption, of advances difficult, if not impossible. In the case that this tool the form of a polyfill, it would be important that the polyfill were small enough to inline within the `<head>` of a document in order to mitigate any potention FOUC.

### Is it possible to make components that are valuable to users with 0 JS?
An oft confused (or similarly highly important) topic when dicussing "declarative" APIs is "does it work with no JS?" and that is something that will come up often in any conversations here. If we bind this approach to a `<script>` tag, does that imply that the "declarative" API requires JS? Could a browser vendor speak to whether or not `<script>` is the boundary at which JS being turned off the parser turns back, or is it possible that `asserttype` would allow `customelement` content within a `<script>` tag to still be run in that context? IF that is not possible, then this may already start to shape the aboslutely possibilities of this addition and point us towards alternate choices. The `<template>` tag is certainly an option for inline techniques but we would still be lacking a graph mechanism by which we could build this code. Long term web component developers might lament the loss of HTML Imports in this case, and it might be worth revisiting that concept where actual 0 JS a requirment.

### Graph reliance and side effects
There are developers that rely on a single mass import of side effectful registrations of custom elements at the top of their application to coverall custom element usage across the rest of their application. This appears to be a side effect (get it?) of custom elements having a single global scope. If single file or single tag custom elements were to be "module scoped" this reliance could be seen as confusing when going back and forth between JS context. How can we manage this? JSON modules are just data. CSS modules only apply when adopted. CE modules should be handled how?
