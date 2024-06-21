# Components Basic
## Defining a Component
When using a build step, we typically define each Vue component in a dedicated file using the ```.vue``` extension - known as Single-File Component

```
<script>
    export default {
    data() {
        return {
        count: 0
        }
    }
    }
</script>

<template>
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>
```

When not using a build step, a Vue component can be defined as a plain JavaScript object containing Vue-specific options:

```
export default {
  data() {
    return {
      count: 0
    }
  },
  template: `
    <button @click="count++">
      You clicked me {{ count }} times.
    </button>`
}
```

The example above defines a single component and exports it as the default export of ```.js``` file, but you can use named exports to export multiple components from the same file.

## Using a component

To use a child component, we need to import it in parents component. Assuming we placed our counter component inside a file called ```ButtonCounter.vue```, the component will be exposed as the file's default export:

``` 
<script>
    import ButtonCounter from './ButtonCounter.vue'

    export default {
        components: {
            ButtonCounter
        }
    }
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

To expose the imported component to our template, we need to register it with ```components``` option. With it, the component tag will be available to be used.

In SFC's is recommended that we use ```PascalCase``` (```CamelCase```) tag names for a child component to differentiate from native HTML elements. Although native HTML element is case insensitive, Vue SFC is a compiled format so we're able to use case-sensitive tag names in it. We're also able to use ```/>``` to close a tag.

If you're authoring your template directly in a DOM (e.g. as the content of a native ```<template>``` element), the template will be subject to the browser's native HTML parsing, and in this case you can use ```snake-case``` and explicit closing tags for components: 
``` 
<!-- if this template is written in the DOM -->
<button-counter></button-counter>
<button-counter></button-counter>
<button-counter></button-counter>
```

## Parsing props

Props are custom attributes you can register on a component. For example, we have a blog that we want to maintain the same visual to all posts, so in order to do this we can create a component that accepts blog post as a prop. To do this in Vue we can use the ```props``` option:

``` 
<!-- BlogPost.vue -->
<script>
    export default {
        props: ['title']
    }
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

When a value is passed to a prop it becomes a property in the component. The value of the prop is accessible within its template and on the component's using ```this``` context, just like any other component property.

A component can have as many props as you want and, by default, it can have any value.

Once a prop is registered, you can pass data to it as a custom attribute, like this:

``` 
<BlogPost title="My journey with Vue" />
<BlogPost title="Blogging with Vue" />
<BlogPost title="Why Vue is so fun" />
```

In a typical app, however, you'll likely have an array of posts in your parents component:

``` 
export default {
  // ...
  data() {
    return {
      posts: [
        { id: 1, title: 'My journey with Vue' },
        { id: 2, title: 'Blogging with Vue' },
        { id: 3, title: 'Why Vue is so fun' }
      ]
    }
  }
}
```

Then want to render a component for each one, using  ```v-for```:

``` 
<BlogPost
  v-for="post in posts"
  :key="post.id"
  :title="post.title"
 />
```

Notice how ```v-bind``` syntax (```:title="post.title```) is used to pass a dynamic prop values. This is especially used when you don't know the exact content you're going to render ahead of time.

## Listening to Events

As we developed ```<BlogPost>``` component, some feature may require communicating back up to the parent. For example, we may decide to include an accessibility feature to enlarge text of blog post, while leaving the rest of the page of its default size.


In the parent, we can support this feature by using a ```postFontSize``` data property:

``` 
data() {
  return {
    posts: [
      /* ... */
    ],
    postFontSize: 1
  }
}
```

Which can be used in the template to control the font size of all blog posts:

``` 
<div :style="{ fontSize: postFontSize + 'em' }">
  <BlogPost
    v-for="post in posts"
    :key="post.id"
    :title="post.title"
   />
</div>
```

Now let's add a button to the  ```<BlogPost>``` component's template:

``` 
<!-- BlogPost.vue, omitting <script> -->
<template>
  <div class="blog-post">
    <h4>{{ title }}</h4>
    <button>Enlarge text</button>
  </div>
</template>
```

As the button doesn't do anything wet, we can solve this problem using a custom events system. The parent can choose to listen to any event on the child component instance using ```v-on``` or ```@```, just as we would with native DOM events:

``` 
<BlogPost
  ...
  @enlarge-text="postFontSize += 0.1"
 />
```

Then the child component can emit an event by using a built-in ```$$emit``` method, passing the name of the event:

``` 
<!-- BlogPost.vue, omitting <script> -->
<template>
  <div class="blog-post">
    <h4>{{ title }}</h4>
    <button @click="$emit('enlarge-text')">Enlarge text</button>
  </div>
</template>
```

Thanks to ```@enlarge-text="postFontSize += 0.1"``` listener, the parent will receive an event and update the ```postFontSize```.

We can optionally declare emitted events using the ```emits``` option:

``` 
<!-- BlogPost.vue -->
<script>
    export default {
        props: ['title'],
        emits: ['enlarge-text']
    }
</script>
```

This documents all the events that the component emits and optionally validate them. It also allows Vue to implicitly applying them as native listeners to the child component's root element.

## Content Distribution with Slots

Just like with HTML elements, it's often useful to be able to pass content to a component, like this:

``` 
<AlertBox>
  Something bad happened.
</AlertBox>
```

This can be achieved by using ```<slot>``` element:

``` 
<!-- AlertBox.vue -->
<template>
  <div class="alert-box">
    <strong>This is an Error for Demo Purposes</strong>
    <slot />
  </div>
</template>

<style scoped>
.alert-box {
  /* ... */
}
</style>
```

## Dynamic components

Sometimes, it's useful to dynamically components, like in a tabbed interface. It's possible by using Vue's ```<component>``` element with a special ```is``` attribute:

``` 
<!-- Component changes when currentTab changes -->
<component :is="currentTab"></component>
```

In the above example, the value passed to ```:is``` can contain either:

- the name string of a registered component, OR
- the actual imported component object

You can also use ```is``` attribute to create regular HTML elements.

When switching between multiple components with ```<component :is="...">```, a component will be unmounted when it's switched away from. We can force the inactive components to stay "alive" with the built-in ```<KeepAlive>``` component.


