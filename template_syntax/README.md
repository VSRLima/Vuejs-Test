# Template Syntax
Vue uses an HTML-based syntax that allows you to declaratively bind the rendered DOM to the underlying component instance's data.

## Text Interpolation
The most basic form of data binding is text interpolation using "mustache syntax" (double curly braces):

```
<span> Message: {{msg}} </span>
```

## Raw HTML
The double mustache interpret data as plain text, not HTML. In order to output real HTML, you'll need to use ```v-html``` directive: 

```
//template
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>

//fake result
Using text interpolation: <span style="color: red">This should be red.</span>

Using v-html directive: This should be red.
```

The content of ```span``` will be replaced with plain HTML - data bindings are ignored. Note that you cannot use ```v-html``` to compose template partials, because Vue isn't a string-based template engine. Instead, components are preferred as the fundamental unit for UI reuse and composition.

### Security Warning
Dynamically rendering arbitrary HTML on your site can be very dangerous since it can lead to XSS vulnerabilities. Only use ```v-html``` on trusted content and never on user-provided content.

## Attribute Bindings
Since mustaches can't be used inside of HTML attributes, use a ```v-bind``` directive:

```
<div v-bind:id="dynamicId"></div>
```

The ```v-bind``` directive instructs Vue to keep the element's ```id``` attribute in sync with the component's ```dynamicId``` property. If the bound value is ```null``` or ```undefined```, then the attribute will be removed from the rendered element.

### Shorthand
Because ```v-bind``` is so commonly used, it has a dedicated shorthand syntax:

```
<div :id="dynamicId"></div>
```

The ```:``` is a valid character for attribute names and all Vue-supported browsers can parse it correctly. In addition, they do not appear in the final rendered markup.

### Same-name Shorthand
If the attribute has the same name with the JavaScript value being bound, the syntax can be further shortened to omit the attribute value:

```
<!-- same as :id="id" -->
<div :id></div>

<!-- this also works -->
<div v-bind:id></div>
```

Note this feature that is only available in Vue 3.4 and above.

## Dynamically Binding Multiple Attributes
If you have a JavaScript object representing multiple attributes that looks like this:
```
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
  style: 'background-color:green'
}
```

You can bind them to a single element by using ```v-bind``` without an argument:

```
<div v-bind="objectOfAttrs"></div>
```

### Using JavaScript Expression
Vue actually supports the full power of JavaScript expressions inside all data bindings:

```
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>
```

In Vue templates, JavaScript expressions can be used in the following positions:

- Inside text interpolations (mustaches)
- In the attribute value of any Vue directives (special attributes that stats with ```v-```)

### Calling Functions

It's possible to call a component-exposed method inside a binding expression:

```
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```

#### TIP
Functions called inside binding expressions can be called every time the component updates, so they should not have any side effects, such as changing data or triggering asynchoronous operations.

### Restricted Globals Access
Template expressions are sandboxed and only have access to a restricted list of globals. The list exposes commonly used built-in globals such as ```Math``` and ```Date```.

You can, however, explicitly define additional globals for all Vue expressions by adding them to ```app.config.globalProperties```.

## Directives
Directive attribute values are expected to be single JavaScript expressions, with the expection of ```v-for```, ```v-on``` and ```v-slot```. A directive's job is to reactively apply updates to the DOM when the value of its expressions changes. Take ```v-if``` as an example:

``` 
<p v-if="seen">Now you see me</p>
```

### Arguments
Some directives can take an "argument", denoted by a colon after the directive name. For example, the ```v-bind``` directive is used to reactively update a HTML attribute:

``` 
<a v-bind:href="url"> ... </a>

<!-- shorthand -->
<a :href="url"> ... </a>
```

Here, ```href``` is the argument, which tells the ```v-bind``` directive to bind the element's ```href``` attribute to the value of the expression ```url```.

Another example is the ```v-on``` directive, which listens to DOM events:

``` 
<a v-on:click="doSomething"> ... </a>

<!-- shorthand -->
<a @click="doSomething"> ... </a>
```

Here, the argument is the event name to listen to: ```click```. ```v-on``` has a corresponding shorthand, namely the ```@``` character.

### Dynamic Arguments
It's also possible to use a JavaScript expression in a directive argument by wrapping it with square brackets:

``` 
<!--
Note that there are some constraints to the argument expression,
as explained in the "Dynamic Argument Value Constraints" and "Dynamic Argument Syntax Constraints" sections below.
-->
<a v-bind:[attributeName]="url"> ... </a>

<!-- shorthand -->
<a :[attributeName]="url"> ... </a>
```

Here, ```attributeName``` will be dynamicaly changed and its value will be used as the final value for the argument.

You can also use dynamic arguments to bind a handler to a dynamic event name:

``` 
<a v-on:[eventName]="doSomething"> ... </a>

<!-- shorthand -->
<a @[eventName]="doSomething"> ... </a>
```

#### Dynamic Argument Value Constraints
Dynamic arguments are expected to evaluate to a string, with the exception of ```null```.
The special value ```null``` can be used to explicitly remove the binding. Any other non-string value will trigger a warning.

#### Dynamic Argument Syntax Constraints
Dynamic argument expressions have some syntax constraints because certain charactes, such as spaces and quotes, are invalid in HTML attribute names. For example, this one bellow:

``` 
<!-- This will trigger a compiler warning. -->
<a :['foo' + bar]="value"> ... </a>
```

If you need to pass a complex dynamic argument, it's probably better use a computed property, which we'll cover shortly.

When using in-DOM templates (templates directly written in a HTML file), you should avoid naming keys with uppercase characters, as browsers will coerce attribute names into lowercase:

``` 
<a :[someAttr]="value"> ... </a>
```

The above will converted to ```:[someattr]``` in in-DOM templates. If your component has a ```someAttr``` property instead of ```someattr```, your code won't work.

### Modifiers
Modifiers are special postfixes denotes by a dot, which indicate that a directive should be bound in some special way. For example, the ```.prevent``` modifier tells the ```v-on``` directive to call ```event.preventDefault()``` on the triggered event:

``` 
<form @submit.prevent="onSubmit">...</form>
```