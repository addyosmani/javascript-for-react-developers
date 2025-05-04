# Vanilla JavaScript for React Developers

This book will guide you through re-implementing common React patterns and features using nothing but **Vanilla JS**, helping you deepen your understanding of the web platform and become a more versatile developer.

*Note: This book was formatted using an LLM*

## Introduction

If you're comfortable building apps with React and its ecosystem but feel less confident working with raw JavaScript and browser APIs, this book is for you. We assume you know the basics of web development (HTML/CSS/JS) and have built React apps, but you might not have created complex apps without a framework. We'll bridge that gap by showing how React's abstractions map to plain JavaScript techniques.

You'll learn how to create dynamic interfaces, manage state, compose components, handle routing, and perform side effects using **just the platform** (no frameworks). We'll review JavaScript fundamentals (like closures and the event loop) to firm up your understanding. Then, we'll dive into building real projects – from a simple to-do list to more advanced apps like a movie explorer and beyond – all in Vanilla JS. Along the way, we'll emphasize a **no-build toolchain** where possible (just use a browser and editor), and then show how to prepare for production with bundlers (using Vite) and even a touch of Node.js for backend concepts.

Each chapter addresses a specific aspect of app development. Chapters include explanations, code examples, and comparisons to how you might do the same in React. We've included several hands-on tutorials/projects to practice the concepts. You can type the code yourself, or if you have an AI coding assistant, you'll find optional prompts to help generate or validate solutions. *(For example, after reading a project chapter, you might ask your AI assistant to "build a to-do app in vanilla JavaScript" to compare with your code.)* Use these AI prompts as a learning tool – but we encourage you to work through the code manually first for maximum learning.

Let's get started on our journey from React to **real** Reactivity – the kind that doesn't require a library at all!

## Chapter 1. JavaScript Fundamentals for React Developers

Before we build anything, let's review some core JavaScript concepts. As a React developer, you've certainly used JavaScript daily, but some fundamentals might have been abstracted away by the framework. Understanding these will make it easier to implement React-like patterns in plain JS.

### 1.1 Closures and Scope

JavaScript functions are **first-class citizens** and form closures. A **closure** is created whenever a function retains access to its lexical scope even after that outer function has finished executing. In practical terms, this means an inner function can remember and use variables from an outer function even after the outer function returns. For example:

```js
function createCounter() {
  let count = 0;
  return function increment() {
    count++;
    console.log(count);
  };
}

const counter = createCounter();
counter(); // 1
counter(); // 2
```

Here, `createCounter` returns an `increment` function. The returned function "closes over" the `count` variable. Each time you call `counter()`, it remembers the last value of `count` because of the closure. Closures are useful for **encapsulating state and creating private variables** in Vanilla JS, similar to how React components might hold state via hooks. In fact, hooks like `useState` leverage closures under the hood to retain state between renders.

### 1.2 The Event Loop and Async JavaScript

In React, you often use effects or callbacks that rely on JavaScript's asynchronous nature (like fetching data or debouncing inputs). Understanding the JavaScript **event loop** will clarify how tasks are handled without blocking the UI.

JavaScript has a **single-threaded** concurrency model based on an event loop. The runtime maintains a **call stack** (for function execution) and a **message queue** (for events and async callbacks). As an example:

