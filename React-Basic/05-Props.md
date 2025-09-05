# Props in React

## What Are Props?

Props (short for "properties") are a fundamental concept in React. They are the primary way to pass data from parent components to child components. Think of props as the arguments you pass to a function - they allow you to make your components dynamic and reusable.

Props are:
- **Read-only**: Child components cannot modify the props they receive
- **Immutable**: Once set, props cannot be changed within the component
- **Uni-directional**: Data flows down from parent to child, not the other way around

This one-way data flow helps make your application more predictable and easier to understand.

## Why Props Matter

Props are essential to React for several reasons:

1. **Component Reusability**: Props allow you to create generic components that can be customized for different use cases
2. **Component Communication**: Props enable parent components to pass data to their children
3. **Dynamic Content**: Props make it possible to render dynamic content based on the data provided
4. **Separation of Concerns**: Props help separate where data is defined from where it's used

## Basic Props Usage

### Passing Props to a Component

To pass props to a component, you add them as attributes when using the component:

```jsx
// Parent component passing props
function App() {
  return (
    <div>
      <UserProfile 
        name="John Doe" 
        age={25} 
        isAdmin={true} 
        hobbies={['Reading', 'Hiking', 'Coding']}
      />
    </div>
  );
}
```

In this example, we're passing four props to the `UserProfile` component:
- `name`: a string
- `age`: a number
- `isAdmin`: a boolean
- `hobbies`: an array of strings

Notice that we use curly braces `{}` for all non-string values (numbers, booleans, arrays, objects, expressions).

### Accessing Props in a Functional Component

In a functional component, props are received as the first parameter to the function:

```jsx
// Child component receiving props
function UserProfile(props) {
  return (
    <div className="user-profile">
      <h2>{props.name}</h2>
      <p>Age: {props.age}</p>
      {props.isAdmin && <p className="admin-badge">Administrator</p>}
      <h3>Hobbies:</h3>
      <ul>
        {props.hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Destructuring Props

For cleaner and more concise code, you can use destructuring to extract props:

```jsx
// Using destructuring for cleaner code
function UserProfile({ name, age, isAdmin, hobbies }) {
  return (
    <div className="user-profile">
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <p className="admin-badge">Administrator</p>}
      <h3>Hobbies:</h3>
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}
```

This approach has several advantages:
- It makes your code cleaner and more readable
- You immediately see which props the component expects
- You can provide default values directly in the destructuring

### Accessing Props in a Class Component

In class components, props are accessed using `this.props`:

```jsx
import React, { Component } from 'react';

class UserProfile extends Component {
  render() {
    const { name, age, isAdmin, hobbies } = this.props;
    
    return (
      <div className="user-profile">
        <h2>{name}</h2>
        <p>Age: {age}</p>
        {isAdmin && <p className="admin-badge">Administrator</p>}
        <h3>Hobbies:</h3>
        <ul>
          {hobbies.map((hobby, index) => (
            <li key={index}>{hobby}</li>
          ))}
        </ul>
      </div>
    );
  }
}
```

## Types of Props

Props can be of any JavaScript data type:

### 1. Primitive Props

```jsx
<User 
  name="John"           // String
  age={25}              // Number
  isActive={true}       // Boolean
  status={null}         // Null
  level={undefined}     // Undefined
/>
```

### 2. Object Props

```jsx
const userData = {
  firstName: 'John',
  lastName: 'Doe',
  address: {
    street: '123 Main St',
    city: 'Boston',
    zipCode: '02101'
  }
};

<UserDetails user={userData} />
```

Accessing object props:

```jsx
function UserDetails({ user }) {
  return (
    <div>
      <h2>{user.firstName} {user.lastName}</h2>
      <p>
        {user.address.street}, {user.address.city}, {user.address.zipCode}
      </p>
    </div>
  );
}
```

### 3. Array Props

```jsx
const colors = ['Red', 'Green', 'Blue'];

<ColorList colors={colors} />
```

Rendering array props:

```jsx
function ColorList({ colors }) {
  return (
    <ul>
      {colors.map((color, index) => (
        <li key={index} style={{ color: color.toLowerCase() }}>
          {color}
        </li>
      ))}
    </ul>
  );
}
```

### 4. Function Props

Props can also be functions, which is how parent components receive feedback from child components:

```jsx
function ParentComponent() {
  const handleClick = (message) => {
    alert(message);
  };

  return <Button onClick={handleClick} label="Click Me" />;
}

