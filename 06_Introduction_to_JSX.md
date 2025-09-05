# Introduction to JSX

## What is JSX?

JSX (JavaScript XML) is a syntax extension for JavaScript that looks similar to HTML. It makes it easier to write and understand React code by allowing you to write HTML-like code within JavaScript.

## JSX vs. React.createElement()

Compare these two ways of creating the same element:

**Using React.createElement()**:
```javascript
const element = React.createElement(
  'div',
  { className: 'greeting-container' },
  React.createElement('h1', null, 'Hello, World!'),
  React.createElement('p', null, 'Welcome to React.')
);
```

**Using JSX**:
```jsx
const element = (
  <div className="greeting-container">
    <h1>Hello, World!</h1>
    <p>Welcome to React.</p>
  </div>
);
```

The JSX version is more:
- **Readable**: Resembles the HTML structure it will produce
- **Intuitive**: Easier to visualize the component tree
- **Maintainable**: Easier to modify and update

## JSX Basic Syntax

### HTML-Like Tags

JSX uses HTML-like tags to define elements:

```jsx
const element = <h1>Hello, World!</h1>;
```

### Attributes in JSX

Attributes are written in camelCase (not kebab-case like in HTML):

```jsx
const element = <div className="container" id="main-container"></div>;
```

Note that `class` becomes `className` in JSX because `class` is a reserved word in JavaScript.

### Self-Closing Tags

In JSX, all tags must be properly closed. Self-closing tags need a slash:

```jsx
const image = <img src="logo.png" alt="Logo" />;
```

### Multiple Lines

For multiple lines, wrap JSX in parentheses:

```jsx
const element = (
  <div>
    <h1>Title</h1>
    <p>Paragraph</p>
  </div>
);
```

## Embedding JavaScript in JSX

### Using Curly Braces {}

You can embed any JavaScript expression inside JSX using curly braces:

```jsx
const name = 'John';
const element = <h1>Hello, {name}!</h1>;
```

### Using Expressions

Any valid JavaScript expression can go inside the curly braces:

```jsx
const element = <div>{2 + 2}</div>; // Renders as <div>4</div>
```

### Using Variables

```jsx
const user = { firstName: 'John', lastName: 'Doe' };
const element = <h1>Hello, {user.firstName} {user.lastName}!</h1>;
```

### Using Functions

```jsx
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = { firstName: 'John', lastName: 'Doe' };
const element = <h1>Hello, {formatName(user)}!</h1>;
```

### Conditional Rendering

Using ternary operators:

```jsx
const isLoggedIn = true;
const element = <div>{isLoggedIn ? 'Welcome back!' : 'Please sign in'}</div>;
```

Using logical AND:

```jsx
const showMessage = true;
const element = <div>{showMessage && <p>This is a message</p>}</div>;
```

## JSX Attributes

### Standard HTML Attributes

Most HTML attributes work in JSX with camelCase naming:

```jsx
const button = <button className="btn" onClick={handleClick}>Click Me</button>;
```

Common attribute transformations:
- `class` → `className`
- `for` → `htmlFor`
- `tabindex` → `tabIndex`
- `onclick` → `onClick`

### Custom Attributes

You can create custom attributes with `data-` prefix:

```jsx
const element = <div data-testid="test-div">Test</div>;
```

### Spread Attributes

You can use the spread operator to pass all properties from an object:

```jsx
const props = { id: 'unique', className: 'highlight', disabled: true };
const button = <button {...props}>Click Me</button>;
```

## JSX Children

### Text Children

```jsx
const element = <h1>Hello, World!</h1>;
```

### Element Children

```jsx
const element = (
  <div>
    <h1>Title</h1>
    <p>Paragraph</p>
  </div>
);
```

### Mixed Children

```jsx
const element = (
  <div>
    <h1>Hello, {name}!</h1>
    {isLoggedIn && <UserProfile user={user} />}
    <button onClick={handleLogout}>Logout</button>
  </div>
);
```

### Arrays of Elements

```jsx
const items = ['Apple', 'Banana', 'Orange'];
const list = (
  <ul>
    {items.map((item, index) => (
      <li key={index}>{item}</li>
    ))}
  </ul>
);
```

## JSX Fragments

To return multiple elements without a wrapper div:

```jsx
const fragment = (
  <>
    <h1>Title</h1>
    <p>Paragraph</p>
  </>
);
```

This is equivalent to:

```jsx
const fragment = React.createElement(
  React.Fragment,
  null,
  React.createElement('h1', null, 'Title'),
  React.createElement('p', null, 'Paragraph')
);
```

## JSX is Not HTML

Despite looking like HTML, JSX has important differences:

1. **Attributes are camelCase**: `className` instead of `class`
2. **All elements must be closed**: `<img />` instead of `<img>`
3. **JavaScript expressions are embedded with {}**
4. **Comments use JavaScript syntax**: `{/* Comment */}`
5. **It ultimately compiles to JavaScript**

In the next section, we'll explain how JSX is transformed into React elements using Babel.