- When you call functions, each call is pushed onto the stack, and popped off when it returns.
- Asynchronous tasks (like `setTimeout` callbacks, promises, or DOM events) are placed in a queue (or *microtask* queue for promises) to be processed later.
- The event loop continuously checks the queue and moves ready tasks to the stack **when the stack is free** ([Concurrency model and Event Loop - JavaScript | MDN](https://lia.disi.unibo.it/materiale/JS/developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop.html#:~:text=Queue)). In other words, *"when the stack is empty, a message (task) is taken out of the queue and processed by calling its associated function, until the stack is empty again"* ([Concurrency model and Event Loop - JavaScript | MDN](https://lia.disi.unibo.it/materiale/JS/developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop.html#:~:text=Queue)).

This design ensures the UI isn't blocked while waiting for long operations – the browser can handle other work (like rendering or user input) while JavaScript awaits events. For instance, when you do a `fetch().then(...)` in Vanilla JS, the `.then` callback goes to the queue and runs only after the current call stack is clear. Knowing this helps you avoid pitfalls like updating DOM outside of an event or race conditions with state.

### 1.3 Prototypes and Objects

React uses class components (less nowadays) and plain objects for props/state. In Vanilla JS, understanding prototypes is key for working with objects and even for creating classes.

**Prototypal Inheritance:** JavaScript uses prototypes for inheritance. Every object has an internal link to a prototype object, from which it can inherit properties. As MDN notes, *"Prototypes are the mechanism by which JavaScript objects inherit features from one another."* ([Object prototypes - Learn web development | MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Advanced_JavaScript_objects/Object_prototypes#:~:text=Prototypes%20are%20the%20mechanism%20by,an%20object%20can%20be%20set)). When you access a property on an object, the JS engine will first look at the object itself, and if not found, it will climb up the prototype chain to see if the property exists on the prototype, then the prototype's prototype, and so on.

For example, if you create an array `let arr = [1,2,3]`, `arr` automatically inherits methods like `push` and `pop` from `Array.prototype`. This is why `arr.push(4)` works even though you didn't define `push` on `arr` – it's inherited.

You can leverage prototypes directly (using `Object.create` or setting `MyConstructor.prototype`) or use the newer class syntax which under the hood uses prototypes. For instance:

```js
class Car {
  constructor(model) {
    this.model = model;
  }
  drive() {
    console.log(`${this.model} is driving`);
  }
}
const c = new Car('Tesla');
c.drive(); // "Tesla is driving"
```

Here `Car.prototype.drive` is what provides the `drive` method to instances. Understanding this will help when we create multiple components or want to structure our code with classes in later chapters. It's also useful when reading framework code – many libraries use prototypal patterns under the hood.

### 1.4 The `this` Keyword

In React's functional components, you may not deal with `this` much (aside from maybe in class components or event handlers). But in Vanilla JS, especially when working with DOM events or object methods, `this` is still important. It refers to the context object on which a function is invoked. For example:

```js
const player = {
  name: 'Alice',
  greet() {
    console.log(`Hello, ${this.name}`);
  }
};
player.greet(); // "Hello, Alice"
```

In `player.greet()`, `this` refers to `player`. However, if you detach the method, e.g. `const hello = player.greet; hello();`, `this` would be `undefined` (or the global object in non-strict mode) because it's called without an owner. We'll see this in DOM event handlers: `this` inside a handler refers to the element that received the event. We'll mostly use modern arrow functions and `bind` where necessary to control `this`, but keep this behavior in mind.

### 1.5 Summary

These fundamentals – **closures**, **event loop**, **prototypes**, and **this** – form the foundation for the patterns we'll implement. If they feel abstract now, don't worry. As we build actual projects, we'll reinforce these concepts in context. Feel free to refer back to this chapter as needed. Next, we'll start building the pieces of a user interface with Vanilla JS, much like how React composes a UI from components.

*(You've completed the foundational crash course! If you want a quick refresher via an AI assistant, you could ask: "Explain JavaScript closures, the event loop, and prototypes in simple terms with examples." But otherwise, onward to building UIs without React!)*

## Chapter 2. Building and Updating the DOM: Views Without React

One of React's core strengths is abstracting the DOM and handling updates efficiently via a virtual DOM. In Vanilla JS, **you are the framework** – you'll create, update, and remove DOM elements directly. This chapter will show you how to build a dynamic UI from scratch and keep it updated when state changes, mimicking what React does with components and JSX.

### 2.1 Creating DOM Elements and Rendering Content

In React, you'd write JSX like `<button>Hello</button>` and it magically appears. In Vanilla JS, you explicitly create a DOM node and attach it. There are two main ways:

- **Using HTML strings** and `innerHTML`/`insertAdjacentHTML`: e.g. `container.innerHTML = "<button>Hello</button>"`.
- **Using DOM APIs**: e.g. `document.createElement('button')` and then setting properties.

Let's try the DOM API approach, as it's safer (avoids parsing HTML that could include scripts, etc.):

```js
// Vanilla JS render example
const appRoot = document.getElementById('app');  // assume <div id="app"></div> in HTML
const helloBtn = document.createElement('button');
helloBtn.textContent = 'Hello';  // equivalent to setting innerText
appRoot.appendChild(helloBtn);
```

Three lines replaced what JSX would do in one, but it's straightforward: we created a `<button>` element and set its text content, then appended it to a container in the DOM. We can also set attributes or classes:

```js
helloBtn.className = 'btn primary';      // add CSS classes
helloBtn.setAttribute('data-value', '42'); // custom attribute example
```

For more complex structures, creating many elements by hand can be verbose. Alternatives include:
- Building a string of HTML and using `container.innerHTML = ...` (just be cautious about user-generated content to avoid injection issues).
- Using the `<template>` tag in HTML to define HTML fragments and cloning them in JS.
- Using libraries like lit-html or a minimal templating utility (though that drifts away from pure Vanilla JS – our focus is to do without additional libraries).

We'll explore a simple templating approach in projects, but for now remember: **in Vanilla JS, you control the DOM directly**.

### 2.2 Updating the DOM Efficiently

React's virtual DOM diffing gives the illusion that you can re-render the whole UI on each state change. With Vanilla JS, you *could* do something similar (e.g. clear a list and rebuild it on update), but you can also optimize by updating only the specific parts that changed.

For example, imagine a simple counter displayed in a `<span>` and a "+1" button. In React, you'd call `setState` and React would update the DOM for you. In Vanilla JS:

- Keep a reference to the DOM node that displays the counter.
- When the counter state changes, update that node's text.

```js
let count = 0;
const counterEl = document.createElement('span');
counterEl.textContent = count;
appRoot.appendChild(counterEl);

const incButton = document.createElement('button');
incButton.textContent = 'Increment';
appRoot.appendChild(incButton);

incButton.addEventListener('click', () => {
  count++;
  counterEl.textContent = count; // update only the text node
});
```

This way, no unnecessary DOM nodes are created or removed – we just mutate the existing element's content. For simple apps, manual updates are fine. As apps grow, you might organize updates in a more systematic way (we'll discuss state management in the next chapter).

**Batching and reflows:** A quick performance tip – frequent DOM manipulations can cause reflows (layout calculations). If updating many pieces at once, batch them (e.g. use a fragment or do all changes inside a `requestAnimationFrame` callback). In practice, frameworks handle this, but as a Vanilla JS developer you should be mindful. Modern browsers are fast, though, so focus on correctness first, optimize if needed.

### 2.3 Handling User Events

Interactivity is where "views" come to life. In React, you'd attach event handlers in JSX (e.g. `onClick={...}`). In Vanilla JS, use the DOM API `addEventListener` to listen for events on elements.

Example – toggling a CSS class on click (like a favorite star toggling filled/outline):

```js
const star = document.createElement('span');
star.textContent = '☆';          // a star icon, could use Unicode or <svg> in real case
star.style.cursor = 'pointer';   // make it look clickable
let starred = false;
star.addEventListener('click', () => {
  starred = !starred;
  star.textContent = starred ? '★' : '☆'; // filled star vs outline
});
appRoot.appendChild(star);
```

A few things to note:
- We used a closure here (`starred` variable from outer scope) to keep track of state across clicks – a Vanilla JS analog to React's internal state.
- The event callback updates the DOM (switching the star character). This is an example of a **UI update based on an event**.
- `addEventListener` attaches the handler. There's also an older way: setting `star.onclick = () => {...}`, but `addEventListener` is preferred because you can add multiple listeners and have more control.

**Event delegation:** In dynamic lists (say a list of to-do items), adding separate listeners to each item can be inefficient. A common Vanilla JS pattern is event delegation: attach one listener on a parent (like the `<ul>`) and in the handler, check the event target to decide what to do. This way, as items are added/removed, you don't need to constantly attach/detach listeners. We'll use this pattern in our projects.

### 2.4 Comparing to React: Imperative vs Declarative

By now you might notice a shift in thinking:
- **React (Declarative):** You declare how the UI should look based on state, and React takes care of updating the DOM to match that state. You rarely manipulate the DOM directly.
- **Vanilla JS (Imperative):** You often explicitly tell the DOM what to do: create this element, update this text, remove that node, etc. 

There's no virtual DOM diff in between – *you* are effectively doing what React would do for you.

For instance, in React you might have:

```jsx
// React pseudo-code
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <span>{count}</span>
      <button onClick={() => setCount(count+1)}>Increment</button>
    </div>
  );
}
```

In Vanilla JS, we manually wrote the equivalent: keeping track of `count`, updating the `<span>` text, etc. It's more code, but it demystifies what frameworks do.

**Tip:** You can still structure your Vanilla JS in a declarative style by writing small functions that render pieces of UI given some data. For example, `function renderUser(user) { ...create DOM for user... }`. This can help maintain clarity as your app grows, and it's something we'll explore (essentially creating our own tiny UI library patterns).

### 2.5 Chapter Challenge

We've covered creating and updating DOM elements and handling user interactions. As a quick exercise (before moving on to formal state management in the next chapter), try this: **build a simple like counter widget in Vanilla JS.** It has a "Like" button and a count of likes. Each click increases the count. Use what we discussed: create elements, attach event, update text. *(If you need a hint or want to verify, consider prompting an AI with "vanilla JS code for a like button with counter").*

In the next chapter, we'll tackle state management – how to manage and structure data (beyond trivial cases like a single counter) in a Vanilla JS app, similar to how you might use React's state and context or Redux.

## Chapter 3. Managing State in Vanilla JavaScript

State management is the heart of any dynamic app. In React, you use `useState`, `useReducer`, Context, or external libraries like Redux/MobX to handle state. Without React, **you still need to manage state** – there's just no built-in global solution, so you'll use plain objects, closures, or custom patterns. The good news is that JavaScript is flexible enough to implement your own light state management.

### 3.1 What is "State" Outside React?

Simply put, **state is any data that influences what's shown on the UI,** which can change over time. In a Vanilla JS app, state can live in:
- **Variables/objects in your script module:** e.g. an array of todo items in memory.
- **The DOM itself:** e.g. the checked status of a checkbox input, content of an input field, etc., are forms of state stored in DOM properties.
- **Browser APIs:** e.g. `localStorage` for persistent state across page loads, or the URL (which holds state in query params or hash).
- **In-memory singleton store:** You might create a custom object or use an ES6 class or closure to hold global state and notify parts of your app to update (similar to a Redux store but simpler).

Let's consider a simple global state example. Suppose we want a state object that holds a count and allows components to subscribe to changes:

```js
// A simple state store in Vanilla JS
const appState = (() => {
  let state = { count: 0 };
  const listeners = [];
  return {
    getState() {
      return state;
    },
    setState(newState) {
      state = { ...state, ...newState };
      listeners.forEach(fn => fn(state));
    },
    subscribe(fn) {
      listeners.push(fn);
      return () => { // return unsubscribe function
        const index = listeners.indexOf(fn);
        if (index > -1) listeners.splice(index, 1);
      };
    }
  };
})();
```

This is a very trimmed-down store (inspired by Redux patterns). It uses a closure to hold a private `state` and `listeners`. Components can call `appState.subscribe(renderFunction)` to be notified when state changes, and `appState.setState({count: 5})` to update state. For example:

```js
// Component A: display count
const countDisplay = document.createElement('div');
function renderCount(state) {
  countDisplay.textContent = `Count: ${state.count}`;
}
appState.subscribe(renderCount);
renderCount(appState.getState());
appRoot.appendChild(countDisplay);

// Component B: increment button
const incBtn = document.createElement('button');
incBtn.textContent = "Increment";
incBtn.onclick = () => {
  const current = appState.getState().count;
  appState.setState({ count: current + 1 });
};
appRoot.appendChild(incBtn);
```

Now, whenever `incBtn` is clicked, the state's count is updated and our subscribed render function updates the display. We've essentially mimicked a React `useState` + re-render in Vanilla JS! This approach uses the **observer pattern** (listeners). It's rudimentary but works for small apps. We'll refine how we organize state as we proceed.

### 3.2 Module Pattern and Closures for State

Using closures (like the IIFE above) is one way to encapsulate state. Another common pattern is to simply use an ES module. Since each module is a singleton (evaluated once), you can have a `state.js` module:

```js
// state.js
export const state = {
  todos: [],
  filter: 'all'
};
```

Any module that imports `state` will get a reference to the same object. If one part of your code modifies `state.todos`, other parts see the change. This is a simple, albeit *ad hoc*, state container. The downside is you don't have an event system to notify of changes. You might combine this with custom events (discussed next) or manual function calls to update UI.

### 3.3 Using Custom Events to Manage State Changes

Browsers provide a `CustomEvent` constructor that lets you create and dispatch your own events. This can be a lightweight way to communicate state changes without a full pub-sub implementation.

For example, we could dispatch a custom event whenever our state updates:

```js
// When state changes:
const event = new CustomEvent('stateChange', { detail: state });
document.dispatchEvent(event);
```

And elsewhere:

```js
document.addEventListener('stateChange', e => {
  const newState = e.detail;
  // update UI accordingly
});
```

This approach treats the `document` (or any central element) as an event bus. It's not too far from how Redux might inform subscribers (though Redux doesn't use DOM events). The benefit is decoupling – components don't need to import a specific state module; they can listen for events.

We mention this pattern for completeness. In our projects, we might not need to go this far – often simply calling a render function after state change is enough. But remember, as apps grow, patterns like these become useful.

### 3.4 State in the DOM vs. State in Memory

Sometimes the DOM itself holds the state you need. For instance, the value of a text input is in the DOM (input's `.value`). You could treat that as the source of truth. Other times, you may want to sync the DOM state to your JavaScript state (e.g. on input change, save it to a `state.formData` object).

Deciding the single source of truth is important. A general rule:
- Use in-memory state (objects, etc.) as the source of truth for application data.
- Use DOM state for ephemeral UI state that doesn't need to be stored long term (e.g. current value of a search field, which you can always read directly when you need it).

If you keep multiple sources of truth, ensure they stay in sync. For example, if you have `state.filter = 'all'` and also some `<select>` with filter options, updating one should update the other.

We'll see this in projects, e.g. a todo app where the checkbox's checked state represents the todo's completed property – we'll update the data and the DOM together.

### 3.5 Local Storage and Persistence

One big advantage of writing direct browser code: you have access to `localStorage`, `sessionStorage`, IndexDB, etc., without extra libraries. `localStorage` is a simple key-value store that persists data across page loads for the same domain. It's synchronous but fine for small amounts of data (5-10MB in most browsers).

In Vanilla JS apps, it's common to save state to `localStorage` so that if the user refreshes, they don't lose their data (e.g. keeping our to-do list saved).

Example: Saving and loading state from `localStorage`:

```js
// Save todos array to localStorage
localStorage.setItem('todos', JSON.stringify(state.todos));

// Later, on page load, initialize state from storage:
const saved = localStorage.getItem('todos');
if (saved) {
  state.todos = JSON.parse(saved);
  renderTodos();  // update UI to reflect saved todos
}
```

We'll implement this in our to-do app project. It's the Vanilla JS equivalent of using a persistence layer or calling an API in a React app.

### 3.6 Summary

Managing state in Vanilla JS might seem manual, but it boils down to:
- **Where you store the data** (in-memory object, closure, etc.)
- **How you update the data** (simple assignment or through a function like `setState`)
- **How you notify the UI** to update (direct function calls, events, or re-render logic).

By understanding these mechanics, you could even write your own mini-state library if needed. In fact, many devs have created "framework-free" state management systems using exactly the patterns described ([Managing State in Small Apps - DEV Community](https://dev.to/vicradon/managing-state-in-small-apps-25kd#:~:text=Managing%20State%20in%20Small%20Apps,management%20because%20variables%20weren%27t%20enough)).

Up next, we'll discuss how to structure a larger UI by composing pieces – essentially, how to mimic React components with Vanilla JS, so our code stays maintainable as the app grows.

*(AI Assistant Prompt: "Show an example of a simple state store in plain JavaScript with subscribe/unsubscribe functionality." – Use this to compare with our `appState` example and deepen your understanding.)*

## Chapter 4. Component Composition and Code Organization

One reason for React's popularity is the **component model** – breaking the UI into self-contained, reusable pieces that manage their own state and logic. In Vanilla JS, we don't have a built-in component system, but we can **create our own structures and patterns** to achieve similar modularity. This chapter explores ways to compose UIs from smaller parts using functions, classes, and web standards.

### 4.1 Thinking in Components (Without a Framework)

Even without React, it's wise to break your UI into logical chunks. For example, a to-do app might have a header, a list, and a form. We can represent each chunk with a function that creates its DOM elements and returns an element or a set of elements.

**Functional Components in Vanilla JS:** You can write a function that returns a DOM node. For instance:

```js
function createTodoItem(todo) {
  const li = document.createElement('li');
  li.textContent = todo.title;
  if (todo.done) li.classList.add('done');
  li.addEventListener('click', () => {
    todo.done = !todo.done;
    li.classList.toggle('done');
  });
  return li;
}
```

This function takes some data (`todo`) and returns a DOM element (`<li>`). You could use it like:

```js
const list = document.createElement('ul');
state.todos.forEach(todo => {
  list.appendChild(createTodoItem(todo));
});
```

We've essentially made a Vanilla JS "component" for a todo list item, with internal behavior (toggling done). It's not as robust as a React component (no lifecycle beyond what we coded, and it directly uses the `todo` object), but it's a building block.

**Composite Components:** We can have one component use another. For example, a `TodoList` component could loop through todos and call `createTodoItem` for each, as above. Or a `Header` component could create a title element and perhaps some buttons, and return a `<header>` element containing them.

By composing functions, you keep each part focused and reuse them easily.

### 4.2 Encapsulating Behavior with Classes

ES6 classes provide a convenient way to bundle state, DOM creation, and event handlers in one place. You can think of a class instance as a component instance.

For example, let's create a simple **Counter component** as a class:

```js
class CounterComponent {
  constructor(initialCount = 0) {
    this.count = initialCount;
    // Create DOM elements
    this.el = document.createElement('div');
    this.display = document.createElement('span');
    this.display.textContent = this.count;
    const incBtn = document.createElement('button');
    incBtn.textContent = '+1';
    // Append elements
    this.el.appendChild(this.display);
    this.el.appendChild(incBtn);
    // Event handler
    incBtn.addEventListener('click', () => this.increment());
  }
  increment() {
    this.count++;
    this.display.textContent = this.count;
  }
  mount(parent) {
    parent.appendChild(this.el);
  }
}
```

Using it:

```js
const counter = new CounterComponent(5);
counter.mount(document.body);
```

This class encapsulates the counter's state (`this.count`), the DOM (`this.el` and children), and the logic to update (`increment`). The `mount` method attaches the component to a parent. We could also add an `unmount` method if we wanted to remove it and do any cleanup (in this simple case, removing the DOM node removes the event listener automatically because it's tied to that node).

This pattern is similar to how one might write a component in plain JS or even using something like Stimulus or other light frameworks. It's imperative, but organized.

**Pros:** You keep related code together. If the component gets more complex (say a counter with +/- and reset), the class can hold all that logic.

**Cons:** It's a bit verbose, and you need to be careful with `this` binding (we used an arrow function for the event, which captures `this` correctly).

We won't necessarily use classes for every component in the book, but it's good to know this option, especially for widgets that manage their internal state extensively.

### 4.3 Templates and HTML Fragments

Another approach to defining a component's structure is using HTML templates. The `<template>` tag in HTML is inert content that you can clone in JS. For example:

```html
<template id="itemTemplate">
  <li class="item">
    <span class="title"></span>
    <button class="delete">✕</button>
  </li>
</template>
```

In JS:

```js
const template = document.getElementById('itemTemplate');
function renderItem(item) {
  const fragment = template.content.cloneNode(true);
  fragment.querySelector('.title').textContent = item.title;
  fragment.querySelector('.delete').onclick = () => removeItem(item);
  return fragment;
}
```

This returns a DocumentFragment containing a populated `<li>` for the item. Using templates can separate HTML structure from JS logic, which some developers prefer. It's somewhat analogous to JSX – you design the structure in HTML and fill in data via JS.

We won't heavily rely on templates in our examples (to keep everything in one place in examples), but be aware of this tool for larger projects or when you have a lot of static HTML to generate.

### 4.4 Web Components (Custom Elements)

Web Components are a set of web platform APIs for creating reusable custom elements with encapsulated HTML, styling, and behavior. They are truly the browser's answer to a component system. You can define a class that extends `HTMLElement` and then use your element via a custom tag.

Basic example of a Web Component:

```js
class HelloWorld extends HTMLElement {
  connectedCallback() {
    this.textContent = "Hello, World!";
  }
}
customElements.define('hello-world', HelloWorld);
```

Now in HTML/JS, you can do `<hello-world></hello-world>` and it will invoke that class. Web Components can even have their own internal shadow DOM for encapsulated styling.

While powerful, Web Components can be overkill for our needs here. They shine for creating UI libraries or design systems that can be used across frameworks. In our context – converting React devs to Vanilla JS – we typically won't need full custom elements. However, it's good to know that the platform offers this. If you find yourself reinventing a lot of component management, exploring Web Components might be worthwhile.

### 4.5 Organizing Files and Modules

As you build components, consider splitting them into modules (files). Maybe you have `components/Header.js`, `components/TodoList.js`, etc., which export functions or classes. Then your main script can import those and assemble the app. For example:

```js
// main.js
import { Header } from './components/Header.js';
import { TodoList } from './components/TodoList.js';

document.body.appendChild(Header());
document.body.appendChild(TodoList());
```

Using ES module imports keeps each file focused. In a no-build setup, you can include `<script type="module" src="main.js"></script>` in your HTML and it will load imports (note: you must serve via HTTP, not file://, due to module CORS restrictions).

We'll use modules in later chapters when introducing bundling and for organizing project code. For now, know that the **composition doesn't only happen in code structures but in how you lay out your codebase**.

### 4.6 Reusability and Parameterization

A Vanilla JS "component" can be made configurable by parameters. For instance, a `createButton(text, onClick)` function can generate different buttons as needed. This is similar to React props.

```js
function createButton(label, onClick) {
  const btn = document.createElement('button');
  btn.textContent = label;
  if (onClick) btn.addEventListener('click', onClick);
  return btn;
}
// usage
const saveBtn = createButton('Save', () => saveData());
const cancelBtn = createButton('Cancel', () => cancel());
```

By passing callbacks and other data, you make your Vanilla components flexible. Composing these, you might build an entire toolbar or form out of small pieces.

### 4.7 Summary

We've explored multiple strategies to compose and reuse UI pieces without React:
- Functions that return DOM (like our `createTodoItem`).
- Classes to encapsulate more complex components (`CounterComponent`).
- Template-driven rendering.
- Even Web Components as an advanced option.

In practice, you might mix and match these. For example, use simple functions for straightforward markup and a class for something with interactive internal state.

The key is **modularity**: keeping code for each piece together and making it easy to assemble pieces into a full app.

In upcoming project chapters, we will apply these techniques. You'll see how a to-do app can be built from small functions, or how a movie app can separate concerns (search bar, movie list, movie details) into distinct units of code.

*(AI Assistant Tip: To experiment, ask "How can I create a reusable modal dialog component with Vanilla JS classes?" This can show you another real-world use of classes for components and reinforce the concepts.)*

## Chapter 5. Routing: Single-Page Application Navigation

Most modern apps are **Single-Page Applications (SPAs)**, meaning they update the UI without full page reloads, even as the user navigates to different "pages" or views. In React, you might use React Router to handle this. In Vanilla JS, we can achieve routing using the **History API** (or the older hash-based approach). This chapter covers implementing client-side routing from scratch.

### 5.1 The Browser History API

The History API lets us modify the browser's URL and history stack without reloading the page. Key parts of the API:
- `history.pushState(stateObj, title, url)` – adds a new entry to the history stack with a new URL (the domain must remain the same) ([How do you use `window.history` API? - GreatFrontEnd](https://www.greatfrontend.com/questions/quiz/how-do-you-use-windowhistory-api#:~:text=How%20do%20you%20use%20,entry%20to%20the%20history%20stack)).
- `history.replaceState(stateObj, title, url)` – replaces the current entry.
- The **`popstate`** event – fired on the window when the active history entry changes (e.g., user clicks Back or forward, or you call pushState).

Using `pushState`, we can change the URL as if navigating. For example:

```js
// Navigate to "/about" without reloading
history.pushState({}, "", "/about");
```

This will change the URL shown in the address bar to `/about`. If the user hits enter or refresh at this point, the server needs to handle `/about` (or you need a fallback to index.html). But within our SPA, after calling pushState, we should update the UI to show the "About" content. 

Listening to `popstate` allows us to respond to back/forward navigation. For instance:

```js
window.addEventListener('popstate', (event) => {
  // event.state contains the stateObj we passed (if any)
  renderCurrentRoute();
});
```

We'll likely use `location.pathname` to determine what to show.

### 5.2 Implementing a Simple Router

Let's sketch a basic router for our Vanilla JS app. Suppose we want three "routes": Home, About, and a dynamic route for a user profile (e.g., `/user/123`).

We can define a function `routeTo(path)` that uses pushState and then calls a render:

```js
function routeTo(path) {
  history.pushState({}, "", path);
  renderCurrentRoute();
}
```

And `renderCurrentRoute` might look like:

```js
function renderCurrentRoute() {
  const path = location.pathname;
  appRoot.innerHTML = ""; // clear existing UI or use a specific outlet element
  if (path === "/") {
    appRoot.appendChild(renderHome());
  } else if (path === "/about") {
    appRoot.appendChild(renderAbout());
  } else if (path.startsWith("/user/")) {
    const userId = path.split("/")[2];
    appRoot.appendChild(renderUserProfile(userId));
  } else {
    appRoot.appendChild(renderNotFound());
  }
}
```

Here, `renderHome`, `renderAbout`, etc., are functions that return DOM content for those screens (these would be like our components or page modules). We clear the `appRoot` (assumed to be a container div) and append the new content.

We also need to handle clicks on links. Normally, an `<a href="/about">About</a>` click would do a full page load. We can intercept clicks and call `routeTo` instead:
```js
document.addEventListener('click', e => {
  const link = e.target.closest('a');
  if (link && link.getAttribute('href').startsWith("/")) {
    e.preventDefault();
    routeTo(link.getAttribute('href'));
  }
});
```
This uses event delegation to catch any anchor click where the href is a path (we ensure it's not an external link by checking it starts with "/"). We prevent the default navigation and instead do client-side routing.

Now our app can navigate between views without reload. This is essentially what React Router does behind the scenes (minus a lot of bells and whistles).

**Hash-based Routing:** As an alternative, you can use the URL hash (`location.hash`) to track state. E.g., `example.com/index.html#about`. This is simpler (the `hashchange` event fires when `window.location.hash` changes), and doesn't require server-side handling for different paths. However, it makes your URLs a bit uglier (`#` fragment) and is generally less powerful. Still, if you can't configure your server to handle SPA routes, hash routing is a fallback. For brevity, we focus on History API routing here.

### 5.3 Dynamic Routes and Parameters

As shown with `/user/123`, you can parse `location.pathname` to extract IDs or names. You might create a simple router configuration:

```js
const routes = [
  { path: /^\/$/, render: renderHome },
  { path: /^\/about$/, render: renderAbout },
  { path: /^\/user\/(\d+)$/, render: (id) => renderUserProfile(id) }
];
```

Then in `renderCurrentRoute`, find the first route whose regex matches the current path:
```js
for (let route of routes) {
  const match = route.path.exec(path);
  if (match) {
    const params = match.slice(1); // capture groups as params
    const page = route.render(...params);
    appRoot.innerHTML = "";
    appRoot.appendChild(page);
    return;
  }
}
// if no match, render 404
appRoot.innerHTML = "";
appRoot.appendChild(renderNotFound());
```

This way, our router can handle patterns and pass parameters. We're essentially building a mini router. For complex apps, you might consider using a small routing library even in Vanilla JS, but our goal is to demystify, so rolling our own is fine.

### 5.4 Preserving Application State Between Routes

One tricky part: when we "navigate" (pushState) and rebuild the UI, what happens to our state? For example, if you have a list of items loaded on Home, then go to About and back, do you reload the list or keep it?

In a SPA, since we never truly unload the page, we can keep state in memory. Our `appState` or other global variables remain. It's up to us to decide if we recreate components or preserve them. 

A simple approach:
- When leaving a route, you could save some state (maybe in a global or in history state).
- When entering, check if you already have the data.

For instance, if our Home view fetches data, we might store it in `state.homeData`. Next time we render Home, see if `state.homeData` exists to avoid refetching.

Alternatively, use the History state object: `pushState({ someData }, "", "/path")` and retrieve it via `event.state` in popstate. This is more advanced and often not necessary unless you want to restore scroll positions or form data on back navigation.

For our purposes, we'll keep it simple: global state (or module-level state) stays alive throughout page navigation.

### 5.5 Integrating Routing into Our Projects

When we build the movie app or others, we may incorporate routing. For example, in the movie app, clicking a movie could push a `/movie/123` route and show details. We'll apply what we learned to make that happen.

One thing to be careful of: If the user directly visits `/movie/123` as a fresh load, our SPA code should handle it (likely by detecting the path on load and fetching the movie details). Also, server-side should ideally serve the base page for any route (or redirect to `index.html`) to ensure the SPA can boot up. This is beyond JavaScript's scope – it's a deployment/config task.

### 5.6 Summary

We've created a basic client-side router:
- Used `history.pushState` to change URL without reload.
- Listened to `popstate` to handle back/forward.
- Intercepted link clicks to trigger our routing.
- Parsed dynamic segments from the URL.

With this in place, your Vanilla JS app can have multiple views just like a React app with React Router – and users can bookmark/share URLs for specific screens.

This manual approach gives insight into how routing works under the hood. It's not much code, and for many cases it's enough. For extremely complex routing (nested routes, etc.), you could still bring in a tiny library, but most SPAs can be handled with patterns like above.

*(Try It: Use an AI assistant to ask "How do I implement a single-page app router in vanilla JavaScript?" Compare the answer to our approach – you might find similar patterns or alternative techniques to learn from.)*

## Chapter 6. Managing Side Effects and Lifecycle in Vanilla JS

In React, effects (via `useEffect` or lifecycle methods) let you run code on mount/unmount or when certain state changes, often for things like data fetching or integrating with external APIs (including the browser APIs). In Vanilla JS, you don't have a formal lifecycle, but you still need to manage **side effects** – e.g., fetch data when a view loads, set up an interval or subscription, and clean it up when done.

This chapter covers how to handle these scenarios explicitly.

### 6.1 What Are "Effects" in Our Context?

A side effect is anything that affects something outside the function's scope or is affected by the outside world. Common examples in web apps:
- Network requests (fetching data from an API).
- Timer functions (setInterval, setTimeout).
- Subscribing to events (DOM events, WebSocket messages, etc.).
- Logging, localStorage access, etc.

In React, an effect might run on component mount (and optionally cleanup on unmount). In Vanilla JS, we'll typically run such code when we **render or show a component**, and ensure to stop/cleanup when we **hide or remove that component**.

### 6.2 Data Fetching (Async Effects)

Let's say we have a profile view that needs to fetch user data from an API when it loads. In Vanilla JS:
- We can call the fetch function at the appropriate time (e.g., right after creating the view's DOM, or when routing to that view).
- We then update the DOM with the fetched data when it arrives.

Example pattern:
```js
function renderUserProfile(userId) {
  const container = document.createElement('div');
  container.textContent = "Loading...";
  // Kick off fetch
  fetch(`https://api.example.com/users/${userId}`)
    .then(res => res.json())
    .then(data => {
      container.textContent = ""; // clear "Loading"
      container.appendChild(renderUserDetails(data));
    })
    .catch(err => {
      container.textContent = "Error loading user.";
      console.error(err);
    });
  return container;
}
```

Here, `renderUserProfile` initiates an effect (a data fetch) and returns a container that will eventually be filled. We immediately set some loading text to give feedback. When the promise resolves, we populate the content. This is analogous to a React component that on mount triggers a fetch and then re-renders with the data.

**Avoiding Memory Leaks:** If the user navigates away before the fetch completes, our callback still runs and tries to update `container`. In a SPA, `container` might still be in DOM if we didn't remove it, or if we removed it (on route change), updating it doesn't show anything but still occupies memory. This is generally fine, but in bigger apps you may want to abort fetches if not needed (using AbortController) or check if the element is still in DOM before updating.

### 6.3 Timers and Intervals

Consider a component that shows the current time and updates every second. In React, you might use `useEffect` to set an interval on mount and clear it on unmount. In Vanilla JS:

```js
function createClock() {
  const clockEl = document.createElement('div');
  function updateTime() {
    clockEl.textContent = new Date().toLocaleTimeString();
  }
  updateTime(); // initial call
  const timer = setInterval(updateTime, 1000);
  // we return an object with the element and a cleanup function
  return {
    el: clockEl,
    cleanup: () => clearInterval(timer)
  };
}
```

Using it:
```js
const { el: clockElement, cleanup: stopClock } = createClock();
document.body.appendChild(clockElement);
// ... later, if we want to remove the clock
stopClock();
clockElement.remove();
```

This pattern explicitly returns a cleanup function. You have to remember to call it when appropriate (e.g., when navigating away or removing that UI). 

Alternatively, if this clock is part of a larger component class, you could make `cleanup` a method of the class (like `ClockComponent.stop()` that clears the interval).

There's no magic – it's on you to stop intervals, unsubscribe from events, etc., to prevent memory leaks or unwanted background tasks.

### 6.4 Cleaning Up Event Listeners

If you attach event listeners to global objects (like `window` or `document`), be sure to remove them if they're no longer needed. For example, a component that listens to `window.resize` should probably remove that listener when the component is destroyed.

Example:
```js
function createResizableBox() {
  const box = document.createElement('div');
  box.style.width = window.innerWidth + "px";
  // Handler to resize the box with window
  function onResize() {
    box.style.width = window.innerWidth + "px";
  }
  window.addEventListener('resize', onResize);
  return {
    el: box,
    cleanup: () => window.removeEventListener('resize', onResize)
  };
}
```

Again, we package the `cleanup`. In a routed app, you might call all relevant cleanups when navigating away from a view. One strategy is to keep track: for each page, store an array of cleanup functions that should be called when leaving that page.

### 6.5 Using MutationObserver for Automatic Cleanup (Advanced)

If you want to get fancy, the DOM provides a `MutationObserver` API that can watch for DOM nodes being removed. In theory, you could attach an observer to your component's root element and detect when it's removed from DOM, then trigger cleanup. This is advanced and not commonly needed, but worth mentioning.

For instance:
```js
function createComponent() {
  const el = document.createElement('div');
  const observer = new MutationObserver((mutations) => {
    for(const m of mutations) {
      if ([...m.removedNodes].includes(el)) {
        // el was removed
        cleanup();
        observer.disconnect();
      }
    }
  });
  observer.observe(el.parentNode || document, { childList: true, subtree: true });
  // ... rest of component setup
}
```

This is complex and assumes the element has a parent at time of observation. Usually, it's simpler to handle it in your app logic.

### 6.6 Summary: Be Your Own useEffect

The takeaway: in Vanilla JS, **you manually manage lifecycle**:
- Run your setup code when needed (e.g., after inserting a component into DOM or when a route becomes active).
- Tear down when appropriate (before removing from DOM or on route change).

For small apps, you might not even need extensive cleanup – the garbage collector will clean intervals when the page unloads, and removing DOM elements removes their event listeners. But for long-running SPAs, proper cleanup is important to avoid performance issues.

We will implement some of these in the projects:
- In the movie app, fetching on page load and maybe clearing old content if navigating away.
- In a chat or live update scenario, we'd show how to stop intervals or WebSocket connections.

Always ask: "If I create this thing (timer, listener, etc.), when should I dispose of it?" If the answer is "when navigating away or removing the component," then ensure you have a code path for that.

*(AI practice: "How can I mimic React's useEffect cleanup in plain JavaScript?" The answer or discussion can further cement these ideas by seeing how others phrase the solution.)*

## Chapter 7. Modularization: Organizing Code with ES6 Modules

As our Vanilla JS apps grow, organizing code into multiple files becomes crucial. In the past, before ES6 modules, we had to use global variables or module loaders. Now, browsers support ES6 modules natively, enabling a structured approach to code architecture similar to how you import/export in React projects (which usually use bundlers).

This chapter will show how to use JavaScript modules to structure a project, and how to load them in a no-build environment.

### 7.1 Why Modules Matter

In a React app (with Create React App or similar), you're used to splitting code into different files and using `import/export`. This improves readability, maintainability, and reusability. The same applies to Vanilla JS:
- You can have one module handle state (`state.js`), another define a component (`TodoList.js`), etc.
- Modules help encapsulate implementation details. Variables or functions inside a module aren't global; they won't clutter the global namespace or conflict with others.

Without modules, if you include multiple `<script>` files, they all share the global scope, which can lead to name collisions or ordering issues. Modules avoid that by default – each module has its own scope.

### 7.2 Using ES6 Modules in the Browser

Using modules is straightforward:
- Each module is just a JS file that uses `export` to expose things and `import` to bring in dependencies.
- In your HTML, include the main script with `type="module"`.

**Example:**
```html
<!-- index.html -->
<script type="module" src="app.js"></script>
```

`app.js` could import other modules:
```js
// app.js (entry module)
import { initTodoApp } from './todoApp.js';
initTodoApp();
```

And `todoApp.js` might have:
```js
// todoApp.js
import { renderTodos } from './views/todoList.js';
import { setupEventHandlers } from './controllers/todoController.js';
// ... and so on
export function initTodoApp() {
  renderTodos();
  setupEventHandlers();
}
```

This looks very much like code in a React project, except no compilation is needed – modern browsers can load these modules natively.

**Important:** When running via modules, you must serve the files over HTTP (from a server). If you just open the HTML file via `file://`, the imports often get blocked by CORS ([CORS blocked when I use type="module" - The freeCodeCamp Forum](https://forum.freecodecamp.org/t/cors-blocked-when-i-use-type-module/340229#:~:text=Forum%20forum,is%20because%20you%27re%20attempting%20import)). For development, this means using a simple static server (like `python -m http.server` or a VSCode Live Server extension) to serve the files. This is still "no-build" (no bundling or transpiling), but it's "needs a dev server" – which is usually fine.

### 7.3 Module Import Patterns

You can import specific exports:
```js
import { funcA, funcB } from './module1.js';
```
Import everything as an object:
```js
import * as Utils from './utils.js';
Utils.helper();
```
Or default exports:
```js
import myDefault from './someModule.js';
```
We recommend using named exports (no default) for clarity, especially in large codebases (it makes searching for references easier).

**Re-exporting:** Modules can re-export from others. E.g., an `index.js` that re-exports all components:
```js
export { Header } from './Header.js';
export { Footer } from './Footer.js';
```
This way other parts can do `import { Header, Footer } from './components/index.js';`.

### 7.4 Module Gotchas

- **Ordering:** With modules, you don't worry about script tag order. Imports handle dependency order. Just ensure you don't create circular dependencies (Module A imports Module B which imports Module A) unless they're designed to handle that (which can lead to `undefined` for one of them if not careful).
- **Hoisting:** In a traditional script, you could use a function declared later because of hoisting. In modules, imports are static and must exist at load time. But within a module, function declarations still hoist normally.
- **Async nature:** Module loading is asynchronous by default, but your code in each module runs synchronously after dependencies load. The top-level `await` is allowed in modules if needed (in modern browsers), but use with care.

### 7.5 Example: Refactoring a Vanilla JS Project into Modules

Suppose our to-do app starts as one big `app.js`. We might refactor:
- `state.js` for state management (e.g., functions to add/remove todos and an array to hold them).
- `dom.js` for DOM references or generic helpers.
- `todoView.js` for functions to create DOM elements for todos.
- `app.js` for the main entry (wiring everything together, initial render, event binding).

Then:
```js
// state.js
export const todos = [];
export function addTodo(item) {
  todos.push(item);
}
// ... etc.
```
```js
// todoView.js
import { todos } from './state.js';
export function renderTodoList() {
  const ul = document.createElement('ul');
  todos.forEach(todo => {
    const li = document.createElement('li');
    li.textContent = todo;
    ul.appendChild(li);
  });
  return ul;
}
```
```js
// app.js
import { addTodo } from './state.js';
import { renderTodoList } from './todoView.js';

const appRoot = document.getElementById('app');
appRoot.appendChild(renderTodoList());

// e.g., add a new todo and re-render list
addTodo('Learn Vanilla JS');
appRoot.innerHTML = '';                   // clear old
appRoot.appendChild(renderTodoList());    // render updated list
```
This separation concerns: state logic vs UI rendering. This is a simplistic breakdown, but it shows that you can structure things similarly to a React+Redux separation, if you want.

### 7.6 Modules and Build Tools

One nice aspect: if you eventually introduce a bundler (Chapter 15), it will also use these same module imports. So writing your code as modules now means it's ready for a build step later if needed. Vite, Webpack, etc., will process these `import` statements and bundle the files for production. But during development, you might not even need them if your environment supports modules.

### 7.7 Summary

Using ES modules in the browser allows us to write organized, maintainable code. It's one of the biggest enablers of large-scale development without a framework. By splitting code into modules, you gain clarity and avoid the pitfalls of huge monolithic files or too many globals.

In our projects section, we'll start structuring code by features, possibly simulating what a multi-file project would look like (even if, for brevity in the book, we show code in one place, you can imagine how to separate it).

*(For a quick check, you could ask an AI: "How do I use ES6 modules in a browser without a bundler?" to see a summary of what we just covered and ensure all key points align.)*

## Chapter 8. Dynamic Component Loading and Code Splitting

In React, you might use **lazy loading** (e.g., `React.lazy`) to split code for different routes or heavy components. In Vanilla JS, the equivalent is using the `import()` function to dynamically load a module at runtime. This allows you to **pull in code only when needed**, which can improve initial load times for large apps.

We'll also touch on dynamically creating script elements or other assets, though the module approach is usually cleaner.

### 8.1 The `import()` Function (Dynamic Imports)

ESM (ECMAScript Modules) support a special syntax `import(modulePath)` that returns a Promise which resolves to the module exports. According to MDN, *"The `import()` syntax, commonly called dynamic import, is a function-like expression that allows loading an ECMAScript module asynchronously and dynamically..."* ([import() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import#:~:text=The%20,module%20environment)). In other words, instead of all imports being at the top (static), you can conditionally load one.

Example use case: We have an admin panel component that is large and only needed when the user logs in as admin.

```js
function loadAdminPanel() {
  import('./adminPanel.js')
    .then(module => {
      const showAdminPanel = module.default;  // assuming adminPanel.js exports default a function
      showAdminPanel();
    })
    .catch(err => {
      console.error("Failed to load admin panel", err);
    });
}
```

We could call `loadAdminPanel()` only after an admin login. Until that point, the code in `adminPanel.js` is not even loaded, saving bandwidth and parsing time.

If using dynamic imports for routing, perhaps in `renderCurrentRoute`:
```js
if (path === "/admin") {
  // lazy load admin page
  const { renderAdminPage } = await import('./pages/adminPage.js');
  appRoot.appendChild(renderAdminPage());
}
```
Using `await` (inside an async function) can make it cleaner than `.then`. Note that top-level await is supported in modules, so if `renderCurrentRoute` is at top level of a module script, you can do:
```js
const { renderAdminPage } = await import('./pages/adminPage.js');
```
provided your environment supports it (most modern browsers do as of 2025).

### 8.2 Code Splitting without a Bundler

When you use dynamic import in a no-bundler environment, the browser simply makes another HTTP request for that module when needed. This means you can structure your app into many modules and not all of them will load upfront, just like code splitting in bundlers.

However, if you plan to eventually bundle your app, bundlers will detect dynamic imports and create separate chunk files for them. So either way, dynamic import is the way to hint "this part can be split."

### 8.3 On-Demand Script Injection (Older technique)

Before `import()` was widespread, developers sometimes dynamically created script tags to load additional JS:

```js
function loadScript(url) {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = url;
    script.onload = resolve;
    script.onerror = reject;
    document.head.appendChild(script);
  });
}

// usage
loadScript('extra-feature.js').then(() => {
  // assume extra-feature.js defines some global function we need
  extraFeatureInit();
});
```

This is a more manual way and requires the loaded script to attach itself to global scope or call some global callback. With modules, you get a cleaner encapsulation (no globals needed, you get the module exports directly). So, while you might see this pattern historically or for loading third-party scripts, for your own code prefer dynamic import.

### 8.4 Dynamic Loading of Non-JS Assets

Sometimes you might want to dynamically load a CSS file or an image. CSS can be loaded by creating a `<link rel="stylesheet" href="...">` and appending it. Images can be created or their src changed on the fly as needed.

For example, if your app supports themes, you might not include all theme CSS by default:
```js
function loadTheme(themeName) {
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = `${themeName}.css`;
  document.head.appendChild(link);
}
```
This is less common in app logic and more in initial setup, but it's an option.

For our focus, code (JS) splitting is more relevant.

### 8.5 Real-world Application: Routing with Code Splitting

Combining with Chapter 5, you can split each route's code:
```js
switch(route) {
  case '/':
    renderHome();
    break;
  case '/movies':
    import('./pages/moviesPage.js').then(mod => {
      mod.renderMoviesPage();
    });
    break;
  // ...
}
```
If the movies page code is large (maybe includes templates or a lot of logic), it won't affect initial load. The user pays the cost only when they navigate there.

One thing to consider: error handling. If the dynamic import fails (maybe network error), you should handle the promise rejection and perhaps show an error message or retry option.

### 8.6 Performance Considerations

While splitting code can reduce initial bundle size, overdoing it might cause too many requests and slow down navigation if each piece is small but loads separately. It's a balance. Generally split along natural boundaries (pages, big features).

Browsers also often prefetch modules speculatively or you can use `<link rel="modulepreload">` to hint a module likely to be needed soon (to overlap the loading time). That's an advanced optimization.

### 8.7 Summary

Vanilla JS supports dynamic code loading through `import()`. This means our no-framework app can still get the benefits of lazy loading features, just like a React app does with code splitting. It keeps the app lean and fast, especially the initial load. 

We will use this concept for any heavy optional features in our projects. For instance, maybe the TMDB app only loads the movie detail module when a movie is clicked.

*(AI hint: Asking something like "JavaScript dynamic import example for code splitting" could yield additional insights or patterns on using this in front-end apps, reinforcing our understanding.)*

## Chapter 9. Project 1: Building a To-Do List App (Step-by-Step Tutorial)

Now that we've covered the core concepts of Vanilla JS development, let's put them into practice with our first project: a classic **To-Do List application**. This project will solidify basics like DOM manipulation, events, and state management. Even if you've built something similar with React, doing it in plain JS will be enlightening.

**Project Overview:** A simple to-do app where users can:
- Add new tasks.
- See a list of tasks.
- Mark tasks as completed or delete tasks.
- (Optional) Persist tasks in localStorage so they remain on refresh.

We'll build this incrementally, focusing on clean structure and Vanilla JS techniques.

### 9.1 Setting Up the HTML and CSS

For this project, our HTML is minimal – we mostly generate elements via JavaScript, but we'll set up a basic structure:

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>To-Do App</title>
  <style>
    /* Basic styles for a nicer appearance */
    body { font-family: sans-serif; max-width: 400px; margin: 2em auto; }
    h1 { text-align: center; }
    li.completed { text-decoration: line-through; color: gray; }
    button { margin-left: 0.5em; }
  </style>
</head>
<body>
  <h1>My To-Do</h1>
  <div id="app"></div>
  <script type="module" src="todo-app.js"></script>
</body>
</html>
```

We include a `<div id="app"></div>` as a mounting point, and a `todo-app.js` module where our JS code will reside. We used a few inline styles for simplicity (a real project might use a separate CSS file).

### 9.2 Structuring the App Script

In **todo-app.js**, we'll outline our plan:
- Maintain an array of tasks as state.
- Render the list of tasks.
- Provide a form or input to add a new task.
- Handle click events for completing/deleting tasks.

Let's start by defining our state and initial render:

```js
// todo-app.js
let todos = [];  // state: array of { text: string, completed: bool }
const app = document.getElementById('app');

// Initial render function
function render() {
  app.innerHTML = "";  // clear the app container

  // Create the input form
  const form = document.createElement('form');
  form.innerHTML = `
    <input type="text" name="task" placeholder="New task" required />
    <button type="submit">Add</button>
  `;
  app.appendChild(form);

  // Create the list
  const ul = document.createElement('ul');
  todos.forEach((todo, index) => {
    ul.appendChild(createTodoItem(todo, index));
  });
  app.appendChild(ul);
}
```

We used `innerHTML` for the form for brevity – injecting a small snippet. Alternatively, we could create the elements manually. The form includes a text input and a submit button.

For each todo, we call `createTodoItem(todo, index)` to get an `<li>` element for that todo.

Now, implement `createTodoItem`:

```js
function createTodoItem(todo, index) {
  const li = document.createElement('li');
  li.textContent = todo.text;
  if (todo.completed) {
    li.classList.add('completed');
  }

  // Complete/uncomplete toggle on click
  li.addEventListener('click', () => {
    todos[index].completed = !todos[index].completed;
    render();
  });

  // Delete button
  const delBtn = document.createElement('button');
  delBtn.textContent = "Delete";
  delBtn.addEventListener('click', (e) => {
    e.stopPropagation();  // prevent also toggling completion
    todos.splice(index, 1);
    render();
  });
  li.appendChild(delBtn);

  return li;
}
```

Each list item displays the todo text, and if `todo.completed` is true, we add a CSS class that crossed it out. Clicking the `<li>` toggles completion state (we update the array and re-render). We also append a "Delete" button to each item; clicking delete removes that item from the array and re-renders. Note the `e.stopPropagation()` – this ensures the click on the delete button doesn't trigger the `<li>`'s click which would toggle the item. This demonstrates event propagation control.

### 9.3 Handling New Task Submission

We have a form in our render. We need to handle its submit event to add a new task:

```js
// Attach event handler after rendering form
form.addEventListener('submit', (e) => {
  e.preventDefault();
  const input = form.elements.task;
  const newTaskText = input.value.trim();
  if (newTaskText) {
    todos.push({ text: newTaskText, completed: false });
    input.value = "";
    render();
  }
});
```

We prevent default form submission (which would reload the page). We take the input's value, trim whitespace, and if it's not empty, push a new todo object into `todos`. Then clear the input and call `render()` to update the UI.

To attach this event, we should place that code inside `render()` right after creating/adding the form element, so that `form` is in scope. Alternatively, we could attach once and not re-create form each time – but re-rendering is simple enough here.

Actually, because we re-render everything on each change, note that we're recreating the form and list each time. This is a straightforward approach (similar to re-rendering in React), but it means the entire list DOM is rebuilt. For a small app it's fine. If performance was an issue, we could update only parts (but that adds complexity; our goal here is clarity). Modern browsers handle this scale easily (dozens or even hundreds of items is fine).

### 9.4 Persistence with localStorage (Optional Enhancement)

To keep tasks across page reloads, we can utilize `localStorage`. We will:
- Load any saved tasks from localStorage on startup.
- Save the tasks to localStorage whenever they change.

Loading at the top of script:
```js
const saved = localStorage.getItem('todos');
if (saved) {
  todos = JSON.parse(saved);
}
```
After modifying `todos` (in add, toggle, delete), we can update storage:
```js
function saveTodos() {
  localStorage.setItem('todos', JSON.stringify(todos));
}
```
Call `saveTodos()` inside our event handlers whenever we mutate `todos`. For example, in the form submit, after `todos.push(...)` we do `saveTodos()`. Same in the toggle and delete handlers (after splicing or toggling).

This way, each change is persisted. When the user revisits or refreshes, the list restores.

### 9.5 Testing the App

Time to test our to-do app manually:
1. Open the HTML in a browser (via a local server since we used a module).
2. Add a few tasks, see them appear.
3. Click tasks to mark completed (they should get a line-through).
4. Click "Delete" to remove tasks.
5. Refresh the page – tasks should still be there (if you implemented localStorage).
6. Check that no errors occur in console.

You should see something functional and straightforward, built with under ~50 lines of JS logic.

This small project exemplifies many things:
- Direct DOM creation (`document.createElement`).
- Event handling (`addEventListener`).
- State management via a simple array.
- Re-rendering strategy from state (similar to React's declarative approach).
- Using browser storage for persistence.
- Preventing event propagation for nested events (important detail in event handling).

### 9.6 Possible Extensions

Feel free to extend this app as practice:
- Add a filter to show all/active/completed tasks (you'd need to add some UI and maybe a variable for current filter).
- Add an "Edit" feature to edit a task's text.
- Improve the UI with more CSS or animations on add/remove.
- Limit the scope of re-rendering (e.g., only update one item instead of full list – more complex but educational).
- Use classes or modules to organize code if converting this into a multi-file project.

### 9.7 Wrap Up

The To-Do app demonstrates that with Vanilla JS, you can accomplish the same end result as with React for simple state-driven UI. The code is a bit more manual, but also more transparent. We explicitly manipulated the DOM and managed our data – there's no magic.

Now that you're warmed up, let's move to more complex projects. Next, we'll build a **Movie Explorer app** using the TMDB API, which will involve fetching data from a server, handling asynchronous flows, and possibly routing for movie details.

*(AI Assistant Prompt: "Generate a simple to-do app in JavaScript without frameworks, with add, complete, delete features and using localStorage." – You can use this to compare with our code or get an alternate approach. But try coding it yourself first, then see if the AI does something differently or similarly.)*

## Chapter 10. Project 2: Movie Explorer (Using The Movie Database API)

Our second project is a **Movie Explorer** app that interacts with a real external API – The Movie Database (TMDB). This project will reinforce:
- Making network requests (fetching data).
- Rendering data-driven views (movie lists, details).
- Basic routing (to navigate between a list view and a detail view).
- Working with asynchronous operations in Vanilla JS.

**Project Overview:** The app will display a list of movies (e.g., popular or top-rated movies from TMDB) with their poster and title. Users can:
- Search for movies by name.
- Click a movie to view details (like description, rating, etc.) on a separate view (single page app style).
- Navigate back to the list from the detail view.

We'll focus on the core functionality, not every possible feature of TMDB (like paging, multi-category, etc., which you can add later).

### 10.1 Setup and API Key

TMDB requires an API key (which you can obtain by creating a free account on their website). For the purposes of this tutorial, let's assume you have a key and store it in a variable. In a real project, you might fetch from a backend or secure it differently, but we'll just keep it simple.

```js
const API_KEY = 'YOUR_TMDB_API_KEY_HERE';
const BASE_URL = 'https://api.themoviedb.org/3';
```

TMDB API endpoints we'll use:
- Search: `/search/movie?api_key=...&query=SEARCH_TERM`
- Popular movies: `/movie/popular?api_key=...` (for initial list or a "Trending" section)
- Movie details: `/movie/MOVIE_ID?api_key=...`

Also note: Image URLs – TMDB provides poster images via a base URL and path returned by the API. E.g., `https://image.tmdb.org/t/p/w500/POSTER_PATH` to get a 500px wide image.

### 10.2 HTML Structure

We can keep a simple HTML like:

```html
<body>
  <h1>Movie Explorer</h1>
  <div id="app"></div>
  <script type="module" src="movie-app.js"></script>
</body>
```

We'll dynamically create content, including a search form and results container.

### 10.3 Main Views and Routing

We'll have two primary views in this app:
- **Home/List View:** Shows a search bar and a grid or list of movie cards.
- **Detail View:** Shows detailed info about one movie.

We'll implement a rudimentary router to switch between these in one page. For simplicity, using hash routing might be easiest here (to avoid server setup for different paths). We can change the URL hash to `#home` or `#movie/12345`. 

Using hash means we avoid dealing with pushState for now, and just listen to `hashchange`.

Alternatively, we could manage views internally without URL changes (just show/hide), but using the URL is nice for bookmarking.

Let's plan with hash:
- Default (no hash or `#home`) -> show list view.
- `#search=term` could reflect a search query (optional).
- `#movie/{id}` -> show detail for that movie id.

### 10.4 Rendering the List View

In `movie-app.js`:

```js
const app = document.getElementById('app');

// Elements we'll reuse
let searchInput, resultsContainer;

// Render the home/list view
function renderHomeView() {
  app.innerHTML = '';
  
  // Search form
  const form = document.createElement('form');
  form.innerHTML = `
    <input type="text" id="searchInput" placeholder="Search movies..." />
    <button type="submit">Search</button>
  `;
  app.appendChild(form);
  
  resultsContainer = document.createElement('div');
  resultsContainer.id = 'results';
  app.appendChild(resultsContainer);
  
  // Event: on search form submit
  form.addEventListener('submit', e => {
    e.preventDefault();
    const query = form.querySelector('#searchInput').value.trim();
    if (query) {
      searchMovies(query);
    }
  });
  
  // If no query, load popular movies by default
  loadPopularMovies();
}
```

We create a form with an input and search button, and a container for results (could style as grid via CSS, but we'll skip detailed styling here). On submit, we call `searchMovies(query)` which will fetch and display results. If the page loads fresh, we call `loadPopularMovies()` to show some initial content.

Now, implement the functions to fetch data:

```js
async function loadPopularMovies() {
  resultsContainer.textContent = 'Loading...';
  try {
    const res = await fetch(`${BASE_URL}/movie/popular?api_key=${API_KEY}`);
    const data = await res.json();
    displayMovieResults(data.results);
  } catch (err) {
    resultsContainer.textContent = 'Failed to load movies.';
    console.error(err);
  }
}

async function searchMovies(query) {
  resultsContainer.textContent = 'Searching...';
  try {
    const res = await fetch(`${BASE_URL}/search/movie?api_key=${API_KEY}&query=${encodeURIComponent(query)}`);
    const data = await res.json();
    displayMovieResults(data.results);
    // Update the URL hash to reflect search (optional)
    window.location.hash = `search=${encodeURIComponent(query)}`;
  } catch (err) {
    resultsContainer.textContent = 'Search failed.';
    console.error(err);
  }
}
```

We use `fetch` with `async/await` for readability. We set a loading text in the results container while waiting. On success, we call `displayMovieResults`.

Now, `displayMovieResults(results)`:

```js
function displayMovieResults(movies) {
  resultsContainer.innerHTML = '';
  if (!movies || movies.length === 0) {
    resultsContainer.textContent = 'No results found.';
    return;
  }
  movies.forEach(movie => {
    const card = createMovieCard(movie);
    resultsContainer.appendChild(card);
  });
}
```

If no movies (e.g., search yielded nothing), we show a message. Otherwise, for each movie in the results array, we create a card.

Implement `createMovieCard(movie)`:

```js
function createMovieCard(movie) {
  const card = document.createElement('div');
  card.className = 'movie-card';
  card.style.cursor = 'pointer';
  
  // Poster image
  const img = document.createElement('img');
  if (movie.poster_path) {
    img.src = `https://image.tmdb.org/t/p/w200${movie.poster_path}`;
    img.alt = movie.title;
  } else {
    img.alt = 'No image';
  }
  card.appendChild(img);
  
  // Title
  const title = document.createElement('p');
  title.textContent = movie.title;
  card.appendChild(title);
  
  // Click to go to detail view
  card.addEventListener('click', () => {
    window.location.hash = `movie/${movie.id}`;
  });
  
  return card;
}
```

We create a simple card: an image and a title. We use a smaller width image (w200) for thumbnails. On card click, we set the hash to `movie/{id}` which will trigger the detail view logic (we'll write that next).

We also might add some basic styling via CSS (either here or in a `<style>`), e.g., `.movie-card { display: inline-block; margin: 10px; width: 120px; text-align: center; } img { width: 100%; }` for a grid.

### 10.5 Rendering the Detail View

When the hash is `movie/12345`, we want to fetch that movie's details and display them.

We need a function to render detail:

```js
async function renderDetailView(movieId) {
  app.innerHTML = 'Loading...';
  try {
    const res = await fetch(`${BASE_URL}/movie/${movieId}?api_key=${API_KEY}`);
    const movie = await res.json();
    displayMovieDetails(movie);
  } catch (err) {
    app.textContent = 'Failed to load movie details.';
    console.error(err);
  }
}
```

And `displayMovieDetails(movie)`:

```js
function displayMovieDetails(movie) {
  app.innerHTML = ''; // clear existing
  
  const detailsDiv = document.createElement('div');
  detailsDiv.className = 'movie-details';
  
  const title = document.createElement('h2');
  title.textContent = movie.title;
  detailsDiv.appendChild(title);
  
  const info = document.createElement('p');
  info.innerHTML = `<strong>Release:</strong> ${movie.release_date} 
                    <br><strong>Rating:</strong> ${movie.vote_average}`;
  detailsDiv.appendChild(info);
  
  const overview = document.createElement('p');
  overview.textContent = movie.overview;
  detailsDiv.appendChild(overview);
  
  if (movie.poster_path) {
    const poster = document.createElement('img');
    poster.src = `https://image.tmdb.org/t/p/w300${movie.poster_path}`;
    poster.alt = movie.title;
    detailsDiv.appendChild(poster);
  }
  
  // Back button
  const backBtn = document.createElement('button');
  backBtn.textContent = '← Back';
  backBtn.addEventListener('click', () => {
    window.location.hash = '';  // go back to home (no specific hash)
  });
  detailsDiv.appendChild(backBtn);
  
  app.appendChild(detailsDiv);
}
```

We display the title in an `<h2>`, some info (release date and rating) in a paragraph, the overview (description), and a larger poster image if available. Finally, a "Back" button that simply clears the hash (which will bring us to home view).

We append everything to `app`.

You can style `.movie-details` and its children in CSS as needed (maybe center content or layout nicely).

### 10.6 Hash-based Navigation Handling

We have `renderHomeView()` and `renderDetailView(id)`. Now we tie it together by listening to the URL hash changes:

```js
function handleHashChange() {
  const hash = window.location.hash;
  if (!hash || hash === '#home' || hash.startsWith('#search')) {
    // Home view
    renderHomeView();
    // If hash contains a search query, trigger that
    if (hash.startsWith('#search=')) {
      const q = decodeURIComponent(hash.split('=')[1]);
      if (q) {
        // Put query in search input and perform search
        const input = document.querySelector('#searchInput');
        if (input) input.value = q;
        searchMovies(q);
      }
    }
  } else if (hash.startsWith('#movie/')) {
    const movieId = hash.replace('#movie/', '');
    if (movieId) {
      renderDetailView(movieId);
    }
  }
}

// Initial load
window.addEventListener('hashchange', handleHashChange);
handleHashChange(); // call on first load
```

This function checks the hash:
- If empty or `#home` or a search, it renders the home view. If there's a search term in hash (like `#search=Inception`), it populates the search and runs it.
- If hash starts with `#movie/`, it extracts the ID and calls detail view.

We call `handleHashChange()` at startup to load the correct view based on initial URL, and again on each hash change event.

One caveat: after going to detail and clicking Back (our backBtn sets hash to ''), this triggers home view render, but note that it doesn't automatically restore the previous list of results. In our simple approach, going back to home will reload popular movies or the search if the hash was a search. If we wanted a smoother back (without re-fetching), we'd have to cache results. But to keep it straightforward, reloading is fine (and demonstrates idempotence).

### 10.7 Testing the Movie Explorer

When you run this app:
- It should show popular movies initially (with images and titles).
- Searching should fetch and show relevant movies (and update URL).
- Clicking a movie card goes to detail view with info.
- The Back button returns to home (and triggers a fresh render of home view).

Check the console for any errors or CORS issues (for API calls ensure you use correct API key and endpoint). If images are broken, verify the poster_path and URL.

This project involved more moving parts:
- **Fetching data from an API** using `fetch` and promises/await.
- **Updating the UI based on async results**, ensuring loading states.
- **Minimal routing logic** to handle multiple views (list vs detail).
- **User input handling** in the search form.
- **Dynamic creation of multiple DOM elements** for lists, which we encapsulated in helper functions for clarity.

It's essentially a small single-page application. You can enhance it by adding features like pagination (TMDB returns page info), multiple categories (Now Playing, Top Rated etc. as tabs), or even a favorite movies list stored locally.

### 10.8 Highlight: Working with Async in Vanilla JS

This app underscores how you manage async tasks without React:
- We manually show "Loading..." text while waiting for fetch responses. In React you might have a loading state and conditional render; here we directly manipulate DOM to show feedback.
- We handle errors similarly, by showing a message and logging.
- The code uses `async/await` which makes it look synchronous, but under the hood it's the event loop (the fetch finishes and then calls our code, as explained in Chapter 1).

Everything is explicit – which can actually make debugging easier since you see exactly when things happen.

*(AI Assistant Prompt: "Create a simple JavaScript app that uses the TMDB API to list movies and show details without using any frameworks." This can help you compare your solution or get hints if you get stuck with API usage. Always ensure to replace `YOUR_TMDB_API_KEY_HERE` with a valid key when running.)*

## Chapter 11. Project 3: Notes App with Routing and Persistence

For our third project, we'll build a **Notes App** – a slightly more complex CRUD application that involves multiple views and client-side routing. This will combine several concepts:
- Multi-view app structure (list of notes vs. note editor view).
- Local state management and `localStorage` persistence (to save notes).
- Event handling for creating/editing/deleting notes.
- Using URL routing (History API or hash) to navigate to specific notes.

**Project Overview:** The app will allow users to create text notes. The main page shows a list of note titles. Clicking a note opens an editor view to edit its content. There's also a button to create a new note. All notes are saved in localStorage so they persist. We'll implement routing so that each note has its own URL (like `#note/123` or using pushState), making it a richer SPA.

### 11.1 Designing the Data Structure

We need to store notes, each with an ID, title, and content. We can use an array of note objects or an object map. For simplicity, an array is fine, but we should generate unique IDs for notes (perhaps using timestamp or a simple counter).

Example note object:
```js
{ id: 1623423423, title: "Groceries", content: "Milk\nEggs\nBread" }
```

We'll keep an array `notes` in memory, and also sync it to `localStorage` for persistence.

### 11.2 Setting Up HTML and Basic Layout

Minimal HTML:
```html
<h1>Notes App</h1>
<div id="app"></div>
<script type="module" src="notes-app.js"></script>
```

We may use simple CSS: e.g., style a list, etc., but skip heavy styling in code.

### 11.3 Routing for List and Editor Views

We will have two main views:
- **ListView:** Shows all note titles and a "New Note" button.
- **EditorView:** Shows a textarea for note content and an input for title, with Save/Delete options.

We'll use hash routing again for simplicity: 
- `#notes` or empty hash -> List view.
- `#note/<id>` -> Editor view for that note.

Let's structure our script:

```js
let notes = [];
const app = document.getElementById('app');

// Load from localStorage
const savedData = localStorage.getItem('notes');
if (savedData) {
  notes = JSON.parse(savedData);
}

// Utility: save to localStorage
function saveNotes() {
  localStorage.setItem('notes', JSON.stringify(notes));
}

// Views
function renderListView() { ... }
function renderEditorView(noteId) { ... }

// Routing handler
function handleHashChange() { ... }

// Init
window.addEventListener('hashchange', handleHashChange);
handleHashChange();
```

### 11.4 Implementing List View

```js
function renderListView() {
  app.innerHTML = '';
  const listContainer = document.createElement('div');
  
  // New Note button
  const newBtn = document.createElement('button');
  newBtn.textContent = 'New Note';
  newBtn.addEventListener('click', () => {
    // Create a new note and navigate to it
    const newId = Date.now(); // simple unique id based on timestamp
    notes.push({ id: newId, title: 'Untitled', content: '' });
    saveNotes();
    window.location.hash = `note/${newId}`;
  });
  listContainer.appendChild(newBtn);
  
  // Notes list
  const ul = document.createElement('ul');
  notes.forEach(note => {
    const li = document.createElement('li');
    const title = note.title || "(Untitled)";
    li.textContent = title;
    li.style.cursor = 'pointer';
    li.addEventListener('click', () => {
      window.location.hash = `note/${note.id}`;
    });
    ul.appendChild(li);
  });
  listContainer.appendChild(ul);
  
  app.appendChild(listContainer);
}
```

This shows a list of notes. Each item clickable sets hash to that note. The "New Note" button creates a new note with a unique ID (timestamp used here; collisions unlikely if not too fast). It immediately saves and navigates to edit it.

We display "(Untitled)" for notes with empty title.

### 11.5 Implementing Editor View

```js
function renderEditorView(noteId) {
  const note = notes.find(n => n.id == noteId);
  if (!note) {
    app.innerHTML = '<p>Note not found.</p>';
    return;
  }
  app.innerHTML = '';
  
  const editorDiv = document.createElement('div');
  
  // Title input
  const titleInput = document.createElement('input');
  titleInput.type = 'text';
  titleInput.value = note.title;
  titleInput.placeholder = 'Title';
  editorDiv.appendChild(titleInput);
  
  // Content textarea
  const contentArea = document.createElement('textarea');
  contentArea.rows = 10;
  contentArea.value = note.content;
  editorDiv.appendChild(contentArea);
  
  // Save button
  const saveBtn = document.createElement('button');
  saveBtn.textContent = 'Save';
  saveBtn.addEventListener('click', () => {
    note.title = titleInput.value;
    note.content = contentArea.value;
    saveNotes();
    window.location.hash = '';  // go back to list
  });
  editorDiv.appendChild(saveBtn);
  
  // Delete button
  const deleteBtn = document.createElement('button');
  deleteBtn.textContent = 'Delete';
  deleteBtn.style.marginLeft = '10px';
  deleteBtn.addEventListener('click', () => {
    notes = notes.filter(n => n.id != noteId);
    saveNotes();
    window.location.hash = ''; // back to list
  });
  editorDiv.appendChild(deleteBtn);
  
  // Back link (if user doesn't want to save changes)
  const backLink = document.createElement('a');
  backLink.href = '#';
  backLink.textContent = 'Cancel';
  backLink.style.marginLeft = '10px';
  editorDiv.appendChild(backLink);
  
  app.appendChild(editorDiv);
}
```

This creates a simple editor:
- An input for title, pre-filled.
- A textarea for content.
- Save: updates the note object and persists, then navigates to list.
- Delete: removes the note from the array and persists, then back to list.
- A "Cancel" link that just goes back (to `#` which we treat as list). Note: if user made changes and clicks cancel, those changes are lost unless they saved; handling unsaved changes is beyond our scope here, but one could add a confirmation if needed.

We used `window.location.hash = ''` to go home (list view).

We must ensure styles to make it user-friendly (maybe width 100% for input and textarea, some margin).

### 11.6 Routing Handler

```js
function handleHashChange() {
  const hash = window.location.hash;
  if (!hash || hash === '#') {
    renderListView();
  } else if (hash.startsWith('#note/')) {
    const noteId = hash.replace('#note/', '');
    renderEditorView(noteId);
  } else {
    // Fallback for unknown hash
    renderListView();
  }
}
```

This directs to list or editor based on the hash.

### 11.7 Testing the Notes App

When you load the app:
- You should see "New Note" and perhaps an empty list if no notes.
- Click "New Note": It should create a note and navigate to editor.
- In editor, type a title and content. Click Save: should return to list and show the new note title.
- Click the note in list to reopen, edit, save again, see updates.
- Try adding multiple notes.
- Try Delete: note should be removed from list.
- Refresh page: notes should persist (localStorage working).
- Try using the Cancel link: ensure it goes back to list.

This project demonstrates:
- **Routing and multi-view management** with a shared data store (notes array).
- **Creating, updating, deleting data** and keeping UI in sync by re-rendering list or updating state.
- We didn't use any fancy framework – just logic and DOM updates.
- The use of `find` and `filter` to manage array state, similar to how you'd use useState in React with new arrays.

We should also consider edge cases:
- If list is empty, maybe show a message "No notes yet, create one".
- If editing note and user refreshes on that hash, handle it (we do: if note not found, we say not found).
- Unique ID generation: using `Date.now()` might generate same ID if user creates two notes in the same millisecond (unlikely). Could also check and loop until find unused.
- We might also want to update the list view in memory immediately when editing: currently, we update the note object which is the same reference in array, so list view will reflect new title on next render (we re-read from notes array each time we render list).

### 11.8 Reflection

This notes app essentially functions like a mini single-page CRUD application, akin to something you might use a framework for. But doing it in Vanilla JS shows that with careful planning, it's entirely feasible:
- We manually managed state (`notes` array) and UI updates.
- We leveraged browser storage without extra libraries.
- The code is longer than a typical React version might be, but it's straightforward and each part is understandable with basic JS knowledge.

*(AI Assistant Practice: "How to build a single-page notes app with JavaScript and localStorage without frameworks?" The AI might provide a solution outline; comparing it with our implementation can offer insights or alternative methods.)*

## Chapter 12. Back-End JavaScript: An Introduction to Node.js

So far we've focused on front-end (browser) JavaScript. In this chapter, we'll briefly step into the **back-end** and see how Node.js allows you to use JavaScript on the server side. While a full Node course is beyond our scope, we'll highlight key ideas so that a React/JS developer can understand the basics of running JS outside the browser.

### 12.1 What is Node.js?

Node.js is a runtime environment that allows JavaScript to run on the server (outside the browser) using Google Chrome's V8 engine ([Node.js — Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs#:~:text=Node.js%20is%20an%20open,almost%20any%20kind%20of%20project)). Essentially, Node.js lets you execute JS code on your machine or server just like you would run Python, Ruby, etc. It comes with built-in modules for things like file system access, networking, and more.

One of Node's biggest appeals: front-end developers can use the same language (JS) for server-side logic. As the Node documentation says, *"millions of frontend developers that write JavaScript for the browser are now able to write the server-side code... without the need to learn a completely different language."* ([Node.js — Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs#:~:text=Node,learn%20a%20completely%20different%20language)).

### 12.2 Running a Simple Node Script

If you have Node.js installed, you can create a file `hello.js`:
```js
console.log("Hello from Node");
```
and run it with the command `node hello.js`. This will execute the code and print to your terminal.

Node provides `console.log` just like the browser, but no `document` or `window` (since there's no DOM). Instead, you have access to Node-specific globals like `process` (for environment info, arguments, etc.).

### 12.3 Building a Simple Web Server

One common introductory example is an HTTP server. In Node, you can create a web server in a few lines:

```js
// server.js
const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

Run this with `node server.js`. It will start a server on port 3000. When you visit http://localhost:3000 in a browser, it responds with "Hello, World!" ([Node.js — Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs#:~:text=const%20,node%3Ahttp)). This example uses Node's built-in `http` module. We set headers and status, and end the response with some text.

This is low-level compared to using frameworks like Express, but it shows what's happening under the hood.

### 12.4 Using npm and Packages

Node comes with **npm (Node Package Manager)**, which lets you download and manage libraries. For instance, the popular Express framework simplifies server creation:

```bash
npm init -y            # initialize a package.json
npm install express    # install express package
```

Then in a file:
```js
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello from Express');
});

app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
```

This does the same as the raw http example but with less code, and easier routing. We won't dive deeper, but know that npm hosts thousands of packages for all kinds of functionality.

### 12.5 Node and Front-end Tooling

Even if you don't write back-end code, you've likely used Node through tools like Webpack, Babel, ESLint, Create React App, etc. These are Node programs (often installed via npm) that automate front-end build processes. For instance, when you run `npm start` in a React project, Node is running a development server behind the scenes.

You can create scripts in `package.json` to run Node-based tools. E.g.:
```json
"scripts": {
  "build": "webpack --mode production",
  "lint": "eslint src/**/*.js"
}
```
These might execute Node commands for bundling and linting code.

### 12.6 Sharing Code Between Frontend and Backend

One neat aspect of using JS everywhere is the ability to share logic. For example, if you wrote a validation function or utility in your front-end code, you could move it to a common module and use it in Node (for server-side validation) and import it in the front-end too (with bundlers or modules).

Of course, not all browser APIs exist in Node (no DOM, no `fetch` by default (though Node 18+ has fetch), etc.), but pure logic or data processing functions can be shared.

### 12.7 When to Consider Node.js

If you extend your vanilla JS project and need a backend (say, to save data permanently or to implement user authentication), Node is a natural choice as you can stay in JS. You might use Node to:
- Build a REST API that your front-end calls (like we called TMDB API, you could create your own API).
- Serve your static files (though any web server can do that).
- Perform server-side rendering (for SEO or performance) – advanced use case.
- Write scripts or command-line tools (Node is often used to make utilities, e.g., a script to generate a file from some data).

### 12.8 Quick Example: API Endpoint with Express

Just to illustrate, here's how an Express server might provide a simple API for a notes app (from project 3):

```js
const express = require('express');
const app = express();
app.use(express.json()); // to parse JSON request bodies

let notes = [];

// Get all notes
app.get('/api/notes', (req, res) => {
  res.json(notes);
});

// Create a new note
app.post('/api/notes', (req, res) => {
  const newNote = { id: Date.now(), title: req.body.title, content: req.body.content };
  notes.push(newNote);
  res.json(newNote);
});

// Delete a note
app.delete('/api/notes/:id', (req, res) => {
  notes = notes.filter(n => n.id != req.params.id);
  res.sendStatus(204);
});

app.listen(3000, () => console.log('API server running on 3000'));
```

This is a basic RESTful interface for notes. Your front-end could `fetch('/api/notes')` or `fetch('/api/notes', { method:'POST', body: JSON.stringify(note) })` etc., to interact with it.

### 12.9 Conclusion: Full-Stack JS

By combining front-end skills with Node knowledge, you become a full-stack JavaScript developer. You can handle both sides and use a single language throughout – which can be efficient for both your development process and for sharing code.

Even if you continue focusing on the front-end, understanding Node.js is useful because so many front-end tools (and the concept of module bundling, etc.) involve Node. Plus, debugging issues like CORS or server responses is easier when you know what's happening on the server.

To wrap up:
- Node.js opens up JS beyond the browser, from servers to scripts.
- It uses the same language fundamentals, but has its own APIs (file system, network, etc.) and a different event loop environment tuned for server tasks.
- Experiment with Node by writing small scripts or starting a tiny server – it helps reinforce how JavaScript can be used in different contexts.

*(AI prompt idea: "Explain Node.js to a front-end developer and show a simple HTTP server example." The answer should align with what we covered, reinforcing understanding.)*

## Chapter 13. Bundling and Build Tools: Going Production-Ready with Vite

Throughout this book, we've emphasized a no-build approach – writing modern JS and running it directly in the browser. This works great for development and simpler projects, but when you're ready to deploy a serious application, you'll likely introduce a **build step**. Bundling, minification, and other optimizations can improve performance and browser compatibility.

In this chapter, we'll discuss why bundling is useful and walk through using a modern tool like **Vite** to bundle our Vanilla JS app for production.

### 13.1 Why Bundle at All?

Historically, browsers didn't support modules, so bundlers concatenated files into one. Now that modules are supported, do we still need bundling? Possibly, for several reasons:
- **HTTP overhead**: If your app has many small files (modules), bundling can reduce the number of network requests. (Though HTTP/2 has mitigated this to some extent.)
- **Minification**: Bundlers/uglifiers remove whitespace, shorten variable names, etc., reducing file size.
- **Transpilation**: To support older browsers, you might need to transpile modern JS (like arrow functions, etc.) down to ES5. Tools like Babel can integrate with bundlers.
- **Assets & Code Splitting**: Bundlers can handle importing CSS/images and split code into chunks for lazy loading (our dynamic imports become separate files).
- **Convenience**: You can use future JS features or different languages (TypeScript, JSX, etc.) and let the build tool convert them appropriately.

As the Vite docs note, bundler-based dev setups handle many tasks but can be slow on large projects ([Why Vite | Vite](https://vite.dev/guide/why#:~:text=However%2C%20as%20we%20build%20more,affect%20developers%27%20productivity%20and%20happiness)). Vite addresses dev speed by leveraging native modules and bundling only for production ([Why Vite | Vite](https://vite.dev/guide/why#:~:text=Vite%20aims%20to%20address%20these,native%20languages)). So you get the best of both worlds: fast dev, optimized build.

### 13.2 Introducing Vite

**Vite** is a modern build tool created by Evan You (Vue.js creator). Its name means "fast" in French, and it uses native ES modules for dev (no heavy bundle on dev server) and Rollup under the hood for production bundling. It supports projects in Vanilla JS, Vue, React, etc.

Key features:
- Lightning-fast cold starts (because it doesn't bundle everything upfront).
- Hot Module Replacement (HMR) in dev for a great DX.
- Out-of-the-box support for TS, JSX, CSS, and more via plugins.
- Easy configuration (often zero-config for simple projects).

### 13.3 Setting Up Vite for a Vanilla JS Project

Let's say we want to bundle our **Movie Explorer** app for production. We can use Vite to create a scaffolding:
```
npm init vite@latest vanilla-movie-app -- --template vanilla
```
This will create a `vanilla-movie-app` folder with a basic structure for a vanilla JS project using Vite. Inside, the `index.html` and `main.js` (or similar) will be set up. If you already have your code, you can integrate Vite by:
- Ensuring your HTML has a script type=module pointing to your main JS (like `main.js`).
- Possibly adjusting module import paths if needed (Vite supports relative imports as we wrote).
- Creating a `package.json` with Vite as a dev dependency.

Then install:
```
npm install
npm run dev
```
The dev command (preconfigured by Vite) will start a dev server (likely on port 5173) that serves your app with modules. It supports HMR, so if you edit a file, the changes reflect immediately without a full reload.

### 13.4 Building for Production

When ready:
```
npm run build
```
This triggers Vite's production build. It will output files (in `dist/` by default):
- Your JavaScript bundled (maybe chunked if using dynamic import) and minified.
- An optimized `index.html` referencing the bundled files (with cache-busting hashes in filenames).
- Any CSS extracted (if you imported CSS in JS).
- Possibly smaller polyfill or manifest files if needed.

For example, our movie app might result in `assets/index-abc123.js` etc., and images might be copied over or inlined depending on config.

You can preview the production build with:
```
npm run preview
```
This serves the `dist` directory to test the result locally.

### 13.5 Configuration Options

Vite allows configuration via `vite.config.js` (or `.ts`). For a vanilla project, you might not need to change much. Some useful options:
- Setting a `base` path if your app will be served from a subdirectory.
- Adding plugins (for example, a plugin to work with a templating language or to enable legacy browser support).
- Define environment variables – Vite will replace `import.meta.env.VARIABLE` in your code with values from a `.env` file or command line.

For instance, if we had an API key we don't want in code, we might use an env var.

### 13.6 Deployment

After bundling, you can deploy the contents of `dist/` to any static hosting (e.g., GitHub Pages, Netlify, Vercel, an S3 bucket, or your own server). It's just static files.

If you used client-side routing with the History API (pushState), remember to configure the server to redirect 404s to your `index.html` (so that direct navigation works). With hash routing, that's not an issue.

### 13.7 Example: Bundling Our Notes App with Vite

To cement the idea, let's outline bundling the Notes app:
- It has multiple modules (if we split it) or one file. Vite will handle either.
- We run `npm init vite@latest notes-app -- --template vanilla`.
- Copy our HTML/JS into that scaffold (or modify existing).
- Run dev server: ensures it still works (maybe minor tweaks if needed).
- Run build: get `dist/` output.
- Deploy `dist`.

One nice thing: If we used modern JS features that some older browsers can't handle, we might need to transpile. Vite, by default, targets modern browsers (it assumes roughly ES2017+). If you need IE11 support (rare these days), you'd need a plugin (`@vitejs/plugin-legacy`) to add polyfills and transpile down. This is optional and increases bundle size, so use only if needed.

### 13.8 Summing Up: Modern Build Workflow

In 2025 and beyond, tools like Vite are becoming standard even for projects that could run without bundling, because they streamline the development and optimization process. The great part is:
- You can start without it (as we did), and only add it when needed.
- Or start with it from scratch – since it's not onerous like older bundlers.

For a React developer, Vite's workflow is similar to create-react-app but faster and more flexible.

In conclusion:
- Use bundling for production to optimize your app's load time and compatibility.
- Vite is a top choice due to its speed and simplicity for modern JS apps.
- The concepts of entry points, outputs, and plugins apply similarly if you use Webpack, Parcel, or others – but Vite's philosophy is to leverage the platform (ESM) during dev and only bundle when necessary ([Why Vite | Vite](https://vite.dev/guide/why#:~:text=Before%20ES%20modules%20were%20available,can%20run%20in%20the%20browser)).

Now you have the tools to go from a simple vanilla JS prototype to a fully optimized, deployable web app, all while understanding what's happening under the hood!

*(If curious, ask AI: "What are the benefits of using Vite over Webpack for a vanilla JS project?" to hear a reasoning similar to what we've covered, often highlighting dev experience and performance.)*

## Conclusion

Congratulations on making it through *Vanilla JavaScript for React Developers*! We've journeyed from the fundamentals of JavaScript, through building front-end features without frameworks, and even touched on server-side JS and build tools. By now, you should feel more confident in your ability to create web applications using plain JavaScript and modern browser capabilities.

**When to use Vanilla JS vs. Frameworks:** This book isn't suggesting you ditch React for every project. Frameworks shine for large-scale apps or teams, ensuring consistency and offering powerful abstractions (and a vast ecosystem). However:
- For smaller projects or simpler interactive pages, Vanilla JS can be more than sufficient – no need to pull in a heavy framework.
- Knowing Vanilla JS means you can use frameworks more effectively. You'll write better React code because you understand what happens underneath (e.g., why keys are needed in lists, how state updates trigger re-renders).
- If a framework is misbehaving or limiting, you have the skill to go outside it or debug issues at a lower level.
- Sometimes integrating a bit of Vanilla JS into a React app (for a one-off dynamic behavior or integrating a non-React library) can solve problems – you now know how to do that safely.

**Lifelong Learning:** The web platform evolves. By staying tuned to the fundamentals and new web APIs, you can often do things natively that once required a library. Keep an eye on modern APIs (like Web Components, Service Workers, WebSockets, etc.). As you saw, using the platform directly can be rewarding and performant.

In the end, whether you code with a framework or without, you are now a stronger JavaScript developer. You can appreciate the conveniences of frameworks and also thrive without them. Your "toolbelt" is larger – you can pick the right tool for the job, even if that tool is just plain JavaScript.

Happy coding, and may your future apps be robust, whether built with 100% Vanilla JS, React, or any other library. You have the knowledge to make that choice wisely and the skills to execute it effectively.

**Next Steps:** Try converting one of your small React projects into Vanilla JS as practice, or vice versa. Experiment with Web Components. Dive deeper into Node by writing a simple API for one of the projects we built. The more you apply these concepts, the more fluent you'll become. 