function Button({ onClick, label }) {
  return (
    <button onClick={() => onClick('Button was clicked!')}>
      {label}
    </button>
  );
}
```

In this example:
1. `ParentComponent` defines a function `handleClick`
2. It passes this function as a prop to `Button`
3. `Button` calls this function when clicked, passing a message back to the parent

### 5. React Element Props (Children)

You can pass React elements as props, including other components:

```jsx
function Card({ title, content, footer }) {
  return (
    <div className="card">
      <div className="card-header">{title}</div>
      <div className="card-body">{content}</div>
      <div className="card-footer">{footer}</div>
    </div>
  );
}

function App() {
  return (
    <Card
      title={<h2>Welcome to our app</h2>}
      content={<p>This is the main content of the card.</p>}
      footer={<Button label="Learn More" />}
    />
  );
}
```

## The Children Prop

React provides a special prop called `children`, which contains whatever is between the opening and closing tags of a component:

```jsx
function Container({ children }) {
  return (
    <div className="container">
      {children}
    </div>
  );
}

function App() {
  return (
    <Container>
      <h1>Hello World</h1>
      <p>This content is passed as the children prop.</p>
    </Container>
  );
}
```

In this example:
- The `Container` component receives everything between `<Container>` and `</Container>` as its `children` prop
- This pattern is very useful for creating wrapper components

### Multiple Children

The `children` prop can contain multiple elements:

```jsx
function Layout({ children }) {
  return (
    <div className="layout">
      <header>Header</header>
      <main>{children}</main>
      <footer>Footer</footer>
    </div>
  );
}

function App() {
  return (
    <Layout>
      <h1>Page Title</h1>
      <p>First paragraph</p>
      <p>Second paragraph</p>
    </Layout>
  );
}
```

### Manipulating Children

You can manipulate children before rendering them:

```jsx
function List({ children }) {
  // Create a modified version of each child with additional props
  const enhancedChildren = React.Children.map(children, (child, index) => {
    return React.cloneElement(child, {
      index: index + 1,
      key: index
    });
  });
  
  return <ul>{enhancedChildren}</ul>;
}

