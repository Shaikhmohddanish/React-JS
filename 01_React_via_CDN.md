# Getting Started with React via CDN

React is a JavaScript library for building user interfaces. Before diving into complex tooling, let's understand how to use React in its simplest form using CDN links.

## What is a CDN?

CDN stands for Content Delivery Network. It allows you to include React directly in your HTML file without installing anything locally.

## Basic Setup with React CDN

### Step 1: Create a basic HTML file

Create a file named `index.html` with the following content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>React via CDN</title>
</head>
<body>
    <!-- This is where our React app will be mounted -->
    <div id="root"></div>

    <!-- Load React and ReactDOM from CDN -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>

    <!-- Load our React component -->
    <script src="app.js"></script>
</body>
</html>
```

### Step 2: Create your React code

Create a file named `app.js` with the following content:

```javascript
// Get the root element
const rootElement = document.getElementById('root');

// Create a React root
const root = ReactDOM.createRoot(rootElement);

// Create a React element
const element = React.createElement(
    'h1',
    { className: 'greeting' },
    'Hello, React via CDN!'
);

// Render the React element to the DOM
root.render(element);
```

### Step 3: Open the HTML file in a browser

Open your `index.html` file in a web browser, and you should see "Hello, React via CDN!" displayed.

## Understanding the Basic Example

Let's break down what's happening:

1. We include React and ReactDOM libraries via CDN links
2. We get a reference to a DOM element with ID "root"
3. We create a React root using `ReactDOM.createRoot()`
4. We create a React element using `React.createElement()`
5. We render the element to the DOM using `root.render()`

## React.createElement() Explained

The `React.createElement()` function takes three arguments:

1. **Type**: The type of HTML element ('h1', 'div', etc.) or a React component
2. **Props**: An object containing the properties you want to apply to the element
3. **Children**: The content inside the element (text or other React elements)

```javascript
React.createElement(
    type,
    [props],
    [...children]
)
```

## Creating Nested Elements

You can create more complex structures by nesting elements:

```javascript
const element = React.createElement(
    'div',
    { className: 'container' },
    React.createElement('h1', null, 'Main Heading'),
    React.createElement('p', null, 'This is a paragraph.'),
    React.createElement('button', { onClick: () => alert('Clicked!') }, 'Click Me')
);

root.render(element);
```

## Limitations of Using React via CDN

While using React via CDN is simple to set up, it has several limitations:

1. No JSX support (JSX makes writing React elements much more intuitive)
2. No module system for organizing code
3. No build step for optimizations
4. No easy way to include additional npm packages

In the next section, we'll introduce npm and start setting up a more robust development environment.
