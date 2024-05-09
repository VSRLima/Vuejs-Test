# How reactive works in vue

Actually we have two principal ways of intercepts changes in JavaScript: getter/setter and Proxies. Vue2 uses getter/setter because of browser limitations. In Vue3, Proxies are used for reactive objects and getter/setters are used for refs. Just below are some pseudo-code showing how they works:

```
    function reactive(obj) {
        return new Proxy(obj, {
            get(target, key) {
            track(target, key)
            return target[key]
            },
            set(target, key, value) {
            target[key] = value
            trigger(target, key)
            }
        })
    }

    function ref(value) {
        const refObject = {
            get value() {
            track(refObject, 'value')
            return value
            },
            set value(newValue) {
            value = newValue
            trigger(refObject, 'value')
            }
        }
        return refObject
    }
```

This explains a few limitations in reactive objects that, and they are:

- When you assign or destructure a reactive object's property to a local variable, accessing or assigning to that variable is non-reactive because it no longer triggers the get/set proxy traps on the source object. Note this "disconnect" only affects the variable binding - if the variable point to a non-primitive value such as object, mutating the object would still be reactive.

- The returned proxy from ```reactive()```, although behaving just like the original, has a different identity if we compare it to the original using ```===``` operator.

Inside of ```track()```, we check whether there is a currently running effect. If there's one, we lookup for the subscriber effects (stored in Set) for the property being tracked, and add the effects to the Set:

```
    // This will be set right before an effect is about
    // to be run. We'll deal with this later.
    let activeEffect

    function track(target, key) {
    if (activeEffect) {
            const effects = getSubscribersForProperty(target, key)
            effects.add(activeEffect)
        }
    }
```

Effects subscriptions are stored in a global ```WeakMap<target, Map<key, Set<effect>>>``` data structure. If no subscriptions effects Set was found for a property (tracked for the first time), it will be created. This is what the ```getSubscribersForProperty()``` does, in short.

Inside ```trigger()```, we again lookup the subscriber effects for the property. But this time we invoke them instead: 

```
    function trigger(target, key) {
        const effects = getSubscribersForProperty(target, key)
        effects.forEach((effect) => effect())
    }
```

Now let's circle back to function ```whenDeepsChange```:

```
    function whenDepsChange(update) {
    const effect = () => {
            activeEffect = effect
            update()
            activeEffect = null
        }
        effect()
    }
```

It wraps the raw ```update()``` function in an effect that sets itself as current active effect before running the actual update. This enables ```track()``` calls during the update to locate the current active effect.

At this point, we have created an effect that automatically tracks its dependencies, and re-run whenever a dependency changes. We call this Reactive Effect.

Vue provides an API that allows you to create reactive effects: ```watchEffects()```. In fact, you may have noticed that it works pretty similar to the magical ```whenDeepsChanges()``` in the example. We can now rework the original example with Vue's API:

```
    import { ref, watchEffect } from 'vue'

    const A0 = ref(0)
    const A1 = ref(1)
    const A2 = ref()

    watchEffect(() => {
        // tracks A0 and A1
        A2.value = A0.value + A1.value
    })

    // triggers the effect
    A0.value = 2
```

Using a reactive effects in ref isn't the best way of doing it. Instead, using computed property makes more sense:

```
    import { ref, computed } from 'vue'

    const A0 = ref(0)
    const A1 = ref(1)
    const A2 = computed(() => A0.value + A1.value)

    A0.value = 2
```

Internally, the ```computed``` manages its invalidation and re-computation using reactive effects.

## Reactivity debugging

### Component Debugging Hooks

We can debug what dependencies are used during a component's render and which dependency is triggering an update using ```onRenderTracked``` and ```onRenderTriggered``` lifecycle hooks. Both hooks will receive a debugger event which contains information on dependency in question. Its recommend to place a ```debugger``` statement in the callbacks interactively inspect the dependency:

```
    <script setup>
        import { onRenderTracked, onRenderTriggered } from 'vue'

        onRenderTracked((event) => {
            debugger
        })

        onRenderTriggered((event) => {
            debugger
        })
    </script>
```

The debug event objects have the following format: 

```
    type DebuggerEvent = {
        effect: ReactiveEffect
        target: object
        type:
            | TrackOpTypes /* 'get' | 'has' | 'iterate' */
            | TriggerOpTypes /* 'set' | 'add' | 'delete' | 'clear' */
        key: any
        newValue?: any
        oldValue?: any
        oldTarget?: Map<any, any> | Set<any>
    }
```

### Computed Debugging

We can debug the computed property by passing ```computed()``` a second options object with ```onTrack``` and ```onTrigger``` callbacks:

- ```onTrack``` will be called when a property or ref is tracked as a dependency.
- ```onTrigger``` will be called when the watcher callback is triggered by the mutation of a dependency.

Both callbacks will receive the same object format as component debug hooks:

```
    const plusOne = computed(() => count.value + 1, {
        onTrack(e) {
            // triggered when count.value is tracked as a dependency
            debugger
        },
        onTrigger(e) {
            // triggered when count.value is mutated
            debugger
        }
    })

    // access plusOne, should trigger onTrack
    console.log(plusOne.value)

    // mutate count.value, should trigger onTrigger
    count.value++
```

### Watcher Debugging

Similiar to ```computed()```, watchers also supports ```onTrack``` and ```onTrigger``` options: 

```
    watch(source, callback, {
        onTrack(e) {
            debugger
        },
        onTrigger(e) {
            debugger
        }
    })

    watchEffect(callback, {
        onTrack(e) {
            debugger
        },
        onTrigger(e) {
            debugger
        }
    })
```