function App() {
  return (
    <List>
      <li>First item</li>
      <li>Second item</li>
      <li>Third item</li>
    </List>
  );
}
```

In this example:
- `React.Children.map` iterates over each child
- `React.cloneElement` creates a new element with the original properties plus new ones

## Default Props

Sometimes you want to provide default values for props that aren't passed to a component. There are several ways to define default props:

### Using Default Parameters (Functional Components)

```jsx
function UserProfile({ name = 'Guest', age = 0, isAdmin = false, hobbies = [] }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <p>Administrator</p>}
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}
```

This is the most straightforward approach with modern JavaScript.

### Using defaultProps Property (Functional or Class Components)

```jsx
function UserProfile(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>Age: {props.age}</p>
      {props.isAdmin && <p>Administrator</p>}
      <ul>
        {props.hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}

UserProfile.defaultProps = {
  name: 'Guest',
  age: 0,
  isAdmin: false,
  hobbies: []
};
```

For class components:

```jsx
class UserProfile extends React.Component {
  render() {
    const { name, age, isAdmin, hobbies } = this.props;
    
    return (
      <div>
        <h2>{name}</h2>
        <p>Age: {age}</p>
        {isAdmin && <p>Administrator</p>}
        <ul>
          {hobbies.map((hobby, index) => (
            <li key={index}>{hobby}</li>
          ))}
        </ul>
      </div>
    );
  }
}

UserProfile.defaultProps = {
  name: 'Guest',
  age: 0,
  isAdmin: false,
  hobbies: []
};
```

## Props Validation with PropTypes

As your application grows, it becomes important to validate the props your components receive. This helps catch bugs early and makes your code more self-documenting.

React previously included a built-in prop validation system called PropTypes, but it has since been moved to a separate package.

### Setting Up PropTypes

First, install the package:

```bash
npm install prop-types
```

Then import and use it:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

function UserProfile({ name, age, isAdmin, hobbies }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <p>Administrator</p>}
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}

UserProfile.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isAdmin: PropTypes.bool,
  hobbies: PropTypes.arrayOf(PropTypes.string)
};

UserProfile.defaultProps = {
  age: 0,
  isAdmin: false,
  hobbies: []
};
```

In this example:
- `name` is a required string
- `age` is an optional number
- `isAdmin` is an optional boolean
- `hobbies` is an optional array of strings

### Available PropTypes Validators

PropTypes provides validators for all JavaScript primitives and complex types:

```jsx
MyComponent.propTypes = {
  // Basic types
  optionalString: PropTypes.string,
  optionalNumber: PropTypes.number,
  optionalBool: PropTypes.bool,
  optionalSymbol: PropTypes.symbol,
  
  // Anything that can be rendered (numbers, strings, elements, or an array containing these)
  optionalNode: PropTypes.node,
  
  // A React element
  optionalElement: PropTypes.element,
  
  // A React element type (a class or function component)
  optionalElementType: PropTypes.elementType,
  
  // An instance of a specific class
  optionalMessage: PropTypes.instanceOf(Message),
  
  // Limited to specific values
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),
  
  // One of several types
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number
  ]),
  
  // Array of a specific type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),
  
  // Object with specific property types
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),
  
  // Object with specific shape
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),
  
  // Object with strict shape (no extra properties allowed)
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    age: PropTypes.number
  }),
  
  // Function
  optionalFunc: PropTypes.func,
  
  // Any type
  optionalAny: PropTypes.any
};
```

### Custom Validators

You can also create custom validators for more complex validation logic:

```jsx
function UserProfile({ name, email, age }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Email: {email}</p>
      <p>Age: {age}</p>
    </div>
  );
}

UserProfile.propTypes = {
  name: PropTypes.string.isRequired,
  email: function(props, propName, componentName) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(props[propName])) {
      return new Error(
        `Invalid prop '${propName}' supplied to '${componentName}'. 
        Expected a valid email address.`
      );
    }
  },
  age: function(props, propName, componentName) {
    if (props[propName] < 0 || props[propName] > 120) {
      return new Error(
        `Invalid prop '${propName}' supplied to '${componentName}'. 
        Age must be between 0 and 120.`
      );
    }
  }
};
```

## TypeScript and Props

If you're using TypeScript with React, you can use interfaces or types to define the shape of props instead of PropTypes:

```tsx
interface UserProfileProps {
  name: string;
  age?: number;
  isAdmin?: boolean;
  hobbies?: string[];
}

function UserProfile({ name, age = 0, isAdmin = false, hobbies = [] }: UserProfileProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isAdmin && <p>Administrator</p>}
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
    </div>
  );
}
```

TypeScript provides several advantages over PropTypes:
- Type checking at compile time (not just runtime)
- Better IDE support with autocompletion
- More powerful type system
- No need for an additional library

## Advanced Props Patterns

### 1. Spread Attributes

You can use the spread operator to pass all properties of an object as props:

```jsx
function App() {
  const userProps = {
    name: 'John Doe',
    age: 25,
    isAdmin: true,
    hobbies: ['Reading', 'Hiking', 'Coding']
  };
  
  return <UserProfile {...userProps} />;
}
```

This is especially useful when:
- You're forwarding props to a child component
- You have a lot of props to pass
- You're getting props from an API or other data source

### 2. Prop Forwarding / Props Proxy

Sometimes you want to create a wrapper component that passes most of its props to another component:

```jsx
function ButtonWithIcon({ icon, ...rest }) {
  return (
    <button {...rest}>
      <img src={icon} alt="icon" />
      {rest.children}
    </button>
  );
}

// Usage
<ButtonWithIcon 
  icon="/icons/save.png" 
  onClick={handleSave} 
  className="primary-button"
>
  Save
</ButtonWithIcon>
```

In this example:
- `icon` is used by `ButtonWithIcon`
- All other props (`onClick`, `className`, `children`, etc.) are forwarded to the `button` element

### 3. Render Props

The render props pattern involves passing a function as a prop that returns React elements:

```jsx
function Toggle({ render }) {
  const [isOn, setIsOn] = React.useState(false);
  
  const toggle = () => setIsOn(!isOn);
  
  return render({ isOn, toggle });
}

// Usage
function App() {
  return (
    <Toggle
      render={({ isOn, toggle }) => (
        <div>
          <button onClick={toggle}>
            {isOn ? 'Turn Off' : 'Turn On'}
          </button>
          <p>The switch is {isOn ? 'on' : 'off'}.</p>
        </div>
      )}
    />
  );
}
```

This pattern allows for highly customizable components where the parent controls the rendering logic.

### 4. Compound Components

Compound components are a pattern where components are designed to work together:

```jsx
function Tabs({ children }) {
  const [activeTab, setActiveTab] = React.useState(0);
  
  // Clone each child with additional props
  const enhancedChildren = React.Children.map(children, (child, index) => {
    return React.cloneElement(child, {
      isActive: index === activeTab,
      onActivate: () => setActiveTab(index),
      index
    });
  });
  
  return <div className="tabs">{enhancedChildren}</div>;
}

function Tab({ isActive, onActivate, children, index }) {
  return (
    <div 
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={onActivate}
    >
      {children}
    </div>
  );
}

// Usage
function App() {
  return (
    <Tabs>
      <Tab>Tab 1 Content</Tab>
      <Tab>Tab 2 Content</Tab>
      <Tab>Tab 3 Content</Tab>
    </Tabs>
  );
}
```

This pattern creates a more intuitive API for components that are naturally used together.

## Common Props Pitfalls and How to Avoid Them

### 1. Mutating Props

Props should never be modified inside a component:

```jsx
// WRONG
function Counter(props) {
  // This is wrong and won't work as expected
  props.count = props.count + 1;
  
  return <div>{props.count}</div>;
}

// RIGHT
function Counter({ count, onIncrement }) {
  return (
    <div>
      <p>{count}</p>
      <button onClick={onIncrement}>Increment</button>
    </div>
  );
}
```

### 2. Using Object or Array Props Without Default Values

JavaScript's default comparison for objects and arrays checks references, not values. This can lead to bugs if you don't provide default values:

```jsx
// PROBLEMATIC
function UserList({ users }) {
  // If users is undefined, this will crash
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// BETTER
function UserList({ users = [] }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 3. Not Validating Props

Failing to validate props can lead to subtle bugs and confusing error messages:

```jsx
// WITHOUT VALIDATION
function AgeDisplay({ age }) {
  // If age is a string, this addition will concatenate instead of add
  return <div>In five years, you will be {age + 5}</div>;
}

// WITH VALIDATION
function AgeDisplay({ age }) {
  // Convert to number to ensure mathematical addition
  const numericAge = Number(age);
  
  if (isNaN(numericAge)) {
    return <div>Invalid age provided</div>;
  }
  
  return <div>In five years, you will be {numericAge + 5}</div>;
}
```

### 4. Overusing Props Drilling

Passing props through many layers of components (props drilling) can make your code difficult to maintain:

```jsx
// PROPS DRILLING (can become unwieldy)
function App() {
  const [user, setUser] = useState({ name: 'John' });
  
  return (
    <Page user={user} />
  );
}

function Page({ user }) {
  return (
    <div>
      <Header user={user} />
      <Content user={user} />
    </div>
  );
}

function Header({ user }) {
  return (
    <header>
      <UserInfo user={user} />
    </header>
  );
}

function UserInfo({ user }) {
  return <div>Welcome, {user.name}</div>;
}
```

For deeply nested component trees, consider using React Context or state management libraries instead.

## Summary

Props are a fundamental part of React that allow components to be customizable and reusable. They enable a one-way data flow from parent to child components, making your application more predictable and easier to debug.

Key takeaways:
- Props are read-only and should never be modified by the receiving component
- Props can be of any JavaScript data type: primitives, objects, arrays, functions, and even React elements
- Default props provide fallback values when props aren't passed
- Prop validation helps catch bugs early and documents your component's API
- Advanced props patterns like spread attributes, render props, and compound components allow for flexible and powerful component designs

By mastering props, you'll be able to create more maintainable and reusable components in your React applications.

## Practice Exercise

Create a card component that accepts various props to customize its appearance and content:

1. The card should accept the following props:
   - `title` (string): The card's title
   - `content` (string or React element): The main content
   - `imageUrl` (string, optional): URL for an image
   - `imageAlt` (string, optional): Alt text for the image
   - `actions` (array of objects, optional): Action buttons at the bottom of the card
   - `variant` (string, optional): Style variant ('default', 'compact', 'featured')
   - `className` (string, optional): Additional CSS class

2. Implement prop validation using PropTypes

3. Provide sensible default props

4. Use the card component in different ways to demonstrate its flexibility

## Additional Resources

- [Official React Props Documentation](https://react.dev/learn/passing-props-to-a-component)
- [PropTypes Documentation](https://github.com/facebook/prop-types)
- [TypeScript with React](https://react-typescript-cheatsheet.netlify.app/)
- [React Patterns](https://reactpatterns.com/) for advanced prop usage
