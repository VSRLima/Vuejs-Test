# Template Refs

While Vue's declarative rendering model abstracts away most of the direct DOM operations for you, there may still cases where you'll need direct access to the underlying DOM elements. To achieve this, we can use the special ```ref``` attribute:

```
<input ref="input"> 
```

```ref``` is a special attribute, similar to ```key```. It allows you to obtain a direct reference to a specific DOM element or child component instance after it's mounted. This may be useful when you want to, for example, programmatically focus an input on a component mount, or initialize a 3rd party library on a specific element.

## Accessing Refs

The resulting ref is exposed on ```this.$refs```:

```
<script>
export default {
  mounted() {
    this.$refs.input.focus()
  }
}
</script>

<template>
  <input ref="input" />
</template> 
```

Note that you can only access the ref after the component is mounted. If you try to access ```$refs.input``` in a template expression, it'll be ```undefined``` on the first render.
This because the element doesn't exist until after the first render!

## Refs inside ```v-for```

When ```ref``` is used inside ```v-for```, the resulting ref value will be an array containing the corresponding elements:

``` 
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ]
    }
  },
  mounted() {
    console.log(this.$refs.items)
  }
}
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

It should be noted that the ref array doesn't guarantee the same order as the source array.

## Function Refs

Instead of a string key, the ```ref``` attribute can also be bound to a function, which will be called on each component update and gives you a full flexibility on where to store the element reference. The functions receives the element reference as the first argument:

```
<input :ref="(el) => { /* assign el to a property or ref */ }"> 
```

When the component is unmounted, the argument will be ```null```. You can, of course, use a method instead of an inline function.


## Ref on Component

```ref``` can also be used on child component. In this case the reference will be that of a component instance:

``` 
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child will hold an instance of <Child />
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

The reference instance will be identical to the child component's ```this```, which means the parent component will have full access to every property and method of the child component. This makes it easy to create tightly coupled implementation details between the parent and child, so components refs should be only used when absolutely needed - in most cases, you should try to implement parent / child interactions using the standard props and emit interfaces first.

The ```expose``` option can be used to limit access to a child instance:

```
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() {
      /* ... */
    },
    privateMethod() {
      /* ... */
    }
  }
} 
```
