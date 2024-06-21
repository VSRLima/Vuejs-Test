# Form Input Binding

To sync state of a form input elements we can use ```v-model``` directive, as for example:

```
//Vanilla Javascript
<input
  :value="text"
  @input="event => text = event.target.value">


//Vue
<input v-model="text">
```

In addition, the ```v-model``` can be used on inputs of different types, ```<textarea>```, and ```<select>``` elements. It can automatically expands to different DOM property and events pair based on element it's used on:

- ```<input>``` with text type and ```<textarea>``` elements use ```value``` and ```input``` event;
- ```<input type="checkbox">``` and ```<input type="radio">``` use ```checked``` property and ```changed``` event;
- ```<select>``` uses ```value``` as a prop and ```change``` as an event.


## Basic Usage

### Text
```
<p>Message is: {{ message }}</p>
<input v-model="message" placeholder="edit me" />
```

### Multiline text

```
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<textarea v-model="message" placeholder="add multiple lines"></textarea> 
```

Note that the interpolation on work inside textarea, instead use ```v-model```:

```
<!-- bad -->
<textarea>{{ text }}</textarea>

<!-- good -->
<textarea v-model="text"></textarea> 
```


### Checkbox

```
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{{ checked }}</label> 
```

We can also bind multiple checkboxes to the same array or ```Set``` value:

```
//js
export default {
  data() {
    return {
      checkedNames: []
    }
  }
} 

//template
<div>Checked names: {{ checkedNames }}</div>

<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>

<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>

<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
```

In this case, the ```checkNames``` will always have the value of the checked checkboxes.

### Radio

``` 
<div>Picked: {{ picked }}</div>

<input type="radio" id="one" value="One" v-model="picked" />
<label for="one">One</label>

<input type="radio" id="two" value="Two" v-model="picked" />
<label for="two">Two</label>
```

### Select

```
<div>Selected: {{ selected }}</div>

<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select> 
```

Multiple select (bound to array):

``` 
<div>Selected: {{ selected }}</div>

<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

Select options can be dynamically rendered with ```v-for```:

``` 
//js
export default {
  data() {
    return {
      selected: 'A',
      options: [
        { text: 'One', value: 'A' },
        { text: 'Two', value: 'B' },
        { text: 'Three', value: 'C' }
      ]
    }
  }
}

//template
<select v-model="selected">
  <option v-for="option in options" :value="option.value">
    {{ option.text }}
  </option>
</select>

<div>Selected: {{ selected }}</div>
```

## Value Bindings 
For radio, checkbox and select options, the ```v-model``` binding values are usually static string (or booleans for checkboxes):

```
<!-- `picked` is a string "a" when checked -->
<input type="radio" v-model="picked" value="a" />

<!-- `toggle` is either true or false -->
<input type="checkbox" v-model="toggle" />

<!-- `selected` is a string "abc" when the first option is selected -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select> 
```

When we want to bind it to a dynamic property we can use ```v-bind``` to achieve it. In addition, using ```v-bind``` we can bind non-string values 

### Checkbox

```
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no" /> 
```

```true-value``` and ```false-value``` are Vue-specific attribute that only works with ```v-model```. Here the ```toggle``` will be ```yes``` when the box is checked and ```no``` for unchecked. In case you want to bind it dynamically you can do the following:

``` 
<input
  type="checkbox"
  v-model="toggle"
  :true-value="dynamicTrueValue"
  :false-value="dynamicFalseValue" />
```

### Radio

```
<select v-model="selected">
  <!-- inline object literal -->
  <option :value="{ number: 123 }">123</option>
</select>
```

### Modifiers

#### ```.lazy```
By default, ```v-model``` syncs the input with the data after each ```input``` event. You can add ```.lazy``` modifier to instead sync after ```change``` events:

```

<input v-model.lazy="msg"/> 
```

#### ```.number```
If you want user input to be automatically typecast as number, you can add the ```number``` modifier to your ```v-model``` managed input:

```
<input v-model.number="age" /> 
```

If the value can be parsed using ```parseFloat()``` it'll using the original value instead.
The ```number``` modifier is applied automatically it the input has ```type="number"```.

#### ```.trim```
If you want the whitespace be automatically trimmed, you can add ```.trim``` modifier to your ```v-model```-managed inputs:

``` 
<input v-model.trim="msg" />
```

