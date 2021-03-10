# Slots API specification

## Slots definition

Fluent UI components almost always contain sub parts and these pre-defined layouts can be configured via Slots API. Slots are named areas in a component that can receive props and provide styling and layout for them.

For example, `Button` component contains `content`, `loader` and `icon` slots in its layout:

```jsx
// ⚠️ not a real JSX/DOM markup
<slots.root>
  <slots.loader />
  <slots.children />
  <slots.icon />
</slots.root>
```

Slots let define the complex layouts required by many components according to accessibility and design requirements automatically. Slots can be configured through props, which lets the caller pass in a variety of inputs for a given slot.

## Slots components

Each slot is represented by a React component or a primitive, for example: `slots.loader` can be a `Loader` component and `slots.icon` can be a `span`:

```jsx
// ⚠️ not a real JSX/DOM markup
<>
  {/* renders a <button /> element */}
  <slots.root>
    {/* renders a <Loader /> element */}
    <slots.loader />
    {/* renders a <span /> element */}
    <slots.icon />
  </slots.root>
</>
```

A default set of components is manually configured per each component, but is optional, by default slots are represented by `div` elements.

## Slots usage

There are several forms of values that can be provided, but all of them share one common thing - they will produce a proper layout inside a component.

### An object as a value

If you will pass an object to component's props we will handle it as props for a slot, this behavior can be considered as declaring a JSX element with a javascript object. In an example below, props will be passed to an `icon` slot:

```jsx
<Button icon={{ children: <FooIcon />, className: 'an-awesome-slot', id: '#button-icon' }} />
```

```html
<!-- 💡 Simplified DOM output -->
<button class="ms-Button">
  <!-- 👇 An additional class and id have been added to markup -->
  <span class="ms-Button-icon an-awesome-slot" id="#button-icon">
    <span class="ms-Icon"><svg /></span>
  </span>
</button>
```

Any props what are valid for a component can be passed to a slot, i.e. `data-*` attributes and event handlers:

```jsx
<Button
  icon={{
    children: <FooIcon />,
    'data-test-id': 'button-foo-icon',
    onClick: () => {
      console.log('A click on an icon slot!');
    },
  }}
/>
```

### Primitives as a value

We consider JSX elements as primitive values that will be passed to a meaningful prop, by default to `children` (but this can be configured in the component's implementation):

```jsx
<>
  <Button icon={{ children: <FooIcon /> }} />
  {/* 💡 will produce the same JSX/HTML markup, as will be expanded to { children: <FooIcon /> } */}
  <Button icon={<FooIcon />} />
</>
```

Such usage of slots is called shorthands, similarly to [CSS shorthand properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Shorthand_properties). The same behavior is applicable strings and numbers:

```jsx
<>
  <Button icon={{ children: 'text instead of icon' }} />
  {/* 💡 will produce the same JSX/HTML markup */}
  <Button icon="text instead of icon" />
</>
```

To disable slot rendering you can use falsy (`null`, `false`) values:

```jsx
// 💡 A loader slot will be hidden
<>
  <Button loading loader={null} />
  <Button loading loader={false} />
</>
```

`true` will render a slot without any content:

```jsx
// 💡 A loader slot will be renderer without any content
<Button loading loader={true} />
```

```html
<!-- 💡 Simplified DOM output -->
<button class="ms-Button">
  <span class="ms-Button-loader"></span>
</button>
```

### Renders props via `children` function

Slots can be deeply customized via `children` function that behave similarly to [Render Props pattern](https://reactjs.org/docs/render-props.html) in React:

```jsx
<Button
  icon={{
    // 💡 "Component" is an original element type for slot, for example, it can be a "span"
    //    "props" are defaults that are provided to a slot by a host component, for example, may contain "onClick"
    //    handler
    children: (Component, props) => (
      <div id="icon-wrapper">
        {/* 💡 "props" can be modified there to override component's behavior */}
        <Component {...props} id="icon-slot">
          <FooIcon />
        </Component>
      </div>
    ),
  }}
/>
```

```html
<!-- 💡 Simplified DOM output -->
<button class="ms-Button">
  <!-- 👇 An additional element was added to markup -->
  <div id="icon-wrapper">
    <span class="ms-Button-icon" id="icon-slot">
      <span class="ms-Icon"><svg /></span>
    </span>
  </div>
</button>
```

⚠️An important note is that `children` function replaces the whole slot, not its contents.