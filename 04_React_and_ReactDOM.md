# React and ReactDOM

## Understanding React and ReactDOM

In the React ecosystem, there are two main packages that you need to understand:

1. **React**: The core library for creating user interfaces
2. **ReactDOM**: The glue between React and the DOM (web browser)

These packages are separate because React can be used in different environments, not just the web browser.

## React: The Core Library

React is the core library that includes:

1. **Component API**: For creating and managing components
2. **Virtual DOM**: An in-memory representation of the UI
3. **Reconciliation Algorithm**: Efficiently updates the DOM
4. **Hooks**: Functions that let you use state and other React features

```javascript
import React from 'react';
```

## ReactDOM: The Renderer

ReactDOM is a renderer that:

1. **Renders React components to the DOM**
2. **Manages DOM updates efficiently**
3. **Provides browser-specific methods**

```javascript
import { createRoot } from 'react-dom/client';
```

## Why Are They Separate?

React and ReactDOM are separate packages because:

1. **Platform Independence**: React can render to different platforms
2. **Modularity**: Keeps the core React logic separate from DOM-specific code
3. **Size Optimization**: You only include what you need

## Other React Renderers

Besides ReactDOM (for web), there are other renderers:

- **React Native**: For mobile apps
- **React-Three-Fiber**: For 3D content
- **React-PDF**: For PDF documents
- **React-Canvas**: For canvas rendering

## Key Methods from React

### React.createElement()

Creates a React element (virtual DOM node):

```javascript
React.createElement(
    type,    // Element type (string or component)
    props,   // Properties object
    ...children  // Child elements
);
```

Example:

```javascript
const element = React.createElement(
    'div',
    { className: 'container' },
    React.createElement('h1', null, 'Hello'),
    React.createElement('p', null, 'This is a paragraph')
);
```

### React.Component and React.PureComponent

Base classes for creating class components:

```javascript
class MyComponent extends React.Component {
    render() {
        return React.createElement('div', null, 'Hello from MyComponent');
    }
}
```

### React.createContext()

Creates a Context object for sharing values without prop drilling:

```javascript
const ThemeContext = React.createContext('light');
```

## Key Methods from ReactDOM

### createRoot()

Creates a root to manage a React tree:

```javascript
import { createRoot } from 'react-dom/client';

const rootElement = document.getElementById('root');
const root = createRoot(rootElement);
```

### root.render()

Renders a React element into the DOM:

```javascript
root.render(element);
```

### root.unmount()

Removes a React tree from the DOM:

```javascript
root.unmount();
```

## How React and ReactDOM Work Together

1. **React** creates and manages a virtual representation of your UI
2. **ReactDOM** takes that representation and turns it into actual DOM elements
3. When state changes, **React** computes what needs to update
4. **ReactDOM** efficiently applies those updates to the real DOM

## Basic Example Using Both

```javascript
import React from 'react';
import { createRoot } from 'react-dom/client';

// Get the root element
const rootElement = document.getElementById('root');
const root = createRoot(rootElement);

// Create a React element
const element = React.createElement(
    'div',
    { className: 'container' },
    React.createElement('h1', null, 'Hello World'),
    React.createElement('p', null, 'Welcome to React!')
);

// Render the element to the DOM
root.render(element);
```

## Differences from Regular DOM Manipulation

### React.createElement() vs document.createElement()

**document.createElement()**:
- Creates an actual DOM node immediately
- Requires manual updates to attributes and children
- Works directly with the browser DOM

```javascript
const heading = document.createElement('h1');
heading.textContent = 'Hello World';
heading.className = 'title';
document.getElementById('root').appendChild(heading);
```

**React.createElement()**:
- Creates a lightweight description of what the DOM should look like
- Does not create actual DOM nodes until render
- Creates a virtual DOM element that React can optimize

```javascript
const heading = React.createElement('h1', { className: 'title' }, 'Hello World');
root.render(heading);
```

In the next section, we'll explore creating elements with React in more detail, building more complex structures.
