In reference to https://github.com/WICG/aom/issues/192

```html
<fancy-label
    for="input"
    id="label"
>
  <!-- I've not seen talk of offering "for" functionality on other elements, so we likely need to leverage labelledby instead -->
  #shadow-root
    <label>Label</label>
</fancy-label>
<fancy-input
    id="input"
    aria-labelledby="label"
    role="combobox"
    aria-controls="listbox"
    aria-expanded="false"
    aria-activedescendant="listbox"
>
  #shadow-root delegates='aria-labelledby role aria-controls aria-expanded aria-activedescendant'
    <!-- https://leobalter.github.io/cross-root-aria-delegation/ doesn't currently include 'aria-labelledby', but should? -->
    <!-- https://leobalter.github.io/cross-root-aria-delegation/ doesn't currently include 'role', but should? -->
    <!-- https://leobalter.github.io/cross-root-aria-delegation/ doesn't currently include 'aria-controls', but should? -->
    <!-- https://leobalter.github.io/cross-root-aria-delegation/ doesn't currently include 'aria-activedescendant', but should? -->
    <input type="text" autoAriaLabelledby autoRole autoAriaExpanded autoActivedescendant>
</fancy-input>
<fancy-listbox
    id="listbox"
>
  #shadow-root
    <ul role="listbox">
      <!-- OP skipped over the inability to use this DOM structure, but no need to nitpick -->
      <!-- Placing the role here implies that we can reflect the element out of the shadow DOM in some way,
      I've not see spec for this, but it's been listed as the next step after delegation. See "Reflection" note below. -->
      <fancy-option>
        #shadow-root
          <li role="option" id="option-0">List item</li>
            <!-- Placing the open here implies that we can reflect the element out of the shadow DOM in some way,
            I've not see spec for this, but it's been listed as the next step after delegation. See "Reflection" note below. -->
      </fancy-option>
    </ul>
</fancy-listbox>
```

## Reflection: as a oposite of delegation
I'd much prefer that we work out a reflection process that was similar to the delegation API as it would make both
processes "declarative". In that way you _might_ see something like this horrid pseudo code:

```html
<fancy-listbox
    id="listbox"
>
  #shadow-root reflects="role aria-activedescendant"
    <ul role="listbox" reflectsRole>
      <!-- This is tricky, because it's not "correctly" managed by delegation, maybe it is and this should be the role at the host
      and that role should be delegated to this element. It's tricky because in a custom element many elements could have different
      roles which makes it unclear how to manage this many to many relationship. See "Roles" note below.
      
      I'd love to know more about whther the `aria-controls` values is _supposed_ to connect to a [role="listbox"] element to
      work appropriately, or not. In SWC we currently are testing an approach that _doesn't_ that seems fine, but 🤷‍♂️ -->
      <fancy-option id="option-0" reflectsActivedescendant>
        #shadow-root reflects="role"
          <li role="option" reflectsRole>List item</li>
            <!-- The `reflectsActivedescendant` attribute here would reflect this as the element that was mapped in the accessibility tree
            to the <input> in the <fancy-input> element. I think it follow all of the laws of encapsulation in that there is no reference
            passing in the JS space, and only when you get to the accessibility tree does the association get fully made. -->
      </fancy-option>
    </ul>
</fancy-listbox>
```

## Roles:
Role feels a bit like it should be on individual elements with in a shadow root, but if those roles are relative to the containing
element or application how do you make them available there? Reflection above shows one path, but you might also say one elemnt
holds numerous "roles" and delegates them to its shadow DOM children. In that way you _might_ see something like this horrid pseudo code:

```html
<fancy-listbox
    id="listbox"
    roles="listbox option"
>
  <!-- Add the `roles` attrbute to aggregate all of the roles this element pays in the parent context. -->
  #shadow-root delegates="role"
    <!-- https://leobalter.github.io/cross-root-aria-delegation/ doesn't currently include 'role', but should? -->
    <ul autoRole="listbox">
      <!-- Maps the `role` that will be applied to this one element. -->
      <fancy-option id="option-0" autoRole="option">
        <!-- Maps the `role` that will be applied to this one element. -->
        #shadow-root delegates="role"
          <!-- https://leobalter.github.io/cross-root-aria-delegation/ doesn't currently include 'role', but should? -->
          <li autoRole>List item</li>
          <!-- Maps the `role` that will be applied to this one element. -->
      </fancy-option>
    </ul>
</fancy-listbox>
```

## IDL attributes:
Being able to set aria references in the JS space is nice, but it does inherently contain you to a single document tree, as noted
by the OP, or force you to surface element references as part of the API of your shadow root containing elements, which would be seen
as a leaky API. In maybe ways this is what work arounds do today. In SWC we assume out focuasable elements have a `focusElement`
property and when we need to reach into them to label them our `<sp-field-label>` element knows that that property might exist
and checks for it when deciding how best to label an element. In this case we'd need to do similar:

```html
<fancy-listbox
    id="listbox"
>
  <!-- This element would need to surface a property for `activedescendent` and possibly `listbox` (see question about relationship
  above) in order to satisfy the IDL requirements of <fancy-input>. In this way the -->
  #shadow-root
    <ul role="listbox">
      <fancy-option id="option-0">
        <!-- This element would need to surface a property for `option` in order for it do be set as the value of `activedescendent` 
        in the containing element for it to be leveraged in <fancy-input>. -->
        #shadow-root
          <li role="option">List item</li>
      </fancy-option>
    </ul>
</fancy-listbox>
```

In this was it would be great to pair IDL attributes with combinations of the delagate and reflect patterns that we've seen above.
With these in concert is will be possible for the developer of <fancy-listbox> to manage internally the mapping of aria attribute
content across the DOM tree that it has ownership of without needing to leak references to its descendents into the parent tree.
With this being so, you might wonder what even is the benefit of IDL references at all. To understand this this it is important to
clarify the assumption that the demos above live in a single root DOM tree, hopefully something like `document`. If all these things
like in document then you'll always be able to make the ID ref not matter what you might do with the content in question. However,
if you are contained in any parent elements, and any of those elements apply some form of CSS clip/layering and others apply their
own shadow roots, then without the presence of something like https://open-ui.org/components/popup.research.explainer you might 
find yourself throwing any overlaid contents to the end of the `document` to ensure that the entirety of that content is visible and
that it is above all other content. That is to say that while "open" the DOM might actual end up looking like:

```html
<body>
  <parent-element>
    #shadow-root
      <fancy-input>
        #shadow-root
          <input type="text">
      </fancy-input>
  </parent-element>
  <!-- interceeding content -->

  <fancy-listbox>
    #shadow-root
      <ul role="listbox">
        <fancy-option>
          #shadow-root
            <li role="option">List item</li>
        </fancy-option>
      </ul>
  </fancy-listbox>
</body>
```

For the above code to work, you will have needed to make and applied an IDL reference before the content was reparented in order for
the accessibility tree to continue to be made approapriately. At this distance, interceeding shadow roots, the ID ref would not be
possible, and the imperative JS connection would be required.

This does, however, raise the question of how one might make this relationship hold in a context where Declarative Shadow DOM was 
leveraged for the delivery of the content. Being reparenting isn't possible without JS, it is possible that we may be able to 
avoid addressing this context, but it is the sort of question that we should be sure to keep on our mind as we continue to expand
the feature surface of the browser in imparative ways.