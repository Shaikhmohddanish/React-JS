# Components and Elements Composition

## Understanding Composition in React

Composition is a fundamental concept in React that allows you to build complex UIs from smaller, reusable parts. Let's explore different ways to compose components and elements together.

## Elements Inside Components

The most basic form of composition is returning React elements from a component:

```jsx
function Greeting() {
  return <h1>Hello, World!</h1>;
}
```

Here, the `Greeting` component returns a single `h1` element.

## Components Inside Elements

You can include components inside JSX elements:

```jsx
function App() {
  return (
    <div className="app">
      <Greeting />
    </div>
  );
}
```

Here, the `Greeting` component is included inside a `div` element.

## Components Inside Components

Components can render other components:

```jsx
function UserProfile({ user }) {
  return (
    <div className="profile">
      <Avatar user={user} />
      <UserInfo user={user} />
    </div>
  );
}

function Avatar({ user }) {
  return <img src={user.avatarUrl} alt={user.name} />;
}

function UserInfo({ user }) {
  return (
    <div className="info">
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}
```

This creates a tree of components, each responsible for a specific part of the UI.

## Elements Inside Curly Braces {}

You can create and include elements inside JavaScript expressions in JSX:

```jsx
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.text}</li>
      ))}
    </ul>
  );
}
```

Here, we're mapping an array to create a list of `li` elements dynamically.

## Components Inside Curly Braces {}

Similarly, you can include components inside JavaScript expressions:

```jsx
function Dashboard({ user, isAdmin }) {
  return (
    <div className="dashboard">
      <Header user={user} />
      {isAdmin && <AdminPanel />}
      <Content />
      {user.notifications.length > 0 && <Notifications items={user.notifications} />}
    </div>
  );
}
```

This pattern is commonly used for conditional rendering of components.

## Understanding the {} Syntax in JSX

The curly braces `{}` in JSX serve as a "window" into JavaScript. You can put any valid JavaScript expression inside these braces:

```jsx
function Greeting({ user, time }) {
  return (
    <div>
      {/* String concatenation */}
      <h1>Hello, {user.firstName + ' ' + user.lastName}</h1>

      {/* Function call */}
      <p>{formatTime(time)}</p>

      {/* Ternary expression */}
      <span>{user.isSubscribed ? 'Premium User' : 'Free User'}</span>

      {/* Object property access */}
      <p>Age: {user.profile.age}</p>
    </div>
  );
}
```

## Rendering Arrays of Elements

React can render arrays of elements:

```jsx
function NumberList({ numbers }) {
  const listItems = numbers.map(number => (
    <li key={number}>{number}</li>
  ));

  return <ul>{listItems}</ul>;
}
```

**Important**: When rendering arrays, each item must have a unique `key` prop to help React identify which items have changed.

## Composition vs. Inheritance

React favors composition over inheritance for reusing code between components:

### Specialization with Composition

Instead of inheriting from a base component, compose components together:

```jsx
function Dialog({ title, message, children }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <p>{message}</p>
      {children}
    </div>
  );
}

function AlertDialog({ title, message }) {
  return (
    <Dialog title={title} message={message}>
      <button>OK</button>
    </Dialog>
  );
}

function ConfirmDialog({ title, message, onConfirm, onCancel }) {
  return (
    <Dialog title={title} message={message}>
      <button onClick={onConfirm}>Yes</button>
      <button onClick={onCancel}>No</button>
    </Dialog>
  );
}
```

## The Children Prop

The `children` prop is a powerful way to compose components by allowing a component to accept and render content:

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-header">
        <h2>{title}</h2>
      </div>
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

// Usage
function App() {
  return (
    <Card title="Welcome">
      <p>This is some content inside the card.</p>
      <button>Click me</button>
    </Card>
  );
}
```

## Multiple Children Slots

You can create components with multiple "slots" for content:

```jsx
function Layout({ header, sidebar, main, footer }) {
  return (
    <div className="layout">
      <header className="header">{header}</header>
      <div className="container">
        <aside className="sidebar">{sidebar}</aside>
        <main className="main">{main}</main>
      </div>
      <footer className="footer">{footer}</footer>
    </div>
  );
}

// Usage
function App() {
  return (
    <Layout
      header={<Header />}
      sidebar={<Navigation />}
      main={<Content />}
      footer={<Footer />}
    />
  );
}
```

## Composition with Render Props

Render props is a pattern where a component accepts a function prop that returns JSX:

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);

  React.useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading });
}

// Usage
function App() {
  return (
    <DataFetcher
      url="https://api.example.com/data"
      render={({ data, loading }) => (
        <div>
          {loading ? (
            <p>Loading...</p>
          ) : (
            <ul>
              {data.map(item => (
                <li key={item.id}>{item.name}</li>
              ))}
            </ul>
          )}
        </div>
      )}
    />
  );
}
```

## Higher-Order Components (HOCs)

HOCs are functions that take a component and return a new enhanced component:

```jsx
// HOC that adds logged-in user info to props
function withUser(Component) {
  return function WithUser(props) {
    const user = useCurrentUser(); // Custom hook to get current user
    return <Component user={user} {...props} />;
  };
}

// Original component
function UserGreeting({ user, ...props }) {
  return <h1>Hello, {user.name}!</h1>;
}

// Enhanced component
const UserGreetingWithUser = withUser(UserGreeting);

// Usage
function App() {
  return <UserGreetingWithUser />;
}
```

## Understanding How Composition Works Behind the Scenes

When you write JSX with components and elements composed together, Babel transforms it into nested `React.createElement()` calls:

**JSX Version**:
```jsx
function App() {
  return (
    <div className="app">
      <Header title="My App" />
      <main>
        <Sidebar items={menuItems} />
        <Content>
          <p>Welcome to my app!</p>
        </Content>
      </main>
      <Footer year={2023} />
    </div>
  );
}
```

**Transformed Version**:
```javascript
function App() {
  return React.createElement(
    "div",
    { className: "app" },
    React.createElement(Header, { title: "My App" }),
    React.createElement(
      "main",
      null,
      React.createElement(Sidebar, { items: menuItems }),
      React.createElement(
        Content,
        null,
        React.createElement("p", null, "Welcome to my app!")
      )
    ),
    React.createElement(Footer, { year: 2023 })
  );
}
```

This structure creates a tree of React elements that React can efficiently process and update.

## Conclusion

Composition is the heart of React's component model. By combining smaller, focused components in different ways, you can build complex UIs that are still maintainable and reusable. The flexibility of JSX and the component-based architecture of React make it possible to express UI structures in a way that's both powerful and intuitive.

Understanding the different ways to compose components and elements gives you the tools to structure your applications in a clean, efficient, and reusable manner. As you continue your React journey, you'll discover even more patterns and techniques for effective composition.
