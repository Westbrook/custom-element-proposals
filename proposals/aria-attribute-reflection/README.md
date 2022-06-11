# Aria Attribute Reflection

_*REQUIRED READING:* The following assumes that the [Aria Attribute Delegation spec](https://leobalter.github.io/cross-root-aria-delegation/) or similar is accepted into the HTML standard in support of delagating aria attributes down into shadow roots. This spec, its path towards standardization, and the [associated explainer](https://github.com/leobalter/cross-root-aria-delegation/blob/main/cross-root-aria-delegation.md) may provide insights into the final shape of solution to the following problem._

## Background

The Aria Practices Guide outlines an example editable combobox without autocomplete pattern as having the following HTML:

```html
<label for="cb1-input">Search</label>
<div class="combobox combobox-list">
  <div class="group">
    <input
      id="cb1-input"
      class="cb_edit"
      type="text"
      role="combobox"
      aria-autocomplete="none"
      aria-expanded="false"
      aria-controls="cb1-listbox"
    />
    <button
      type="button"
      id="cb1-button"
      tabindex="-1"
      aria-label="Previous Searches"
      aria-expanded="false"
      aria-controls="cb1-listbox"
    >
      <svg
        width="18"
        height="16"
        aria-hidden="true"
        focusable="false"
        style="forced-color-adjust: auto"
      >
        <polygon
          class="arrow"
          stroke-width="0"
          fill-opacity="0.75"
          fill="currentcolor"
          points="3,6 15,6 9,14"
        ></polygon>
      </svg>
    </button>
  </div>
  <ul id="cb1-listbox" role="listbox" aria-label="Previous Searches">
    <li id="lb1-01" role="option">weather</li>
    <li id="lb1-02" role="option">salsa recipes</li>
    <li id="lb1-03" role="option">cheap flights to NY</li>
  </ul>
</div>
```

A fairly HTML-first conversion of this pattern to custom element with shadow DOM might look like:

```html
<custom-label for="cb1-input">Search</custom-label>
<custom-combobox>
    #shadow-root
        <div class="group">
            <slot name="input"></slot>
            <slot name="button"></slot>
        </div>
        <slot name="listbox">
    <custom-input
        id="cb1-input"
        role="combobox"
        aria-autocomplete="none"
        aria-expanded="false"
        aria-controls="cb1-listbox"
        slot="input"
    >
        #shadow-root delegatesRole delegatedAriaExpanded delegatedAriaControls
            <input autoRole autoAriaExpanded autoAriaControls />
    </custom-input>
    <custom-button
        type="button"
        tabindex="-1"
        aria-label="Previous Searches"
        aria-expanded="false"
        aria-controls="cb1-listbox"
        slot="button"
    >
        <custom-icon></custom-icon>
    </custom-button>
    <custom-listbox
        id="cb1-listbox"
        aria-label="Previous Searches"
        slot="listbox"
    >
        <custom-listbox-item id="lb1-01">weather</custom-listbox-item>
        <custom-listbox-item id="lb1-02">salsa recipes</custom-listbox-item>
        <custom-listbox-item id="lb1-03">cheap flights to NY</custom-listbox-item>
    </custom-listbox>
</custom-combobox>
```

In the above revision, the `<custom-*>` elements aren't assumed not to have JS, but there is only a small amount leveraged in this case beyond simply registering custom element.

Some assumptions about the way these elements are prepared include:

- roles that can be fixed on elements like `<custom-listbox>` and `<custom-listbox-item>` and removed based on the ability to either sprout that data or apply it with `ElementInternals` were available
- the `for` attribute on `<custom-label>` is either fully synthetic
- slots are used to move general layout of the UI into the `<custom-combobox>` element

Already two important pieces of functionality are seen to be missing in browsers today that we won't be going into in this document:

-  `for`-centric functionality needing to be synthetic outlines a need for some form of the mythical "element mixins" based approach to impowering custom elements with features otherwise available to built-in elements that varius browsers have outlined a lack of desire to allow customization of
- [Aria Attribute Delegation](https://leobalter.github.io/cross-root-aria-delegation/) is available to pass neccessary attributes into the `<custom-input>` element and onto the `<input />` that would need to be therein

However, beyond initial delivery to appropriately deliver this UI in an accessible manner JS is required, and as a UI gets further and further into an application stack, the more likely it is for a component author to both expect and rely on JS to make their work easier. In this way, a `<custom-combobox>` in the wild might look more like this:

```html
<custom-label for="cb1-combobox">Search</custom-label>
<custom-combobox
    id="cb1-combobox"
    label="Previous Searches"
    items='["weather", "salsa recipes", "cheap flights to NY"]'
>
    #shadow-root delegatedAriaLabel
        <div class="group">
            <custom-input
                role="combobox"
                aria-autocomplete="none"
                aria-expanded="false"
                slot="input"
                aria-controls="listbox"
                autoLabel
            >
                #shadow-root delagatesLabel delegatesRole delegatedAriaExpanded delegatedAriaControls
                    <input autoLabel autoRole autoAriaExpanded autoAriaControls />
            </custom-input>
            <custom-button
                type="button"
                tabindex="-1"
                aria-label=${label}
                aria-expanded="false"
                aria-controls="listbox"
            >
                <slot name="icon"><custom-icon></custom-icon></slot>
            </custom-button>
        </div>
        <custom-listbox
            id="listbox"
            aria-label=${label}
            items=${items}
        >
            #shadow-root
                <!-- templating/imperative code that creates -->
                <custom-listbox-item id="lb1-01">weather</custom-listbox-item>
                <custom-listbox-item id="lb1-02">salsa recipes</custom-listbox-item>
                <custom-listbox-item id="lb1-03">cheap flights to NY</custom-listbox-item>
        </custom-listbox>
</custom-combobox>
```

The important version of this is:

```html
<custom-label for="cb1-combobox">Search</custom-label>
<custom-combobox
    id="cb1-combobox"
    label="Previous Searches" 
    items='["weather", "salsa recipes", "cheap flights to NY"]'
></custom-combobox>
```

