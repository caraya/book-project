# Revisiting Classes, looking at custom elements and Polymer 2

While at the Polymer Summit I had the chance to review some of the Custom Elements V1 specification and, to a lesser degree, the Shadow DOM V1 specification. 

They bring significant changes both to the way we build vanilla custom elements and Polymer-based elements.  We’ll look at ES2015 classes, how to build vanilla custom elements and then have a look at Polymer 2.0 (currently in Preview). 

## Revisiting ES2015 Classes

I was surprised to find classes as part of ES6…  Ever since the language was created developers have had to learn to work with prototypal inheritance, how to make that chain work for us and what are the pitfalls to avoid when doing so. The first question I asked when I started looking at ES6 was what are classes in ES6? 

They’re syntactic sugar on top of prototypical inheritance. Everything we’ll discuss in this section is built on top of the traditional prototypal inheritance framework and the parser will use that under the hood while we do all out shiny work with classes. 

This will be our example to look at how classes work. It defines a class `Person` with three default elements:

* `first` name
* `last`name
* `age`

It also defines 2 methods: 

* `fullName` returning the first and last name
* `howOld` returing the string `${this.name}` is  `${this.age}` years old

Note that the methods use [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) with string interpolations. The backticks are required 
for string literals and not an artifact of writing in Markdown
    

```javascript
class Person {
  constructor(first, last, age) {
    this.first = first;
    this.last = last;
    this.age = age;
  }

  fullName() {
   return `${this.first} ${this.last}`;
  }

  howOld() {
    return `${this.first} is ${this.age} years old`;
  }
}
```

To instantiate a new object of class Person we must use the new keyword like so:

```javascript
const carlos =  new Person('carlos', 'araya', 49);
```
Now we can use the class methods like this:

```javascript
carlos.fullName();
// returns "carlos araya"
carlos.howOld();
// returns "carlos is 49 years old"
```

## Subclasses, extends and super

Now let’s say that we want to expand on the Person class by assigning additional attributes to the constructor and add additional methods.  We could copy all the material from the Person class into our new class, call it Engineer, but Javascript saves us from having to do so. The `extends` keyword allows us to use a class as the basis for another one. 

To continue the example, we’ll extend `Person` to describe an `Engineer`. We’ll say that the Engineer is a person with all the attributes and methods of the Person class with 2 additions:

* They belong to a `department`
* They have a favorite programming language represented by the `lang` parameter

The Engineer class is presented below:

```javascript
  class Engineer extends Person {
    constructor(first, last, age, department, lang) {
      // super calls the parent class constructorfor the listed attributes
      super(first, last, age);
      this.department = department;
      this.language = lang;
    }

  departmentBelonging() {
     return `${this.first} is in the ${this.department} department`;
  }

  favoriteLanguage() {
     return `${this.first} favorite language is ${this.language}`;
  }
}
```

In the constructor we pass the same three values as for the Person Class but, rather than assign them parameters like we did in the Person class, we simply call the constructor of the parent class using the `super` keyword. 

The instruction `super(first, last, age)` tells the parser to hand the attributes to the parent class for processing.  The attributes that are particular to the Engineer class (`department` and `language`) are assigned normally. 

We then add the methods specific to the Engineer class: What department they belong to and what’s their favorite programming language. 

The instantiation is the same as before

```javascript
const william = new Engineer('William', 'Cameron', 49, 'engineering', 'C++');
```

Now the cool part. Even though we didn’t add the methods `fullName` and `howOld` to the engineer class we get them for free becaure they are defined in the parent class.  WIth william (defined above) we can do:

```javascript
william.fullName() 
// returns "William Cameron"

william.howOld() 
// returns "William is 49 years old"
```

and we get the methods that are exclusive to our Engineer class that are not part of Person:

```javascript
william.departmentBelonging() 
// returns "William is in the engineering department"

william.favoriteLanguage() 
// returns "William favorite language is C++"
```

## static class methods
This is something that still trips me but that will be necessary when working with Polymer elements later on. 

According to MDN:

