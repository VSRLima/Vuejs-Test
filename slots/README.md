# Slots

## Slot Content and Outlet
In some cases, we may want to pass a template fragment to a child component, and let the child component render the fragment within its own template.

For example, we may have ```<FancyButton>``` component that supports usage like this:

``` 
<FancyButton>
    Click me! <!-- slot content -->
</FancyButton>
```

The template of ```<FancyButton>``` looks like this:

``` 
<button class="fancy-btn">
  <slot></slot> <!-- slot outlet -->
</button>
```

The ```<slot>``` element is slot outlet that indicates where the parent-provided slot content should be rendered.

And the final rendered DOM:

``` 
<button class="fancy-btn">Click me!</button>
```

Another way to understand slots is by comparing them to JavaScript functions:

``` 
// parent component passing slot content
FancyButton('Click me!')

// FancyButton renders slot content in its own template
function FancyButton(slotContent) {
  return `<button class="fancy-btn">
      ${slotContent}
    </button>`
}
```

Slot content isn't just limited to text. It can be any valid template content. For example, we can pass in multiple templates, or even other component:

``` 
<FancyButton>
  <span style="color:red">Click me!</span>
  <AwesomeIcon name="plus" />
</FancyButton>
```

By using slots, our ```<FancyButton>``` is more flexible and reusable. We can now use it in different places with different inner content, but all with the same fancy styling.

## Render Scope

Slot content has access to the data scope of the parent component, because it's defined in the parent. For example:

``` 
<span>{{ message }}</span>
<FancyButton>{{ message }}</FancyButton>
```

Here both ```{{ message }}``` interpolations will render the same content.

Slot content doesn't have access to the child component's data. Expressions in Vue templates can only access the scope it's defined in, consistent with JavaScript's lexical scoping. In other words:
- Expressions in the parent template can only have access to the parent scope; expressions in the child

## Fallback Count

There are cases when it's useful to specify a fallback content for a slot, to be rendered only when no content is provided. For example, in a ```<SubmitButton>``` component:

``` 
<button type="submit">
  <slot></slot>
</button>
```

We might want the text "Submit" to be rendered inside the ```<button>``` if the parent didn't provide any slot content. To make "Submit" the fallback content, we can place it in between the ```<slot>``` tags:

``` 
<button type="submit">
  <slot>
    Submit <!-- fallback content -->
  </slot>
</button>
```

## Named Slots

In case of multiple slots we can have as an example the component ```<BaseLayout>```:

```
<div class="container">
  <header>
    <!-- We want header content here -->
  </header>
  <main>
    <!-- We want main content here -->
  </main>
  <footer>
    <!-- We want footer content here -->
  </footer>
</div> 
```

For these cases, the ```<slot>``` has a special attribute, ```name```, which can be use to assign a unique ID to different slots so you can determinate where content should be rendered:

``` 
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

A ```<slot>``` outlet without ```name``` implicity has the name "default"

To pass a named slot, we need to use ```<template>``` element with the ```v-slot``` directive, and then pass the name of the slot as an argument to ```v-slot```:

``` 
<BaseLayout>
  <template v-slot:header>
    <!-- content for the header slot -->
  </template>
</BaseLayout>
```

v-slot has a dedicated shorthanded ```#```, so ```<template v-slot:header>``` can be shortened to ```<template #header>```.

Here's the code passing content for all three slots to ```<BaseLayout>``` using shorthanded syntax:

``` 
<BaseLayout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <template #default>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </template>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</BaseLayout>
```

When a component accepts both a default and named slots, all top-level non-```<template>``` nodes are implicitly treated as content for the default slot. So the above can also be written as:

``` 
<BaseLayout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <!-- implicit default slot -->
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</BaseLayout>
```

## Conditional Slots

In the example below we define a Card component with three conditional slots: ```header```, ```footer``` and the ```default```. When the header/footer/default is present we want to wrap them to provide additional styling:

``` 
<template>
  <div class="card">
    <div v-if="$slots.header" class="card-header">
      <slot name="header" />
    </div>
    
    <div v-if="$slots.default" class="card-content">
      <slot />
    </div>
    
    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer" />
    </div>
  </div>
</template>
```

## Dynamic Slots

Dynamic directive arguments also work on ```v-slot```, allowing the definition of dynamic slot names:

``` 
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>

  <!-- with shorthand -->
  <template #[dynamicSlotName]>
    ...
  </template>
</base-layout>
```

## Scoped Slots

We can pass attributes to a slot outlet just like passing props to a component:

``` 
<!-- <MyComponent> template -->
<div>
  <slot :text="greetingMessage" :count="1"></slot>
</div>
```

Receiving props in a slot is slightly different from a component, it goes like this: 

``` 
<MyComponent v-slot="slotProps">
  {{ slotProps.text }} {{ slotProps.count }}
</MyComponent>
```

Also notice that we can destructure the ```slotProps``` just like a function:

``` 
<MyComponent v-slot="{ text, count }">
  {{ text }} {{ count }}
</MyComponent>
```

### Named Scoped Slots

Name scoped slots work similarly - slot props are accessible as the value of ```v-slot``` directive: ```v-slot:name="slotProps"```. When using the shorthanded, it looks like this:

``` 
<MyComponent>
  <template #header="headerProps">
    {{ headerProps }}
  </template>

  <template #default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>
```

Passing props to a named slot:

```
<slot name="header" message="hello"></slot> 
```

Note the ```name``` of a slot won't be included in the props because it's reserved - so the resulting ```headerProps``` would be ```{ message: 'hello' }```.


