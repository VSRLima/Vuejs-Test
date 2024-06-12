# Event Handling

## Inline Handlers

Inline handlers are as used to in HTML/JS, for example:

```
//js
data() {
  return {
    count: 0
  }
} 

//template
<button @click="count++">Add 1</button>
<p>Count is: {{ count }}</p>
```

## Method Handlers
The logic sometimes is more complex, so inline handlers won't handle this kind of complexity. That's why ```v-on``` accepts the name or path to component method you'd like to call.

```
//js 
data() {
  return {
    name: 'Vue.js'
  }
},
methods: {
  greet(event) {
    // `this` inside methods points to the current active instance
    alert(`Hello ${this.name}!`)
    // `event` is the native DOM event
    if (event) {
      alert(event.target.tagName)
    }
  }
}

//template
<!-- `greet` is the name of the method defined above -->
<button @click="greet">Greet</button>
```

A method handler receives a native DOM Event object that triggers it in the example above, we're able to access the element by being dispatched by ```event.target```.

### Method X Inline Detection
The template compiler detects methods handlers by checking whether the ```v-on``` value string is a value JavaScript Identifier or property access path. For example, ```foo```. ```foo.bar``` or ```foo['bar']``` are treated as method handlers, while ```foo()``` and ```count++``` are treated as inline handlers.

## Event Modifiers
It's a very common need to call ```event.preventDefault()``` or ```event.stopPropagation()``` inside of event handlers. Although we can do this easily inside methods, it would be better if the methods can be purely about data logic than dealing with DOM event details.

To address this problem, Vue provides event modifiers for ```v-on```. Recall that modifiers are directive postfixes denoted by a dot.

- ```.stop```
- ```.prevent```
- ```.self```
- ```.capture```
- ```once```
- ```.passive```

``` 
<!-- the click event's propagation will be stopped -->
<a @click.stop="doThis"></a>

<!-- the submit event will no longer reload the page -->
<form @submit.prevent="onSubmit"></form>

<!-- modifiers can be chained -->
<a @click.stop.prevent="doThat"></a>

<!-- just the modifier -->
<form @submit.prevent></form>

<!-- only trigger handler if event.target is the element itself -->
<!-- i.e. not from a child element -->
<div @click.self="doThat">...</div>
```

### TIP
Order matters while using event modifiers, as in some cases we could use ```@click.prevent.self``` and in this case it will be read as it'll prevent click's default action on the element itself and its children, while ```@click.self.prevent``` will only prevent click's default action on the element itself.

The ```.capture```, ```.once``` and ```.passive``` modifiers mirror the options of the native ```addEventListener``` method:

```
<!-- use capture mode when adding the event listener     -->
<!-- i.e. an event targeting an inner element is handled -->
<!-- here before being handled by that element           -->
<div @click.capture="doThis">...</div>

<!-- the click event will be triggered at most once -->
<a @click.once="doThis"></a>

<!-- the scroll event's default behavior (scrolling) will happen -->
<!-- immediately, instead of waiting for `onScroll` to complete  -->
<!-- in case it contains `event.preventDefault()`                -->
<div @scroll.passive="onScroll">...</div>
```

The ```.passive``` modifier is typically used with touch events to improve perfomance on mobile devices.

### TIP
Do not use ```.passive``` and ```.prevent``` together, because ```.passive``` already indicates to the browser that you do not intend to prevent event's default behavior, and you'll likely see a warning from the browser if you do so.

## Key Modifiers
When listing to keyboard events, we often need to check a specific keyboard key. Vue allows adding key modifiers for ```v-on``` or ```@``` when listening for key events:

``` 
<!-- only call `submit` when the `key` is `Enter` -->
<input @keyup.enter="submit" />
```

You can directly use any valid key names exposed via ```KeyboardEvent.key``` as modifiers by converting them to kebab-case.

```
<input @keyup.page-down="onPageDown" /> 
```

In the above example, the handler will only be called if ```$event.key``` is equal to ```PageDown```.

### Key Aliases
Vue provides aliases some most commonly used keys:

- ```.enter```
- ```.tab```
- ```.delete``` (captures both "Delete" and "Backspace" keys)
- ```.esc```
- ```.space```
- ```.up```
- ```.down```
- ```.left```
- ```.right```

### System Modifiers Keys
You can use the following modifiers to trigger mouse or keyboard event listeners only when the corresponding modifier key is pressed:

- ```.ctrl```
- ```.alt```
- ```.shift```
- ```.meta```

For example:

```
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div> 
```

## ```.exact``` Modifier
The ```.exact``` modifier allows control of the exact combination of system modifiers needed to trigger an event.

```
<!-- this will fire even if Alt or Shift is also pressed -->
<button @click.ctrl="onClick">A</button>

<!-- this will only fire when Ctrl and no other keys are pressed -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- this will only fire when no system modifiers are pressed -->
<button @click.exact="onClick">A</button> 
```

## Mouse Button Modifiers

- ```.left``` 
- ```.right```
- ```.middle```

These modifiers restrict the handler to events triggered by a specific mouse button.