# Reactivity Fundamentals

- Declaring Reactive State

`ref ()` is the recommended way of declaring reactive state and it could be like it:

```
    import { ref } from 'vue'

    const count = ref(0)
```

So basically the ref takes an argument and returns a wrapped within a ref object with a prop `.value` :

```
    const count = ref(0)

    console.log(count) // { value: 0 }
    console.log(count.value) // 0

    count.value++
    console.log(count.value) // 1
```

Note: There’s a way in TS to properly type the [refs](https://vuejs.org/guide/typescript/composition-api#typing-ref)

In case you want to access refs in a component’ template, declare a return them from a component’s `setup()`:

```
    import { ref } from 'vue'

    export default {
    // `setup` is a special hook dedicated for the Composition API.
    setup() {
            const count = ref(0)

            // expose the ref to the template
            return {
            count
            }
        }
    }

    <div>{{ count }}</div>
```

Notice that in this case you don’t need to append `.value` when using the ref in the template. For convenience, refs are automatically unwrapped when used inside templates (with a few (caveats)['src/components/reactive_components/caveants.md'])

You can also  mutate a ref directly in event handlers:

```
    <button @click="count++">
        {{ count }}
    </button>
```

For more complex logic, we can declare functions that mutates refs in the same scope and expose them as methods alongside the state:

```
    import { ref } from 'vue'

    export default {
        setup() {
            const count = ref(0)

            function increment() {
            // .value is needed in JavaScript
            count.value++
            }

            // don't forget to expose the function as well.
            return {
            count,
            increment
            }
        }
    }
```

Exposed methods can be used as event handlers: 

```
    <button @click="increment">
        {{ count }}
    </button>
```

## `<script setup>`

Manually expose states and methods using `setup()` can be a little verbose, in order to fix this we can use Single-File Components (SFCs). We can simplify the usage to this `<script setup>`:

```
    <script setup>
        import { ref } from 'vue'

        const count = ref(0)

        function increment() {
            count.value++
        }
    </script>

    <template>
        <button @click="increment">
            {{ count }}
        </button>
    </template>
```

Top-level imports, variables and functions declared in `<script setup>` are automatically usable in the template in the same component. Think about this like as a JavaScript template where we declare a function in the same scope - It naturally has access to everything declared alongside it.

## Why refs?

When you use a ref in a template, and change the refs's value later, Vue automatically detects the change and update the DOM accordingly. This is made possible with dependency-tracking based on reactivity system. When a component is rendered for the first time, Vue tracks every ref that was used during the render. Later on, when a ref is mutated, it'll trigger and re-render the component that are tracking it.

In a stantard Javascript code there's no access and updating plain variables. However, you can do this by using get and set methods.

The `.value` gives you the opportunity to detect when a ref has been access and mutated. Under the hood, Vue performs the tracking in its getter, and performs the triggers in its setter. Conceptually, you can think of a ref as an object that looks like this:

```
    // pseudo code, not actual implementation
    const myRef = {
        _value: 0,
        get value() {
            track()
            return this._value
        },
        set value(newValue) {
            this._value = newValue
            trigger()
        }
    }
```

Another nice trait of refs is that unlike plain variables, you can pass ref into functions while retaing access to the lastest value and the reactivity connection. This is particulary useful when refactoring complex logic into a resuable code.

The reactivity system is discussed in more details in the (Reactivity in Depth)[src/components/reactive_components/reactivityInDepths.md] section.

## Deep Reactivity

Refs can hold any type of data, including deep nested objects, arrays, or JavaScript built-in data structures like ```Map```.

A ref will make it deeply reactive. This means you can expect changes to be detected even when you mutate nested objeccts or arrays:

```
    import { ref } from 'vue'

    const obj = ref({
        nested: { count: 0 },
        arr: ['foo', 'bar']
    })

    function mutateDeeply() {
        // these will work as expected.
        obj.value.nested.count++
        obj.value.arr.push('baz')
    }
```

Non-primitive values are turned into reactive proxies via ```reactive()```, which is included below.

It is also possible to opt-out of deep reactivity with ```shallow refs```. For shallow refs, only ```.value``` access is tracked for reactivity. Shallow refs can be used for optimizing performance by avoiding the observation cost of large objects, or in cases where the inner state is managed by an external library.

## DOM Update Timing

When you mutate reactive state, the DOM is automatically updated. However, it should be noted that DOM updates are not applied synchronously. Instead, Vue buffers them to wait until ```nextTick()``` in the update cycle to ensure that each component updates only once no matter how many state changes you have made.

To wait for DOM update to complete after a state change, you can use ```nextTick()``` global API: 

```
    import { nextTick } from 'vue'

    async function increment() {
        count.value++
        await nextTick()
        // Now the DOM is updated
    }
```

## reactive()

There is another way of declaring reactive state, with ```reactive()``` API. Unlike a ref which wraps the inner value into a special object, the ```reactive()``` makes the object itself reactive:

```
    import { reactive } from 'vue'

    const state = reactive({ count: 0 })
```

Usage in template:

```
    <button @click="state.count++">
        {{ state.count }}
    </button>
```

Reactive objects are like ```JavaScript Proxies``` and behave just like normal objects. The difference is that Vue is able to intecept the access and mutation of all properties of a reactive object for reactive tracking and triggering.

```reactive()``` converts the object deeply: nested objects are also wrapped with ```reactive()``` when accessed. It is also called by ```ref()``` internally when the ref value is an object. 

### Reactive Proxy vs Original

Its important to notice that the returned value from ```reactive()``` is a Proxy of the original object, which is not equal to the original object:

```
    const raw = {}
    const proxy = reactive(raw)

    // proxy is NOT equal to the original.
    console.log(proxy === raw) // false
```

Only the proxy object is reactive, mutating the original object won't trigger updates.
Therefore, the best practice when working with Vue's reactivity system is to exclusively use proxied versions of our state.

To ensure consistent access to proxy, calling ```reactive()``` on the same object always returns the same proxy, and calling ```reactive()``` on a existing proxy will also returns the same proxy:

```
    // calling reactive() on the same object returns the same proxy
    console.log(reactive(raw) === proxy) // true

    // calling reactive() on a proxy returns itself
    console.log(reactive(proxy) === proxy) // true
```

This rule is also applied to nested objects. Due the deep reactivity, nested objects inside a reactive object are also proxies:

```
    const proxy = reactive({})

    const raw = {}
    proxy.nested = raw

    console.log(proxy.nested === raw) // false
```

### Limitations of reactive()

The ```reactive()``` API has a few limitations:

1. Limited value types: it only works for object types (objects, arrays, collection types like ```Map``` and ```Set```). It cannot hold primitive types such as ```string```, ```number``` or ```boolean```.

2. Cannot replace entire object: since Vue's reactivity tracking works over property access, we must always keep the same reference to the reactive object. This means that we cannot easily "replace" a reactive object because the reactivity reference for the first one is lost: 

```
    let state = reactive({ count: 0 })

    // the above reference ({ count: 0 }) is no longer being tracked
    // (reactivity connection is lost!)
    state = reactive({ count: 1 })
```

3. Not destructure-friendly: when we destructure a reactive object's primitive type into local variables, or when we pass that property to a function, we'll lose the reactivity connection: 

```
    const state = reactive({ count: 0 })

    // count is disconnected from state.count when destructured.
    let { count } = state
    // does not affect original state
    count++

    // the function receives a plain number and
    // won't be able to track changes to state.count
    // we have to pass the entire object in to retain reactivity
    callSomeFunction(state.count)
``` 

Due these limitations its recommend the usage of ```ref()``` as primary API for declaring reactive state.

## Additional Ref Unwrapping Details

### As Reactive Object Property

As ref is automatically unwrapped when access or mutated as a property of a reactive object. In other words, it behave like a normal property:

```
    const count = ref(0)
    const state = reactive({
    count
    })

    console.log(state.count) // 0

    state.count = 1
    console.log(count.value) // 1
```

If a new ref is assigned to a property linked to a existing ref, the old ref will be replaced by the new one:

```
    const otherCount = ref(2)

    state.count = otherCount
    console.log(state.count) // 2
    // original ref is now disconnected from state.count
    console.log(count.value) // 1
```

Ref unwrapping only happens when nested inside of a deep reactive object. It does not apply when it accessed as a property of a shallow reactive object.

### Caveants in Arrays and Collections

Unlike reactive objects, there isn't unwrapping performed when the ref is accessed as an element of a reactive array or native collection type like ```Map```:

```
    const books = reactive([ref('Vue 3 Guide')])
    // need .value here
    console.log(books[0].value)

    const map = reactive(new Map([['count', ref(0)]]))
    // need .value here
    console.log(map.get('count').value)
```

### Caveants when Unwrapping in Templates

Ref unwrapping in templates only applies if ref is a top-level property in template render context.

In the example below, ```count``` and ```object``` are top-level properties, but ```object.id``` is not:

```
    const count = ref(0)
    const object = { id: ref(1) }
```

Therefore, this expression below works as expected: 

```{{ count + 1 }}```

...while this one doesn't: 

```{{ object.id + 1 }}```

The rendered result will be ```[object Object]1``` because ```object.id``` isn't unwrapped when evaluating the expression and remains a ref object . To fix this, we can destructuure ```id``` into top-level property:

```const { id } = object```
``` {{ id + 1 }}```

Now the render will be 2.

Another thing to note is that a ref does get unwrapped if it is the final evaluated value of a text interpolation (i.e. a {{ }} tag), so the following will render 1:

```{{ object.id }}```

This is just a convenience feature of text interpolation and is equivalent to ```{{ object.id.value }}```.