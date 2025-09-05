# Named Exports vs Default Exports in JavaScript/React

When working with JavaScript modules in React applications, you'll frequently use exports and imports to share code between files. There are two primary ways to export functionality: named exports and default exports.

## Named Exports

Named exports allow you to export multiple values from a single module, each identified by a specific name.

### Syntax for Named Exports:

```javascript
// Exporting at declaration
export const Button = () => {
  return <button>Click Me</button>;
};

export const Input = () => {
  return <input placeholder="Type here" />;
};

export const COLORS = {
  primary: '#0066cc',
  secondary: '#ff9900'
};

// Exporting after declaration
const Header = () => {
  return <header>Site Header</header>;
};

const Footer = () => {
  return <footer>Site Footer</footer>;
};

const API_URL = 'https://api.example.com';

export { Header, Footer, API_URL };
```

### Importing Named Exports:

```javascript
// Import specific named exports
import { Button, Input } from './components';

// Rename during import to avoid naming conflicts
import { Button as CustomButton } from './components';

// Import all named exports as a namespace object
import * as Components from './components';
// Usage: <Components.Button />
```

## Default Exports

Default exports allow you to export a single "main" value from a module. Each module can have only one default export.

### Syntax for Default Exports:

```javascript
// Exporting at declaration
export default function App() {
  return (
    <div>
      <h1>My React App</h1>
    </div>
  );
}

// Exporting after declaration
const HomePage = () => {
  return <div>Welcome to our site</div>;
};

export default HomePage;
```

### Importing Default Exports:

```javascript
// Import a default export (can use any name)
import App from './App';
import HomePage from './HomePage';

// Import both default and named exports
import App, { Button, Input } from './components';
```

## Key Differences and Best Practices

1. **Number of exports:**
   - Named exports: Multiple allowed per module
   - Default export: Only one allowed per module

2. **Import syntax:**
   - Named exports: Must use curly braces `{}` and exact names (unless renamed)
   - Default exports: No curly braces, can use any name when importing

3. **Common use cases:**
   - Named exports: Libraries with multiple utilities, component libraries
   - Default exports: Main components in their own files (like `App.js` or page components)

4. **Benefits of named exports:**
   - More explicit imports (you see exactly what you're importing)
   - Better auto-import support in IDEs
   - Better for tree-shaking (removing unused code during bundling)
   - Forces consistent naming across imports

5. **Benefits of default exports:**
   - Slightly shorter import syntax
   - More flexibility in naming during import
   - Conventional for main component in a file

## Example in a React Project Structure

```
src/
├── components/
│   ├── Button.js (default export)
│   ├── Input.js (default export)
│   └── utils.js (multiple named exports)
├── pages/
│   ├── Home.js (default export)
│   └── About.js (default export)
├── constants.js (named exports only)
└── App.js (default export)
```

## Combining Both Approaches

It's common to combine both export types in a single file:

```javascript
// UserProfile.js
export const formatUsername = (username) => {
  return username.toLowerCase();
};

export const UserAvatar = ({ src }) => {
  return <img src={src} alt="User avatar" />;
};

// The main component is the default export
const UserProfile = ({ user }) => {
  return (
    <div>
      <UserAvatar src={user.avatar} />
      <h2>{formatUsername(user.name)}</h2>
    </div>
  );
};

export default UserProfile;
```

Importing would look like:

```javascript
import UserProfile, { formatUsername, UserAvatar } from './UserProfile';
```

## When to Use Each

- Use **named exports** when creating utility functions, constants, or when a file exports multiple components of equal importance
- Use **default exports** when a file has a clear primary component or function
- For larger applications, consider using named exports more frequently for better maintainability and refactoring support
