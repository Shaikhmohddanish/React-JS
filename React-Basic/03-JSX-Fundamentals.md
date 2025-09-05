# JSX Fundamentals

## What is JSX?

JSX (JavaScript XML) is a syntax extension for JavaScript that looks similar to HTML. It was created by Facebook for use with React. JSX makes it easier to write and add HTML in React by allowing you to write HTML-like code directly in your JavaScript.

```jsx
const element = <h1>Hello, world!</h1>;
```

This might look like a simple HTML tag, but it's actually neither a string nor HTML. It's JSX, and it's at the core of how React components render UI.

### Why was JSX created?

React separates concerns with loosely coupled units called "components" that contain both markup and logic, rather than separating technologies (like traditional HTML for structure, CSS for presentation, and JS for behavior). This approach acknowledges that rendering logic is inherently coupled with UI logic:

- How events are handled
- How state changes over time
- How data is prepared for display

JSX was created to facilitate this component-based approach by providing a syntax that makes the relationship between UI structure and logic more intuitive.

## How JSX Works Behind the Scenes

When you write JSX, it's not directly understood by the browser. Instead, a tool called Babel transpiles (converts) your JSX into standard JavaScript that browsers can understand.

For example, this JSX:

```jsx
const element = <h1>Hello, world!</h1>;
```

Is transformed by Babel into this JavaScript:

```javascript
const element = React.createElement(
  "h1",
  null,
  "Hello, world!"
);
```

And when React processes this, it creates an object that looks something like this:

```javascript
// Simplified version of what React creates
const element = {
  type: 'h1',
  props: {
    children: 'Hello, world!'
  }
};
```

This object, known as a "React element," is a lightweight description of what should be rendered. React then uses these objects to build and maintain the DOM.

## JSX Syntax Rules

JSX looks like HTML, but it follows JavaScript's rules since it ultimately compiles to JavaScript. Here are the key syntax rules you need to know:

### 1. All tags must be closed

In HTML, some tags don't need closing tags (like `<img>` or `<input>`). In JSX, every tag must be closed, either with a closing tag or as a self-closing tag:

```jsx
// Correct
<div>Content</div>
<img src="image.jpg" alt="An image" />

// Incorrect - will cause an error
<div>Content
<img src="image.jpg" alt="An image">
```

### 2. Components must return a single root element

When returning JSX from a component, you must wrap multiple elements in a single parent element:

```jsx
// Correct
function Component() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// Also correct (using React Fragment)
function Component() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}

// Incorrect - will cause an error
function Component() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}
```

React Fragment (`<>...</>`) is a special component that allows you to group elements without adding an extra node to the DOM.

### 3. Use className instead of class

Since `class` is a reserved keyword in JavaScript, JSX uses `className` to define CSS classes:

```jsx
// Correct
<div className="container">Content</div>

// Incorrect - will not work as expected
<div class="container">Content</div>
```

### 4. JavaScript expressions in JSX

You can embed JavaScript expressions in JSX by using curly braces `{}`:

```jsx
const name = "John";
const element = <h1>Hello, {name}!</h1>; // Output: Hello, John!

const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) => <li key={number}>{number}</li>);
```

Inside the curly braces, you can put any valid JavaScript expression: variables, function calls, arithmetic operations, etc.

### 5. JSX attributes use camelCase

HTML attributes in JSX are written in camelCase, rather than kebab-case:

```jsx
// Correct
<div tabIndex="0" onClick={handleClick}>Click me</div>

// Incorrect
<div tab-index="0" onclick={handleClick}>Click me</div>
```

### 6. Style attribute takes an object

The `style` attribute in JSX accepts a JavaScript object with camelCased properties:

```jsx
const divStyle = {
  backgroundColor: 'blue',
  fontSize: '20px',
  marginTop: '10px'
};

const element = <div style={divStyle}>Styled content</div>;

// Inline style
const element2 = <div style={{ color: 'red', padding: '10px' }}>Red text</div>;
```

Note that the style property names are camelCased (`backgroundColor` instead of `background-color`).

### 7. Boolean attributes

For boolean attributes, just including the attribute means it's true:

```jsx
// These are equivalent
<input disabled={true} />
<input disabled />

// To set it to false, you need to explicitly do so
<input disabled={false} />
```

### 8. HTML entities

To use HTML entities in JSX, you can either use Unicode characters directly or put them inside JavaScript strings:

```jsx
// Both of these will work
const copyright1 = <p>Copyright © 2023</p>;
const copyright2 = <p>Copyright &copy; 2023</p>;
```

### 9. Comments in JSX

Comments in JSX must be placed inside curly braces and written as JavaScript comments:

```jsx
const element = (
  <div>
    {/* This is a comment in JSX */}
    <h1>Hello, world!</h1>
    {/* 
      Multi-line
      comment
    */}
  </div>
);
```

