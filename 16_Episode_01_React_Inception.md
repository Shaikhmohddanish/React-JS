# Episode 1: React Inception

In this first episode, we'll dive into the fundamentals of React, understanding how it works from the ground up without any build tools or complex setups.

## 1. Setting Up React Without Any Tools

To begin with React in its simplest form, we can use CDN links to include React in our HTML file:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hello React</title>
    <link rel="stylesheet" href="./index.css" />
  </head>
  <body>
    <!-- This is the container where our React app will be mounted -->
    <div id="root">
      <!-- Any content here will be replaced when React renders -->
      <h1>Loading...</h1>
    </div>

    <!-- React library - Core React functionality -->
    <script
      crossorigin
      src="https://unpkg.com/react@18/umd/react.development.js"
    ></script>
    
    <!-- ReactDOM library - React's bridge to the DOM -->
    <script
      crossorigin
      src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"
    ></script>

    <!-- Our React code -->
    <script src="./App.js"></script>
  </body>
</html>
```

## 2. Understanding React.createElement

In React, elements are the smallest building blocks. Unlike browser DOM elements, React elements are plain JavaScript objects that are cheap to create.

React provides a function called `createElement` to create these elements:

```javascript
// React.createElement syntax:
// React.createElement(type, props, ...children)

// Creating a simple h1 element
const heading = React.createElement(
  'h1',                           // Element type
  { id: 'heading', className: 'head' },  // Props (attributes)
  'Hello World from React!'       // Children (content)
); 

console.log(heading); // This is a JavaScript object, not HTML yet
```

The `React.createElement()` function returns a JavaScript object (React element) that describes what you want to render on the screen.

## 3. Creating Nested Elements

Real web applications typically have nested HTML structures. Here's how we create nested elements with React:

```javascript
// Creating a more complex nested structure
const parent = React.createElement(
  'div', 
  { id: 'parent' }, 
  [
    React.createElement(
      'div', 
      { id: 'child' }, 
      [
        React.createElement('h1', {}, "I'm h1 Tag"),
        React.createElement('h2', {}, "I'm h2 Tag"),
      ]
    ),
    React.createElement(
      'div', 
      { id: 'child2' }, 
      [
        React.createElement('h1', {}, "I'm h1 Tag"),
        React.createElement('h2', {}, "I'm h2 Tag"),
      ]
    ),
  ]
);
```

This code creates a structure equivalent to:

```html
<div id="parent">
  <div id="child">
    <h1>I'm h1 Tag</h1>
    <h2>I'm h2 Tag</h2>
  </div>
  <div id="child2">
    <h1>I'm h1 Tag</h1>
    <h2>I'm h2 Tag</h2>
  </div>
</div>
```

## 4. Rendering Elements to the DOM

React elements are just descriptions of what you want to see on the screen. To actually render them to the DOM, we need to use ReactDOM:

```javascript
// Create a root for React to render into
const root = ReactDOM.createRoot(document.getElementById('root'));

// Render our React element into the DOM
root.render(parent);
```

The `root.render()` method takes a React element and displays it in the specified DOM container (the element with id="root" in our HTML).

## 5. Understanding React vs ReactDOM

It's important to understand the distinction between these two libraries:

- **React** is the core library for creating elements, components, and managing state. It's platform-agnostic.
- **ReactDOM** is the bridge between React and the web browser's DOM. It's responsible for rendering React elements to the DOM.

This separation allows React to be used in different environments, not just web browsers (e.g., React Native for mobile apps).

## 6. Key Concepts Covered

### What is React?

React is a JavaScript library for building user interfaces. It's maintained by Facebook and a community of developers. React allows developers to create large web applications that can change data without reloading the page.

### Why is it called "React"?

It's called "React" because it "reacts" to changes in data/state. When data changes, React efficiently updates and renders just the right components, making it fast and responsive to user interactions.

### Development vs Production Versions

- **Development versions** (react.development.js):
  - Include helpful warnings and error messages
  - Better debugging experience
  - Larger file size, slower performance

- **Production versions** (react.production.min.js):
  - Warnings removed
  - Optimized for performance
  - Minified code, smaller file size

### The crossorigin Attribute

When loading scripts from a CDN, the `crossorigin` attribute is used to handle Cross-Origin Resource Sharing (CORS):

```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
```

This attribute helps securely load resources from other domains.

### async and defer Attributes

Both attributes are used to load JavaScript files asynchronously, but they work differently:

- **async**: Loads the script asynchronously and executes it as soon as it's available, potentially before the page finishes parsing.
  ```html
  <script async src="script.js"></script>
  ```

- **defer**: Loads the script asynchronously but executes it only after the page has finished parsing.
  ```html
  <script defer src="script.js"></script>
  ```

## Conclusion

In this episode, we learned:

1. How to include React in a web page via CDN
2. How React elements are created using `React.createElement()`
3. The difference between React and ReactDOM
4. How to render React elements to the DOM

While this approach works, writing complex UIs with `React.createElement()` quickly becomes unwieldy. In the next episodes, we'll explore more efficient ways to build React applications, including JSX, components, and build tools.
