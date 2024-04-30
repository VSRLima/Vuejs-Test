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

The reactivity system is discussed in more details in the (Reactivity in Depth)[] section.