## JSX vs HTML: Key Differences

While JSX looks similar to HTML, there are important differences to be aware of:

| Feature | HTML | JSX |
|---------|------|-----|
| Attribute names | kebab-case | camelCase |
| Class attribute | `class` | `className` |
| For attribute | `for` | `htmlFor` |
| Self-closing tags | Optional `/` | Required `/` |
| Root element | Multiple allowed | Single required (or Fragment) |
| Case sensitivity | Insensitive | Sensitive (components must start with uppercase) |
| Event handlers | `onclick="handler()"` | `onClick={handler}` |
| Style attribute | `style="color: red;"` | `style={{ color: 'red' }}` |

## JavaScript Expressions in JSX

JSX allows you to embed JavaScript expressions between curly braces `{}`. This is one of the most powerful features of JSX, as it enables dynamic content in your UI.

### Variables

You can use variables directly in JSX:

```jsx
const name = "John";
const greeting = <h1>Hello, {name}!</h1>;
```

### Object Properties

You can access object properties:

```jsx
const user = { firstName: "Jane", lastName: "Doe" };
const greeting = <h1>Hello, {user.firstName} {user.lastName}!</h1>;
```

### Functions and Method Calls

You can call functions or methods:

```jsx
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = { firstName: "Jane", lastName: "Doe" };
const greeting = <h1>Hello, {formatName(user)}!</h1>;
```

### Arithmetic Operations

You can perform calculations:

```jsx
const price = 19.99;
const tax = 0.07;
const total = <p>Total: ${price + (price * tax)}</p>;
```

### Ternary Operators

Conditional rendering with ternary operators:

```jsx
const isLoggedIn = true;
const greeting = <h1>{isLoggedIn ? 'Welcome back!' : 'Please sign in'}</h1>;
```

### Array Methods

You can use array methods like `map` to generate lists:

```jsx
const numbers = [1, 2, 3, 4, 5];
const listItems = (
  <ul>
    {numbers.map((number) => (
      <li key={number}>{number}</li>
    ))}
  </ul>
);
```

### Limitations

You can only use expressions in JSX, not statements. This means you cannot use the following inside JSX curly braces:

- `if` statements
- `for` loops
- `switch` statements
- variable declarations (`let`, `const`, `var`)

If you need more complex logic, you should put it outside the JSX or use expressions like ternary operators or immediately-invoked function expressions (IIFE).

## Common JSX Patterns

### Conditional Rendering

There are several ways to conditionally render elements in React:

#### 1. Using if statements outside JSX

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }
  return <h1>Please sign in</h1>;
}
```

#### 2. Using ternary operators

```jsx
function Greeting({ isLoggedIn }) {
  return (
    <h1>
      {isLoggedIn ? 'Welcome back!' : 'Please sign in'}
    </h1>
  );
}
```

#### 3. Using logical && operator

This is useful when you want to render something or nothing:

```jsx
function Notification({ message }) {
  return (
    <div>
      {message && <div className="notification">{message}</div>}
    </div>
  );
}
```

### Rendering Lists

Using the `map` function to render arrays of data:

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

The `key` attribute is crucial when rendering lists. It helps React identify which items have changed, been added, or been removed. Keys should be unique among siblings.

### Fragments

When you don't want to add an extra div to the DOM, use Fragments:

```jsx
function App() {
  return (
    <>
      <Header />
      <Main />
      <Footer />
    </>
  );
}
```

This is equivalent to:

```jsx
function App() {
  return (
    <React.Fragment>
      <Header />
      <Main />
      <Footer />
    </React.Fragment>
  );
}
```

### Spread Attributes

You can use the spread operator to pass all properties from an object as props:

```jsx
const buttonProps = {
  className: 'primary-button',
  onClick: handleClick,
  disabled: false
};

function App() {
  return <button {...buttonProps}>Click Me</button>;
}
```

## JSX Best Practices

### 1. Keep JSX Simple

If your JSX becomes too complex, extract parts into separate components:

```jsx
// Too complex
function App() {
  return (
    <div>
      <header>
        <nav>
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
          </ul>
        </nav>
      </header>
      <main>
        <h1>Welcome</h1>
        <p>Content goes here...</p>
      </main>
      <footer>
        <p>Copyright 2023</p>
      </footer>
    </div>
  );
}

// Better approach
function Navigation() {
  return (
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  );
}

function Header() {
  return (
    <header>
      <Navigation />
    </header>
  );
}

function Main() {
  return (
    <main>
      <h1>Welcome</h1>
      <p>Content goes here...</p>
    </main>
  );
}

function Footer() {
  return (
    <footer>
      <p>Copyright 2023</p>
    </footer>
  );
}

