# Transition

```<Transition>``` for applying animations when an element or component is entering or leaving the DOM.
We can apply animations in Vue using other techniques such as toggling CSS classes or state-driven animations via style bindings. 

## The ```<Transition>``` Component

```<Transition>``` is a built-in component. It can be used to apply enter and leave animations on elements or components passed to it via its default slot. The enter or leave can be triggered by one of the following:

- Conditional rendering via ```v-if```
- Conditional display via ```v-show```
- Dynamic components toggling via the ```<component>``` special element.
- Changing the special ```key``` attribute.

This is an example of the most basic usage:

```
//template
<button @click="show = !show">Toggle</button>
<Transition>
  <p v-if="show">hello</p>
</Transition>

//css 
/* we will explain what these classes do next! */
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
```

#### TIP

```<Transition>``` only supports a single element or component as its slot content. If the content is a component, the component must also have only single root element.


When an element in a ```<Transition>``` component is inserted or removed, this is what happens:

1. Vue will automatically sniff whether the target element has CSS transitions or animations applied. If it does, a number of CSS transitions classes will be added / removed at appropriate timings. 

2. If there are listeners for JavaScript hooks, these hooks will be called at appropriate timings.

3. If no CSS transitions / animations are detected and no JavaScript hooks are provided, the DOM operations for insertion and/or removal will be executed on the browser's next animation frame.

## CSS-Based Transitions

### Transitions Classes 

There are six classes applied to enter / leave transitions.

1. ```v-enter-from```: Starting state for enter. Added before the element is inserted, removed one frame after element is inserted

2. ```v-enter-active```: Active state for enter. Applied during the entire entering phase. Added before the element is inserted, removed when the transition/animation finishes. This class can be used to define the duration, delay and easing curve for the entering transition.

3. ```v-enter-to```: Ending state for enter. Added one frame after element is inserted (at the same time ```v-enter-from``` is removed), removed when the transition/animation finishes. 

4. ```v-leave-from```: Starting state for leave. Added immediately when the leave transition is triggered, removed after one frame.

5. ```v-leave-active```: Active state for leave. Applied during the entire leaving phase. Added immediately when leaving transition is triggered, removed when the transition/animation finishes. This class can be used to define the duration, delay and easing curve for the leaving transition.

6. ```v-leave-to```: Ending state for leave. Added one frame after leaving transition is triggered (at the same time ```v-leave-from```), removed when the transition/animation finishes.

### Named Transitions 