> Static methods are called without instantiating their class and are also not callable when the class is instantiated. Static methods are often used to create utility functions for an application.
> MDN Javascript Reference: [static keyword](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static#Calling_static_methods)

In order to call a static method within another static method of the same class, you can use the this keyword.

```javascript
class StaticMethodCall {
  static staticMethod() {
    return 'Static method has been called';
  }

  static anotherStaticMethod() {
    // Note the use of this to reference staticMethod()
    return this.staticMethod() + ' from another static method';
  }
}

StaticMethodCall.staticMethod();
// 'Static method has been called'

StaticMethodCall.anotherStaticMethod();
// 'Static method has been called from another static method'
```

Static methods are not directly accessible using just the this keyword. You need to call them by using the class name in front: CLASSNAME.STATIC\_METHOD_NAME (like other calls from outside of class) or by using the constructor property:  `this.constructor.STATIC\_METHOD_NAME`.

```javascript
class StaticMethodCall{
  constructor(){
    console.log(StaticMethodCall.staticMethod());
    // 'static method has been called' 

    console.log(this.constructor.staticMethod());
    // 'static method has been called' 
  }

  static  staticMethod(){
    return 'static method has been called.';
  }
}
```

### Static Method Example

In this example we create three classes:

* Triple is our base class. It defines a static `triple` method
* BiggerTriple extends Triple and defines its own version of the `triple` method that returns a different value than 
the parent class
* TP is defined as a literal object

```javascript
class Triple {
  static triple(n) {
    if (n === undefined) {
      n = 1;
    }
    return n * 3;
  }
}

class BiggerTriple extends Triple {
  static triple(n) {
    return super.triple(n) * super.triple(n);
  }
}
```

The first example calls `Triple.triple` without a value wich causes it to default to one. The function will return 3  as the value

```javascript
console.log(Triple.triple());        // 3
```

The second example calls `Triple.tripe` with a value which will be multiplied by 3 and return 18.

```javascript
console.log(Triple.triple(6));       // 18
```

This example calls `BiggerTriple.triple` which is a derived class and uses its own definition of the `triple` method.

```javascript
console.log(BiggerTriple.triple(3)); // 81
```

We next define a new instance of the Triple class

```javascript
var tp = new Triple();
```

`BiggerTripple.triple` still returns a value using its own definition of the triple method that uses the static methods from the parent class. 

```javascript
console.log(BiggerTriple.triple(3)); 
// 81 (not affected by parent's instantiation)
```

However `tp.triple` will fail because the class has not been defined, only instantiated. 

```javascript
console.log(tp.triple());
// 'tp.triple is not a function'.
```

If we wanted this to work we would have to instantiate the `tp` class  and create its own version of `triple`.  Something like this:

```javascript
class TP extends Triple {
  static triple(n) {
    return super.triple(n);
  }
}
```


## Class expressions

Just like with functions we can use literal expressions to create class definitions. We can use unnamed and 
named declarations. The advantage of using named classes when creating them is that we can reuse them in other parts 
of our code. 
    
```javascript
// unnamed
let Polygon = class {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
};

// named
let Polygon2 = class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
};
```
These will become important when we start building mixins in the next section. We'll use anonymous class expressions 
to create the mixin functions before we "mix" them with our classes.


## Mixins

Mixins are another name for abstract classes we use as templates for subclasses.  We use mixins to get around the 
fact that ECMAScript can have only one superclass. This makes it impossible to create smaller reusable classes to add
 to our toolset. By creating mixins we can create these smaller "library" classes and we can add them to our tools.
 
Any function with a superclass as input and a subclass extending that superclass as output can be used to implement 
mix-ins in ECMAScript. Below are some examples:

```javascript
var engineerrMixin = Person => class extends Person {
  affiliation() { }
  
  language() { }
};

var programmerMixin = Person => class extends Person {
  editor() { }
  
  platofrm() { }
};
```

A class that uses these mix-ins can then be written like this: 

```javascript
class Person { }
class Emplyee extends engineerrMixin(ProgrammerMixin(Person)) { }
```

The code is a little hard to understand, particularly if you're not familiar with how to extend mix in multiple files
 at the same time.  What is says is that class `Employee` uses both `engineerrMixin` and `programmerMixing` with 
 `Person` as the base class.  Whenever using Bar we have access to the methods of the base class `Foo` the methods in
  both mixin classes (`engineerMixin` and `programmerMixin`) and any methods exclusive to `Bar`. 

A fully realized `engineerMixin` could look like this:

```javascript
var engineerMixin = Person => class extends Person {
  departmentBelonging() {
       return `${this.first} is in the ${this.department} department`;
  }
  
  favoriteLanguage() {
    return `${this.first} favorite language is ${this.language}`;
  }
};
```

And then we can use the mixin by using when defining another subclass of person. We get all the methods in the Person
 base class, all the methods of the engineer mixin and whatever methods we add to the Employee subclass. 

```javasacript
class Employee extends engineerMixin(Person)) {
  // Define methods of exmployee here
}
```

### Mixin Example



## Notes, caveats and potential pitfalls 

As I researched this post I found several gotchas to consider when working with classes. Here are some of them

### Classes do not hoist

When working with functions is ok to use a functions before it's defined, something like this:

```javascript
drawRectangle();

function drawRectangle() {
  // Definition goes here
};
```

In classes this will return a Reference Error as illustrated in the example below. The class must be defined before 
it's used either in the same file, an external file imported in to the class or in the same inline script.  

```javascript
var p = new Polygon(); 
// This will produce: ReferenceError: Polygon is not defined

class Polygon {}
```

If we define the class first then the definition will work regardless of where it is. 

```javascript
class Polygon {}

// This will work as intended
var p = new Polygon(); 
```

### Always runs on strict mode

The bodies of class declarations and class expressions are executed in strict mode. This may introduce additional 
complications if you're not familiar with strict mode in ES5. 

Mozilla provides a good [definition of strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) and a [transition guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode/Transitioning_to_strict_mode#this_in_function_calls) from non-strict to strict mode

# Custom Elements v1

There's a new version of custom elements going through the standards process, it is known as V1 to distinguish it 
from the original version implemented in Chrome and familiar if you've used Polymer 1.x. before. Custom Elements and 
Shadow DOM V1 are the specs making it to browsers so it pays to learn it now and get up to speed before the specs and
 the libraries supporting them upgrade.  

## The basics

```javascript
class DemoDiv extends HTMLElement {
  // TODO: complete definition of DemoDiv
}

// Defines the demo-div element and 
// associates it with the DemoDiv class
window.customElements.define('demo-div', DemoDiv);
```

We can be more specific in what element definitions we extend. The [full list of HTML element interfaces](https://html.spec.whatwg.org/multipage/indices.html#element-interfaces) is part of the HTML spec maintained by WHATWG


You can use this element as is. Call it using the tag name we defined for it:

```html
<demo-div></demo-div>
```

## Defining the element Javascript API

```javascript
class DemoDiv extends HTMLElement {

  // A getter/setter for a disabled property.
  get disabled() {
    return this.hasAttribute('disabled');
  }

  set disabled(val) {
    // Reflect the value of the disabled property as an HTML attribute.
    if (val) {
      this.setAttribute('disabled', '');
    } else {
      this.removeAttribute('disabled');
    }
  }

  // Can define constructor arguments if you wish.
  constructor() {
    // If you define a ctor, always call super() first!
    // This is specific to CE and required by the spec.
    super();

    // Setup a click listener on <demo-div> itself.
    this.addEventListener('click', e => {
      // Don't toggle the drawer if it's disabled.
      if (this.disabled) {
        return;
      }
      this.toggleDisplay();
    });
  }
  
  toggleDisplay() {
    console.log('toggled');
  }
}

// Defines the demo-div element and 
// associates it with the DemoDiv class
window.customElements.define('demo-div', DemoDiv);
```

### Reaction Callbacks


<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Called When</th>
        </tr>
    <thead>
    <tbody>
        <tr>
            <td>constructor</td>
            <td>
                <p>An instance of the element is created or <a href="https://developers.google.com/web/fundamentals/getting-started/primers/customelements?hl=en#upgrades">upgraded</a>. Useful for 
                initializing state, settings up event listeners, or <a href="https://developers.google
                .com/web/fundamentals/getting-started/primers/customelements?hl=en#shadowdom">creating shadow dom</a>. See the <a href="https://html.spec.whatwg.org/multipage/scripting.html#custom-element-conformance">spec</a> for restrictions on what you can do in the constructor.</p></td>
        </tr>
        <tr>
            <td>connectedCallback</td>
            <td>
                <p>Called every time the element is inserted into the DOM. Useful for running setup 
            code, such as fetching resources or rendering. Generally, you should try to delay work until this time
            .</p></td>
        </tr>
        <tr>
            <td>disconnectedCallback</td>
            <td>
                <p>Called every time the element is removed from the DOM. Useful for running 
            clean up  code (removing event listeners, etc.).</p></td>
        <tr>
            <td>attributeChangedCallback(attrName, oldVal, newVal)</td>
            <td>
                <p>An attribute was added, removed, updated, or replaced.</p>
                <p></p>
                <p>Also called for initial values when an element is created by the parser, or <a href="https://developers.google
                                                                                                               .com/web/fundamentals/getting-started/primers/customelements?hl=en#upgrades">upgraded</a>. <strong>Note</strong>: only attributes listed in the <strong>observedAttributes</strong> property will receive this callback.</p>
            </td>
        </tr>
        <tr>
            <td>adoptedCallback()</td>
            <td>
                <p>The custom element has been moved into a new document (e.g. someone called document.adoptNode(el)).</p>
            </td>
        </tr>
    </tbody>
</table>

The browser calls the attributeChangedCallback() for any attributes whitelisted in the observedAttributes array (see Observing changes to attributes). Essentially, this is a performance optimization. When users change a common attribute like style or class, you don't want to be spammed with tons of callbacks.

Reaction callbacks are synchronous. If someone calls el.setAttribute(...) on your element, the browser will immediately call attributeChangedCallback(). Similarly, you'll receive a disconnectedCallback() right after your element is removed from the DOM (e.g. the user calls el.remove()).

## Element Attribute Manipulation



### Reflecting properties to attributes

Whenever we set a value for an attribute of an HTML tag it is reflected back to the DOM representation of our 
element. The most common I've seen is when assigning a `src` attribute to an image tag. In the example below we'll 
assign an `id` and `src` attributes to an image tag

For example, when the values of hidden or id are changed in JS:

```javascript
img.id = 'image001';
img.src = 'images/image001.png';
```

the values are applied to the live DOM as attributes:

```html
<img id="image001" src="images/image001.png">
```

This is called "reflecting properties to attributes". Almost every property in HTML does this. Why? Attributes are also useful for configuring an element declaratively and certain APIs like accessibility and CSS selectors rely on attributes to work.

Reflecting a property is useful anywhere you want to keep the element's DOM representation in sync with its JavaScript state. One reason you might want to reflect a property is so user-defined styling applies when JS state changes.

Recall our `<demo-div>`. A consumer of this component may want to fade it out and/or prevent user interaction when 
it's disabled:

```css
demo-div[disabled] {
  opacity: 0.5;
  pointer-events: none;
}
```

When the disabled property is changed in JS, we want that attribute to be added to the DOM so the user's selector matches. The element can provide that behavior by reflecting the value to an attribute of the same name:

```javascript
get disabled() {
  return this.hasAttribute('disabled');
}

set disabled(val) {
  // Reflect the value of `disabled` as an attribute.
  if (val) {
    this.setAttribute('disabled', '');
  } else {
    this.removeAttribute('disabled');
  }
  this.toggleDisplay();
}
```

Using this setter/getter pair we can change and retrieve the value of the disabled attribute. We'll discuss why this 
is important when we talk about observing changes to attributes. 
 
 
### Observing changes to attributes

HTML attributes are a convenient way for users to declare initial state:

```html
<demo-div open disabled></demo-div>
```

Elements can react to attribute changes by defining a attributeChangedCallback. The browser will call this method for
 every change to attributes listed in the observedAttributes array.  

```javascript
class DemoDiv extends HTMLElement {

  static get observedAttributes() {
    return ['disabled'];
  }

  get disabled() {
    return this.hasAttribute('disabled');
  }

  set disabled(val) {
    if (val) {
      this.setAttribute('disabled', '');
    } else {
      this.removeAttribute('disabled');
    }
  }

  // Only called for the disabled and open attributes due to observedAttributes
  attributeChangedCallback(name, oldValue, newValue) {
    // When the drawer is disabled, update keyboard/screen reader behavior.
    if (this.disabled) {
      this.setAttribute('tabindex', '-1');
      this.setAttribute('aria-disabled', 'true');
    } else {
      this.setAttribute('tabindex', '0');
      this.setAttribute('aria-disabled', 'false');
    }
  }
}
```

In the example, we're setting additional attributes on `<demo-div>` when a disabled attribute is changed. We also 
change accessibility attributes to make sure it works as intended for assistive technology. Depending on your 
application you may want to add additional attributes using the same formula shown for `tab-index` and 
`aria-disabled`.  

## User content

As good as custom elements are are not enough to convey both structure and content of our elements and applications. 
We have three ways to add content to our custom elements:
  
  * Child content
  * Content created in the Shadow DOM
  * Content created in templates
  
We'll explore each of these  in turn. 

### Child Content

Custom elements can manage their own content by using the DOM APIs inside element code. Reactions come in handy for 
this. In this example we add HTML text to the element when it's added to the DOM

```javascript
customElements.define('x-foo-with-markup', class extends HTMLElement {
  connectedCallback() {
    this.innerHTML = "<strong>I'm an x-foo-with-markup!</strong>";
  }
});
```

This declaration will produce will produce:

```html
<x-foo-with-markup>
 <strong>I'm an x-foo-with-markup!</strong>
</x-foo-with-markup>
```

### Shadow DOM

Shadow DOM provides a way for an element (custom or not)to own, render, and style a chunk of DOM that's separate from 
the rest of 
the page. 

If we wanted to, we could hide an entire app within a single tag:

```html
<!-- chat-app's implementation details are hidden away in Shadow DOM. -->
<wordpress-app></wordpress-app>
```

To use Shadow DOM in a custom element, call this.attachShadow inside your constructor:

```javascript
customElements.define('wordpress-app', class extends HTMLElement {
  constructor() {
    super(); // always call super() first in the constructor.

    // Attach a shadow root to the element.
    let shadowRoot = this.attachShadow({mode: 'open'});
    shadowRoot.innerHTML = `
      <style>:host { ... }</style> <!-- look ma, scoped styles -->
      <strong>I'm in shadow dom!</strong>
      <slot></slot>
    `;
  }
});
````

In the example below the p element will be inserted into the slot and that's what will be displayed to the user along
 with whatever styles we've added for the `:host` pseudo element.  

```html
<wordpress-app>
  <p><strong>User's</strong> custom text</p>
</wordpress-app>

<!-- renders as -->
<wordpress-app>
  <strong>I'm in shadow dom!</strong>
  <p><strong>User's</strong> custom text</p>
</wordpress-app>
```

### Template elements

For those unfamiliar, the <template> element allows you to declare fragments of DOM which are parsed, inert at page load, and can be activated later at runtime. It's another API primitive in the web components family. Templates are an ideal placeholder for declaring the structure of a custom element.

Example: registering an element with Shadow DOM content created from a <template>:

```html
<template id="x-foo-from-template">
  <style>
    p { color: orange; }
  </style>
  <p>I'm in Shadow DOM. My markup was stamped from a &lt;template&gt;.</p>
</template>

<script>
  customElements.define('x-foo-from-template', class extends HTMLElement {
    constructor() {
      super(); // always call super() first in the ctor.
      let shadowRoot = this.attachShadow({mode: 'open'});
      const t = document.querySelector('#x-foo-from-template');
      const instance = t.content.cloneNode(true);
      shadowRoot.appendChild(instance);
    }
  });
</script>
```

The script does a lot and it's not intuitive to understand. Let's unpack it and see what it's actually doing:

1. We define a new element in HTML: <x-foo-from-template>
2. The element's Shadow DOM is created from a <template>
3. The element's DOM is local to the element thanks to Shadow DOM
4. The element's internal CSS is scoped to the element thanks to Shadow DOM


## Styling custom elements

Even if your element defines its own styling using Shadow DOM, users can style your custom element from their page. 
These are called "user-defined styles".

```html
<!-- user-defined styling -->
<style>
  demo-div {
    display: flex;
  }
  panel-item {
    transition: opacity 400ms ease-in-out;
    opacity: 0.3;
    flex: 1;
    text-align: center;
    border-radius: 50%;
  }
  panel-item:hover {
    opacity: 1.0;
    background: rgb(255, 0, 255);
    color: white;
  }
</style>

<demo-div>
  <panel-item>Do</panel-item>
  <panel-item>Re</panel-item>
  <panel-item>Mi</panel-item>
</demo-div>
```

You might be asking yourself how CSS specificity works if the element has styles defined within Shadow DOM. In terms of specificity, user styles win. They'll always override element-defined styling. See the section on Creating an element that uses Shadow DOM.

### Pre-styling unregistered elements

Before an element is upgraded you can target it in CSS using the `:defined` pseudo-class. This is useful for 
pre-styling a component. For example, you may wish to prevent layout or other visual FOUC by hiding undefined 
components and fading them in when they become defined. After <demo-div> becomes defined, the selector `demo-div:not(:defined)` no longer matches. 


```css
demo-div:not(:defined) {
  /* Pre-style, give layout, replicate demo-div's eventual styles, etc. */
  display: inline-block;
  height: 100vh;
  opacity: 0;
  transition: opacity 0.3s ease-in-out;
}
```

# Shadow DOM v1

Shadow DOM is one of the four Web Component standards: 

* HTML Templates
* Shadow DOM
* Custom  elements
* HTML Imports

You don't have to author web components that use shadow DOM. But when you do, you take advantage of its benefits (CSS scoping, DOM encapsulation, composition) and build reusable custom elements, which are resilient, highly configurable, and extremely reusable. If custom elements are the way to create a new HTML (with a JS API), shadow DOM is the way you provide its HTML and CSS. The two APIs combine to make a component with self-contained HTML, CSS, and JavaScript.

Shadow DOM is designed as a tool for building component-based apps. Therefore, it brings solutions for common problems in web development:

* **Isolated DOM**: A component's DOM is self-contained (e.g. document.querySelector() won't return nodes in the 
component's shadow DOM).
* **Scoped CSS**: CSS defined inside shadow DOM is scoped to it. Style rules don't leak out and page styles don't bleed in.
* **Composition**: Design a declarative, markup-based API for your component.
* **Simplifies CSS** - Scoped DOM means you can use simple CSS selectors, more generic id/class names, and not worry 
about naming conflicts.
* **Productivity** - Think of apps in chunks of DOM rather than one large (global) page.

## How the browser build the DOM

```javascript
const header = document.createElement('header');
const h1 = document.createElement('h1');
h1.textContent = 'Hello world!';
header.appendChild(h1);
document.body.appendChild(header);
```

produces the following HTML markup:

```html
<body>
  <header>
    <h1>Hello DOM</h1>
  </header>
</body>
```
## Introducing Shadow DOM

Shadow DOM is just normal DOM with two differences: 

1. how it's created/used and 
2. how it behaves in relation to the rest of the page

Normally, you create DOM nodes and append them as children of another element. 

With shadow DOM, you create a scoped DOM tree that's attached to the element, but separate from its actual children.
 This scoped subtree is called a shadow tree. The element it's attached to is its shadow host. Anything you add in the shadows becomes local to the hosting element, including <style>. This is how shadow DOM achieves CSS style scoping.

### Creating shadow DOM

A shadow root is a document fragment that gets attached to a “host” element. The act of attaching a shadow root is how the element gains its shadow DOM. To create shadow DOM for an element, call element.attachShadow():

```javascript
const header = document.createElement('header');
const shadowRoot = header.attachShadow({mode: 'open'});
shadowRoot.innerHTML = '<h1>Hello Shadow DOM</h1>'; // Could also use appendChild().

// header.shadowRoot === shadowRoot
// shadowRoot.host === header
```

I'm using .innerHTML to fill the shadow root, but you could also use other DOM APIs. This is the web. We have choice.

The spec defines a [list of elements that can't host a shadow tree](http://w3c.github.io/webcomponents/spec/shadow/#h-methods). There are several reasons an element might be on 
the list:

* The browser already hosts its own internal shadow DOM for the element (<textarea>, <input>).
* It doesn't make sense for the element to host a shadow DOM (<img>).

For example, this will not work:

```javascript
document.createElement('input').attachShadow({mode: 'open'});
// Error. `<input>` cannot host shadow dom.
Creating shadow DOM for a custom element
```

Shadow DOM is particularly useful when creating custom elements. Use shadow DOM to compartmentalize an element's HTML, CSS, and JS, thus producing a "web component".

```javscript
// Use custom elements API v1 to register a new HTML tag and define its JS behavior
// using an ES6 class. Every instance of <fancy-tab> will have this same prototype.
customElements.define('fancy-tabs', class extends HTMLElement {
  constructor() {
    super(); // always call super() first in the ctor.

    // Attach a shadow root to <fancy-tabs>.
    const shadowRoot = this.attachShadow({mode: 'open'});
    shadowRoot.innerHTML = `
      <style>#tabs { ... }</style> <!-- styles are scoped to fancy-tabs! -->
      <div id="tabs">...</div>
      <div id="panels">...</div>
    `;
  }
});
```
There are a couple of interesting things going on here. The first is that the custom element creates its own shadow DOM when an instance of <fancy-tabs> is created. That's done in the constructor(). Secondly, because we're creating a shadow root, the CSS rules inside the <style> will be scoped to <fancy-tabs>.

## Composition and slots

Composition is one of the least understood features of shadow DOM, but it's arguably the most important.

In our world of web development, composition is how we construct apps, declaratively out of HTML. Different building blocks (<div>s, <header>s, <form>s, <input>s) come together to form apps. Some of these tags even work with each other. Composition is why native elements like <select>, <details>, <form>, and <video> are so flexible. Each of those tags accepts certain HTML as children and does something special with them. For example, <select> knows how to render <option> and <optgroup> into dropdown and multi-select widgets. The <details> element renders <summary> as a expandable arrow. Even <video> knows how to deal with certain children: <source> elements don't get rendered, but they do affect the video's behavior. What magic!

### Terminology: light DOM vs. shadow DOM

Shadow DOM composition introduces a bunch of new fundamentals in web development. Before getting into the weeds, let's standardize on some terminology so we're speaking the same lingo.

#### Light DOM

The markup a user of your component writes. This DOM lives outside the component's shadow DOM. It is the element's actual children.

```javascript
<button is="better-button">
  <!-- the image and span are better-button's light DOM -->
  <img src="gear.svg" slot="icon">
  <span>Settings</span>
</button>
```

#### Shadow DOM

The DOM a component author writes. Shadow DOM is local to the component and defines its internal structure, scoped CSS, and encapsulates your implementation details. It can also define how to render markup that's authored by the consumer of your component.

```css
#shadow-root
  <style>...</style>
  <slot name="icon"></slot>
  <span id="wrapper">
    <slot>Button</slot>
  </span>
```

#### Flattened DOM tree

The result of the browser distributing the user's light DOM into your shadow DOM, rendering the final product. The flattened tree is what you ultimately see in the DevTools and what's rendered on the page.

```html
<button is="better-button">
  #shadow-root
    <style>...</style>
    <slot name="icon">
      <img src="gear.svg" slot="icon">
    </slot>
    <slot>
      <span>Settings</span>
    </slot>
</button>
```

#### The <slot> element

Shadow DOM composes different DOM trees together using the <slot> element. Slots are placeholders inside your component that users can fill with their own markup. By defining one or more slots, you invite outside markup to render in your component's shadow DOM. Essentially, you're saying "Render the user's markup over here".

Note: Slots are a way of creating a "declarative API" for a web component. They mix-in the user's DOM to help render the overall component, thus, composing different DOM trees together.
Elements are allowed to "cross" the shadow DOM boundary when a <slot> invites them in. These elements are called distributed nodes. Conceptually, distributed nodes can seem a bit bizarre. Slots don't physically move DOM; they render it at another location inside the shadow DOM.

A component can define zero or more slots in its shadow DOM. Slots can be empty or provide fallback content. If the user doesn't provide light DOM content, the slot renders its fallback content.

```html
<!-- Default slot. If there's more than one default slot, the first is used. -->
<slot></slot>
```

```html
<slot>Fancy button</slot> <!-- default slot with fallback content -->
```

```html
<slot> <!-- default slot entire DOM tree as fallback -->
  <h2>Title</h2>
  <summary>Description text</summary>
</slot>
```

You can also create named slots. Named slots are specific holes in your shadow DOM that users reference by name.

Example - the named slots in <fancy-tabs>'s shadow DOM:

```html
#shadow-root
  <div id="tabs">
    <slot id="tabsSlot" name="title"></slot>
  </div>
  <div id="panels">
    <slot id="panelsSlot"></slot>
  </div>
```

Component users declare <fancy-tabs> like so:

```html
<fancy-tabs>
  <button slot="title">Title</button>
  <button slot="title" selected>Title 2</button>
  <button slot="title">Title 3</button>
  <section>content panel 1</section>
  <section>content panel 2</section>
  <section>content panel 3</section>
</fancy-tabs>

<!-- Using <h2>'s and changing the ordering would also work! -->
<fancy-tabs>
  <h2 slot="title">Title</h2>
  <section>content panel 1</section>
  <h2 slot="title" selected>Title 2</h2>
  <section>content panel 2</section>
  <h2 slot="title">Title 3</h2>
  <section>content panel 3</section>
</fancy-tabs>
```html


And if you're wondering, the flattened tree looks something like this:

```html
<fancy-tabs>
  #shadow-root
    <div id="tabs">
      <slot id="tabsSlot" name="title">
        <button slot="title">Title</button>
        <button slot="title" selected>Title 2</button>
        <button slot="title">Title 3</button>
      </slot>
    </div>
    <div id="panels">
      <slot id="panelsSlot">
        <section>content panel 1</section>
        <section>content panel 2</section>
        <section>content panel 3</section>
      </slot>
    </div>
</fancy-tabs>
```html

Notice our component is able to handle different configurations, but the flattened DOM tree remains the same. We can also switch from <button> to <h2>. This component was authored to handle different types of children...just like <select> does!

## Styling

There are many options for styling web components. A component that uses shadow DOM can be styled by the main page, define its own styles, or provide hooks (in the form of CSS custom properties) for users to override defaults.

### Component-defined styles

Hands down the most useful feature of shadow DOM is scoped CSS:

* CSS selectors from the outer page don't apply inside your component.
* Styles defined inside don't bleed out. They're scoped to the host element.
* CSS selectors used inside shadow DOM apply locally to your component. In practice, this means we can use common 
id/class names again, without worrying about conflicts elsewhere on the page. Simpler CSS selectors are a best practice inside Shadow DOM. They're also good for performance.

Example - styles defined in a shadow root are local

```html
#shadow-root
  <style>
    #panels {
      box-shadow: 0 2px 2px rgba(0, 0, 0, .3);
      background: white;
      ...
    }
    #tabs {
      display: inline-flex;
      ...
    }
  </style>
  <div id="tabs">
    ...
  </div>
  <div id="panels">
    ...
  </div>
```

Stylesheets are also scoped to the shadow tree:

```html
#shadow-root
  <!-- Available in Chrome 54+ -->
  <!-- WebKit bug: https://bugs.webkit.org/show_bug.cgi?id=160683 -->
  <link rel="stylesehet" href="styles.css">
  <div id="tabs">
    ...
  </div>
  <div id="panels">
    ...
  </div>
```html


Example - a component styling itself

```html
<style>
:host {
  display: block; /* by default, custom elements are display: inline */
  contain: content; /* CSS containment FTW. */
}
</style>
```html

One gotcha with :host is that rules in the parent page have higher specificity than :host rules defined in the element. That is, outside styles win. This allows users to override your top-level styling from the outside. Also, :host only works in the context of a shadow root, so you can't use it outside of shadow DOM.

The functional form of :host(<selector>) allows you to target the host if it matches a <selector>. This is a great way for your component to encapsulate behaviors that react to user interaction or state or style internal nodes based on the host.

```html
<style>
:host {
  opacity: 0.4;
  will-change: opacity;
  transition: opacity 300ms ease-in-out;
}
:host(:hover) {
  opacity: 1;
}
:host([disabled]) { /* style when host has disabled attribute. */
  background: grey;
  pointer-events: none;
  opacity: 0.4;
}
:host(.blue) {
  color: blue; /* color host when it has class="blue" */
}
:host(.pink) > #tabs {
  color: pink; /* color internal #tabs node when host has class="pink".
}
</style>
```html
### Styling based on context

:host-context(<selector>) matches the component if it or any of its ancestors matches <selector>. A common use for this is theming based on a component's surroundings. For example, many people do theming by applying a class to <html> or <body>:

```html
<body class="darktheme">
  <fancy-tabs>
    ...
  </fancy-tabs>
</body>
```html

`:host-context(.darktheme)` would style `<fancy-tabs>` when it's a descendant of .darktheme:

:host-context(.darktheme) {
  color: white;
  background: black;
}
:host-context() can be useful for theming, but an even better approach is to create style hooks using CSS custom properties.

### Styling distributed nodes

`::slotted(<compound-selector>)` matches nodes that are distributed into a <slot>.

Let's say we've created a name badge component:

```html
<name-badge>
  <h2>Eric Bidelman</h2>
  <span class="title">
    Digital Jedi, <span class="company">Google</span>
  </span>
</name-badge>
```html

The component's shadow DOM can style the user's <h2> and .title:

```html
<style>
::slotted(h2) {
  margin: 0;
  font-weight: 300;
  color: red;
}
::slotted(.title) {
   color: orange;
}
/* DOESN'T WORK (can only select top-level nodes).
::slotted(.company),
::slotted(.title .company) {
  text-transform: uppercase;
}
*/
</style>
<slot></slot>
```html

If you remember from before, <slot>s do not move the user's light DOM. When nodes are distributed into a <slot>, the <slot> renders their DOM but the nodes physically stay put. Styles that applied before distribution continue to apply after distribution. However, when the light DOM is distributed, it can take on additional styles (ones defined by the shadow DOM).

### Styling a component from the outside

There are a couple of ways to style a component from the outside. The easiest way is to use the tag name as a selector:

fancy-tabs {
  width: 500px;
  color: red; /* Note: inheritable CSS properties pierce the shadow DOM boundary. */
}
fancy-tabs:hover {
  box-shadow: 0 3px 3px #ccc;
}
Outside styles always win over styles defined in shadow DOM. For example, if the user writes the selector fancy-tabs { width: 500px; }, it will trump the component's rule: :host { width: 650px;}.

Style the component itself will only get you so far. But what happens if you want to style the internals of a component? For that, we need CSS custom properties.

Creating style hooks using CSS custom properties

Users can tweak internal styles if the component's author provides styling hooks using CSS custom properties. Conceptually, the idea is similar to <slot>. You create "style placeholders" for users to override.

Example - <fancy-tabs> allows users to override the background color:

<!-- main page -->
<style>
  fancy-tabs {
    margin-bottom: 32px;
    --fancy-tabs-bg: black;
  }
</style>
<fancy-tabs background>...</fancy-tabs>
Inside its shadow DOM:

:host([background]) {
  background: var(--fancy-tabs-bg, #9E9E9E);
  border-radius: 10px;
  padding: 10px;
}
In this case, the component will use black as the background value since the user provided it. Otherwise, it would default to #9E9E9E.

Note: As the component author, you're responsible for letting developers know about CSS custom properties they can use. Consider it part of your component's public interface. Make sure to document styling hooks!
Advanced topics

Creating closed shadow roots (should avoid)

There's another flavor of shadow DOM is called "closed" mode. When you create a closed shadow tree, outside JavaScript won't be able to access the internal DOM of your component. This is similar to how native elements like <video> work. JavaScript cannot access the shadow DOM of <video> because the browser implements it using a closed-mode shadow root.

Example - creating a closed shadow tree:

const div = document.createElement('div');
const shadowRoot = div.attachShadow({mode: 'closed'}); // close shadow tree
// div.shadowRoot === null
// shadowRoot.host === div
Other APIs are also affected by closed-mode:

Element.assignedSlot / TextNode.assignedSlot returns null
Event.composedPath() for events associated with elements inside the shadow DOM, returns []
Note: Closed shadow roots are not very useful. Some developers will see closed mode as an artificial security feature. But let's be clear, it's not a security feature. Closed mode simply prevents outside JS from drilling into an element's internal DOM.
Here's my summary of why you should never create web components with {mode: 'closed'}:

Artificial sense of security. There's nothing stopping an attacker from hijacking Element.prototype.attachShadow.
Closed mode prevents your custom element code from accessing its own shadow DOM. That's complete fail. Instead, you'll have to stash a reference for later if you want to use things like querySelector(). This completely defeats the original purpose of closed mode!

customElements.define('x-element', class extends HTMLElement {
  constructor() {
    super(); // always call super() first in the ctor.
    this._shadowRoot = this.attachShadow({mode: 'closed'});
    this._shadowRoot.innerHTML = '<div class="wrapper"></div>';
  }
  connectedCallback() {
    // When creating closed shadow trees, you'll need to stash the shadow root
    // for later if you want to use it again. Kinda pointless.
    const wrapper = this._shadowRoot.querySelector('.wrapper');
  }
  ...
});
Closed mode makes your component less flexible for end users. As you build web components, there will come a time when you forget to add a feature. A configuration option. A use case the user wants. A common example is forgetting to include adequate styling hooks for internal nodes. With closed mode, there's no way for users to override defaults and tweak styles. Being able to access the component's internals is super helpful. Ultimately, users will fork your component, find another, or create their own if it doesn't do what they want :(
Working with slots in JS

The shadow DOM API provides utilities for working with slots and distributed nodes. These come in handy when authoring a custom element.

slotchange event

The slotchange event fires when a slot's distributed nodes changes. For example, if the user adds/removes children from the light DOM.

const slot = this.shadowRoot.querySelector('#slot');
slot.addEventListener('slotchange', e => {
  console.log('light dom children changed!');
});
Note:slotchange does not fire when an instance of the component is first initialized.

To monitor other types of changes to light DOM, you can setup a MutationObserver in your element's constructor.

What elements are being rendering in a slot?

Sometimes it's useful to know what elements are associated with a slot. Call slot.assignedNodes() to find which elements the slot is rendering. The {flatten: true} option will also return a slot's fallback content (if no nodes are being distributed).

As an example, let's say your shadow DOM looks like this:

<slot><b>fallback content</b></slot>
Usage	Call	Result
<button is="better-button">My button</button>	slot.assignedNodes();	[text]
<button is="better-button"></button>	slot.assignedNodes();	[]
<button is="better-button"></button>	slot.assignedNodes({flatten: true});	[<b>fallback content</b>]
What slot is an element assigned to?

Answering the reverse question is also possible. element.assignedSlot tells you which of the component slots your element is assigned to.

The Shadow DOM event model

When an event bubbles up from shadow DOM it's target is adjusted to maintain the encapsulation that shadow DOM provides. That is, events are re-targeted to look like they've come from the component rather than internal elements within your shadow DOM. Some events do not even propagate out of shadow DOM.

The events that do cross the shadow boundary are:

Focus Events: blur, focus, focusin, focusout
Mouse Events: click, dblclick, mousedown, mouseenter, mousemove, etc.
Wheel Events: wheel
Input Events: beforeinput, input
Keyboard Events: keydown, keyup
Composition Events: compositionstart, compositionupdate, compositionend
DragEvent: dragstart, drag, dragend, drop, etc.
Tips

If the shadow tree is open, calling event.composedPath() will return an array of nodes that the event traveled through.

Using custom events

Custom DOM events which are fired on internal nodes in a shadow tree do not bubble out of the shadow boundary unless the event is created using the composed: true flag:

// Inside <fancy-tab> custom element class definition:
selectTab() {
  const tabs = this.shadowRoot.querySelector('#tabs');
  tabs.dispatchEvent(new Event('tab-select', {bubbles: true, composed: true}));
}
If composed: false (default), consumers won't be able to listen for the event outside of your shadow root.

<fancy-tabs></fancy-tabs>
<script>
  const tabs = document.querySelector('fancy-tabs');
  tabs.addEventListener('tab-select', e => {
    // won't fire if `tab-select` wasn't created with `composed: true`.
  });
</script>
Handling focus

If you recall from shadow DOM's event model, events that are fired inside shadow DOM are adjusted to look like they come from the hosting element. For example, let's say you click an <input> inside a shadow root:

<x-focus>
  #shadow-root
    <input type="text" placeholder="Input inside shadow dom">
The focus event will look like it came from <x-focus>, not the <input>. Similarly, document.activeElement will be <x-focus>. If the shadow root was created with mode:'open' (see closed mode), you'll also be able access the internal node that gained focus:

document.activeElement.shadowRoot.activeElement // only works with open mode.
If there are multiple levels of shadow DOM at play (say a custom element within another custom element), you need to recursively drill into the shadow roots to find the activeElement:

function deepActiveElement() {
  let a = document.activeElement;
  while (a && a.shadowRoot && a.shadowRoot.activeElement) {
    a = a.shadowRoot.activeElement;
  }
  return a;
}
Another option for focus is the delegatesFocus: true option, which expands the focus behavior of element's within a shadow tree:

If you click a node inside shadow DOM and the node is not a focusable area, the first focusable area becomes focused.
When a node inside shadow DOM gains focus, :focus applies to the host in addition to the focused element.
Example - how delegatesFocus: true changes focus behavior

<style>
  :focus {
    outline: 2px solid red;
  }
</style>

<x-focus></x-focus>

<script>
customElements.define('x-focus', class extends HTMLElement {
  constructor() {
    super(); // always call super() first in the ctor.

    const root = this.attachShadow({mode: 'open', delegatesFocus: true});
    root.innerHTML = `
      <style>
        :host {
          display: flex;
          border: 1px dotted black;
          padding: 16px;
        }
        :focus {
          outline: 2px solid blue;
        }
      </style>
      <div>Clickable Shadow DOM text</div>
      <input type="text" placeholder="Input inside shadow dom">`;

    // Know the focused element inside shadow DOM:
    this.addEventListener('focus', function(e) {
      console.log('Active element (inside shadow dom):',
                  this.shadowRoot.activeElement);
    });
  }
});
</script>
Result



Above is the result when <x-focus> is focused (user click, tabbed into, focus(), etc.), "Clickable Shadow DOM text" is clicked, or the internal <input> is focused (including autofocus).

If you were to set delegatesFocus: false, here's what you would see instead:


delegateFocus: false and the internal <input> is focused.

delegateFocus: false and <x-focus> gains focus (e.g. it has tabindex="0").

delegateFocus: false and "Clickable Shadow DOM text" is clicked (or other empty area within the element's shadow DOM is clicked).
Tips & Tricks

Over the years I've learned a thing or two about authoring web components. I think you'll find some of these tips useful for authoring components and debugging shadow DOM.

Use CSS containment

Typically, a web component's layout/style/paint is fairly self-contained. Use CSS containment in :host for a perf win:

<style>
:host {
  display: block;
  contain: content; /* Boom. CSS containment FTW. */
}
</style>
Resetting inheritable styles

Inheritable styles (background, color, font, line-height, etc.) continue to inherit in shadow DOM. That is, they pierce the shadow DOM boundary by default. If you want to start with a fresh slate, use all: initial; to reset inheritable styles to their initial value when they cross the shadow boundary.

<style>
  div {
    padding: 10px;
    background: red;
    font-size: 25px;
    text-transform: uppercase;
    color: white;
  }
</style>

<div>
  <p>I'm outside the element (big/white)</p>
  <my-element>Light DOM content is also affected.</my-element>
  <p>I'm outside the element (big/white)</p>
</div>

<script>
const el = document.querySelector('my-element');
el.attachShadow({mode: 'open'}).innerHTML = `
  <style>
    :host {
      all: initial; /* 1st rule so subsequent properties are reset. */
      display: block;
      background: white;
    }
  </style>
  <p>my-element: all CSS properties are reset to their
     initial value using <code>all: initial</code>.</p>
  <slot></slot>
`;
</script>

Finding all the custom elements used by a page

Sometimes it's useful to find custom elements used on the page. To do so, you need to recursively traverse the shadow DOM of all elements used on the page.

const allCustomElements = [];

function isCustomElement(el) {
  const isAttr = el.getAttribute('is');
  // Check for <super-button> and <button is="super-button">.
  return el.localName.includes('-') || isAttr && isAttr.includes('-');
}

function findAllCustomElements(nodes) {
  for (let i = 0, el; el = nodes[i]; ++i) {
    if (isCustomElement(el)) {
      allCustomElements.push(el);
    }
    // If the element has shadow DOM, dig deeper.
    if (el.shadowRoot) {
      findAllCustomElements(el.shadowRoot.querySelectorAll('*'));
    }
  }
}

findAllCustomElements(document.querySelectorAll('*'));

Creating elements from a <template>

Instead of populating a shadow root using .innerHTML, we can use a declarative <template>. Templates are an ideal placeholder for declaring the structure of a web component.

#Polymer 2.0


# Polyfills and testing for support

This set of technologies requires ES2015/ES6 and fairly modern browsers.  The level of support will depend on the technology. 

For custom elements the support list is:
* Chrome 54 ([status](https://www.chromestatus.com/features/4696261944934400)) 
* Safari has [begun prototyping](https://bugs.webkit.org/show_bug.cgi?id=150225). You can test the API in WebKit nightly
* Edge has [begun prototyping](https://twitter.com/AaronGustafson/status/717028669948977153)
* Mozilla has an [open bug](#) to implement.

There is a [polyfill](https://github.com/webcomponents/custom-elements/blob/master/custom-elements.min.js) available. 

For Shadow DOM the list is:
* Chrome 53 (status)
* Opera 40
* Safari 10 (shipping)
* Edge is under consideration with high priority
* Mozilla has an open bug to implement.

Until browser support is widely available, the [shadydom](https://github.com/webcomponents/shadydom) and [shadycss](https://github.com/webcomponents/shadycss) polyfills give you v1 feature. Shady DOM mimics the DOM scoping of Shadow DOM and shadycss polyfills CSS custom properties and the style scoping the native API provides. 

To use the polyfills we need to import them using [Bower](https://bower.io/) like this
 
```bash
bower install --save webcomponents/custom-elements
bower install --save webcomponents/shadydom webcomponents/shadycss
```
We then create constants to test if Custom Elements and Shadow DOM. 

```javascript
const supportsCustomElementsV1 = 'customElements' in window;
const supportsShadowDOMV1 = !!HTMLElement.prototype.attachShadow;
```
We could create one constant for both specifications, something like this:

```javascript
const supportStyledElements = ('customElements' in window) && 
                              (!!HTMLElement.prototype.attachShadow);
```

But it makes it harder to work when a browser support one specification but not the other, so we’ll stick to the two seprate constants. 

We then define a script loader function that returns a promise that will resolve when the polyfill loads and reject if it doesn’t. 

```javascript
function loadScript(src) {
 return new Promise(function(resolve, reject) {
   const script = document.createElement('script');
   script.async = true;
   script.src = src;
   script.onload = resolve;
   script.onerror = reject;
   document.head.appendChild(script);
 });
}
```

Finally we use our loadScript function to lazy load the polyfills if needed, i.e: for browserss with no native support. 

```javascript
if (!supportsCustomElementsV1) {
  loadScript('/bower_components/custom-elements/custom-elements.min.js')
  .then(e => {
    console.log('Custom ELements Polyfill loaded successfully');
  });
} else {
  console.log('Native support for custom elements. Good to go');
}

// Lazy load the polyfill if necessary.
if (!supportsShadowDOMV1) {
  loadScript('/bower_components/shadydom/shadydom.min.js')
    .then(e => loadScript('/bower_components/shadycss/shadycss.min.js'))
    .then(e => {
      console.log('Custom ELements Polyfill loaded successfully');
    });
} else {
  console.log('Native support for shadown DOM. Good to go');
}
```

# DOCS FOR DEVELOPMENT
* ES6 Classes, how they work and how we extend them
  * Prerequisite for any work in Polymer 2.0
  * Links and resources
    1. [ES6 Classes in depth](https://hacks.mozilla.org/2015/07/es6-in-depth-classes/) MDN
    2. [ES6 Classes in depth](https://ponyfoo.com/articles/es6-classes-in-depth) Ponyfoo
    2. ["Real" Mixins with JavaScript Classes](http://justinfagnani.com/2015/12/21/real-mixins-with-javascript-classes/) Justin Fagnani
    1. [Custom Elements v1: Reusable Web Components](https://developers.google.com/web/fundamentals/getting-started/primers/customelements?hl=en)
    2. [Shadow DOM v1: Self-Contained Web Components](https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom)
    3. [\<card-swiper\> custom element](https://gist.github.com/ebidel/36fb1bd8cc53c89243ed1e53f2da2eaa)
    4. [Fancy tabs web component - shadow dom v1, custom elements v1, full a11y](https://gist.github.com/ebidel/2d2bb0cdec3f2a16cf519dbaa791ce1b)
    5. [Custom Elements Specification](https://www.w3.org/TR/custom-elements/)
      * [Autonomous example](https://www.w3.org/TR/custom-elements/#custom-elements-autonomous-example)
      * [Customized builtin example](https://www.w3.org/TR/custom-elements/#custom-elements-customized-builtin-example)
      * [Drawbacks of autonomous custom elements](https://www.w3.org/TR/custom-elements/#custom-elements-autonomous-drawbacks)
    6. [Shadow DOM Specification](https://www.w3.org/TR/shadow-dom/) **Note that this specification has part that are being upstreamed to other specs (DOM and HTML). As such its usefulness as a research and learning tool is limited**
      * [Shadow DOM examples for v0 and v1](http://hayato.io/2016/shadowdomv1/)
      * [WebKit announcement on Slot-based ShadowDOM](https://webkit.org/blog/4096/introducing-shadow-dom-api/)