function App() {
  return (
    <div>
      <Header />
      <Main />
      <Footer />
    </div>
  );
}
```

### 2. Use Semantic HTML

JSX allows you to use the full range of HTML elements. Use semantic elements like `<article>`, `<section>`, `<nav>`, etc., rather than generic `<div>` elements whenever appropriate:

```jsx
// Less semantic
<div className="header">
  <div className="navigation">
    <div className="nav-item">Home</div>
  </div>
</div>

// More semantic
<header>
  <nav>
    <ul>
      <li>Home</li>
    </ul>
  </nav>
</header>
```

### 3. Handle Empty or Nullable Values

Always handle cases where data might be null or undefined:

```jsx
function UserProfile({ user }) {
  // If user is null or undefined, show a loading message
  if (!user) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### 4. Use Destructuring for Props

Destructuring makes your code cleaner and easier to read:

```jsx
// Without destructuring
function UserProfile(props) {
  return (
    <div>
      <h1>{props.name}</h1>
      <p>{props.email}</p>
    </div>
  );
}

// With destructuring
function UserProfile({ name, email }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>{email}</p>
    </div>
  );
}
```

### 5. Avoid Inline Styles for Complex Styling

While inline styles work, they're not ideal for complex styling:

```jsx
// Avoid this for complex styling
<div style={{
  backgroundColor: 'blue',
  color: 'white',
  padding: '20px',
  borderRadius: '5px',
  boxShadow: '0 2px 4px rgba(0, 0, 0, 0.1)'
}}>
  Content
</div>

// Better approach: Use CSS classes
<div className="card">Content</div>
```

### 6. Always Add Alt Text to Images

For accessibility, always include descriptive alt text for images:

```jsx
// Good
<img src="profile.jpg" alt="User profile picture" />

// Bad - missing alt text
<img src="profile.jpg" />
```

## Common JSX Pitfalls and How to Avoid Them

### 1. Forgetting that JSX Requires a Single Root Element

```jsx
// This will cause an error
function Component() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}

// Fix: Wrap in a div or Fragment
function Component() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}
```

### 2. Using `class` Instead of `className`

```jsx
// This will work but generate a warning
<div class="container">Content</div>

// Correct way
<div className="container">Content</div>
```

### 3. Using `for` Instead of `htmlFor` in Labels

```jsx
// Incorrect
<label for="username">Username:</label>
<input id="username" />

// Correct
<label htmlFor="username">Username:</label>
<input id="username" />
```

### 4. Trying to Use If Statements Directly Inside JSX

```jsx
// This will cause an error
<div>
  {if (isLoggedIn) {
    return <p>Welcome!</p>
  }}
</div>

// Fix: Use ternary operator or logical && operator
<div>
  {isLoggedIn ? <p>Welcome!</p> : null}
</div>

// Or
<div>
  {isLoggedIn && <p>Welcome!</p>}
</div>
```

### 5. Not Adding Keys to List Items

```jsx
// This will cause a warning
<ul>
  {items.map(item => (
    <li>{item.text}</li>
  ))}
</ul>

// Correct way
<ul>
  {items.map(item => (
    <li key={item.id}>{item.text}</li>
  ))}
</ul>
```

### 6. Directly Modifying State in JSX

```jsx
// Incorrect - never modify state directly
<button onClick={() => user.name = 'New Name'}>
  Change Name
</button>

// Correct - use state updating functions
<button onClick={() => setUser({ ...user, name: 'New Name' })}>
  Change Name
</button>
```

## Summary

JSX is a syntax extension for JavaScript that looks similar to HTML but works within JavaScript code. It allows you to write HTML-like code in your JavaScript, making it easier to visualize the UI structure. JSX is transpiled to regular JavaScript calls to `React.createElement()`.

Key takeaways:
- JSX combines HTML-like syntax with JavaScript expressions
- It follows JavaScript naming conventions (camelCase for properties)
- It requires all tags to be closed and components to return a single root element
- You can embed JavaScript expressions using curly braces `{}`
- JSX is not required for React, but it makes code more readable and writable

With a good understanding of JSX, you're now ready to build React components that render dynamic user interfaces.

## Practice Exercise

1. Create a simple React component that displays a user card with:
   - The user's name
   - The user's profile picture (with proper alt text)
   - The user's job title
   - A list of the user's skills

2. Add conditional rendering that shows a "Premium Member" badge only if the user is a premium member.

3. Style the card using both className and inline styles (for practice).

## Additional Resources

- [Official React Documentation on JSX](https://react.dev/learn/writing-markup-with-jsx)
- [Babel REPL](https://babeljs.io/repl) - Try JSX and see how it compiles to JavaScript
- [ESLint React Plugin](https://github.com/jsx-eslint/eslint-plugin-react) - Helps catch common JSX mistakes
