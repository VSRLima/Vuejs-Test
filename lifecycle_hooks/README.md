# Lifecycle Hooks

## Registering Lifecycle Hooks

For example, the ```mounted``` hook can be used to run code after the component has finished the initial rendering and created the DOM nodes:

``` 
export default {
    mounted () {
        console.log(`the component is now mounted`)
    }
}
```

There are also other hooks which will be called at different stages of the instance's lifecycle, with the most commonly used being ```mounted```, ```updated``` and ```unmounted```.

All lifecycle hooks are called with their ```this``` context pointing to the current active instance invoking it. Note this means that you should avoid using arrow functions when declaring lifecycle hooks, as you won't be able to access the component instance via ```this``` if you do so.

## Lifecycle Diagram

``` mermaid
    flowchart TD
    id1(Renderer encounters component) -- setup (Composition API) and beforeCreated --> A[Init Options API] -- created --> B{Has pre-compiled template?} 
    B -->|
    YES 
    before Mount
    | C[Initial Render create & insert DOM nodes]
    C --> D[Mounted]
    B -->|NO| E[Compile template on-the-fly]
    E -- beforeMount --> C
    D -. when data changes and beforeUpdate .-> F[re-render and patch]
    F -. updated .-> D
    D -- beforeUnmount --> G[Unmounted]
```