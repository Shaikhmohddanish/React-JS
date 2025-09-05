# Handling Events in React

## Introduction to React Event Handling

Event handling is a fundamental aspect of creating interactive user interfaces in React. Events are actions or occurrences that happen in the browser, such as mouse clicks, form submissions, keyboard presses, and more. React provides a way to capture these events and respond to them by executing JavaScript code.

React's event system is designed to be:
- **Cross-browser compatible**: React normalizes events so they work consistently across different browsers
- **Performant**: React uses event delegation for better performance
- **Familiar**: React events closely follow standard DOM events, but with camelCase naming

In this guide, we'll explore how to handle events in React, from basic onClick handlers to more complex patterns.

## React Events vs DOM Events

While React's event system is based on the DOM event system, there are some important differences:

### 1. Naming Convention

React event names use camelCase instead of lowercase:

```jsx
// DOM event
<button onclick="handleClick()">Click me</button>

// React event
<button onClick={handleClick}>Click me</button>
```

### 2. Event Handlers

In HTML, you pass a string of code to an event handler:

```html
<button onclick="alert('Clicked!')">Click me</button>
```

In React, you pass a function reference:

```jsx
function MyComponent() {
  const handleClick = () => {
    alert('Clicked!');
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

### 3. Preventing Default Behavior

In HTML, you can return `false` to prevent default behavior:

```html
<a href="#" onclick="console.log('Clicked!'); return false;">Click me</a>
```

In React, you must explicitly call `preventDefault()`:

```jsx
function MyComponent() {
  const handleClick = (e) => {
    e.preventDefault(); // Prevents the link from navigating
    console.log('Clicked!');
  };
  
  return <a href="#" onClick={handleClick}>Click me</a>;
}
```

### 4. Synthetic Events

React wraps native DOM events in a cross-browser wrapper called SyntheticEvent to ensure consistent behavior across browsers:

```jsx
function MyComponent() {
  const handleClick = (e) => {
    console.log(e); // This is a SyntheticEvent object
    console.log(e.nativeEvent); // This is the browser's native event
    console.log(e.target); // The DOM element that triggered the event
    console.log(e.currentTarget); // The DOM element that the event listener is attached to
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

## Basic Event Handling

Let's explore how to handle various common events in React:

### Click Events

The most common event you'll handle is the click event:

```jsx
function ClickButton() {
  const handleClick = () => {
    console.log('Button clicked!');
  };
  
  return <button onClick={handleClick}>Click Me</button>;
}
```

### Mouse Events

React provides various mouse events:

```jsx
function MouseEvents() {
  const handleMouseEnter = () => {
    console.log('Mouse entered the element');
  };
  
  const handleMouseLeave = () => {
    console.log('Mouse left the element');
  };
  
  const handleMouseMove = (e) => {
    console.log(`Mouse position: ${e.clientX}, ${e.clientY}`);
  };
  
  return (
    <div 
      style={{ 
        width: '200px', 
        height: '200px', 
        backgroundColor: 'lightblue',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center'
      }}
      onMouseEnter={handleMouseEnter}
      onMouseLeave={handleMouseLeave}
      onMouseMove={handleMouseMove}
    >
      Hover over me
    </div>
  );
}
```

### Form Events

Working with forms requires handling input changes and form submissions:

```jsx
function SimpleForm() {
  const [name, setName] = React.useState('');
  
  const handleChange = (e) => {
    setName(e.target.value);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault(); // Prevent the browser from refreshing the page
    alert(`Hello, ${name}!`);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input 
          type="text" 
          value={name} 
          onChange={handleChange} 
          placeholder="Enter your name"
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Keyboard Events

You can handle keyboard events like keydown, keyup, and keypress:

```jsx
function KeyboardListener() {
  const [key, setKey] = React.useState('');
  
  const handleKeyDown = (e) => {
    setKey(e.key);
    console.log(`Key pressed: ${e.key}`);
    console.log(`Key code: ${e.keyCode}`);
    console.log(`Ctrl key pressed: ${e.ctrlKey}`);
    console.log(`Shift key pressed: ${e.shiftKey}`);
    console.log(`Alt key pressed: ${e.altKey}`);
  };
  
  return (
    <div>
      <p>Press any key. Current key: {key}</p>
      <input 
        type="text" 
        onKeyDown={handleKeyDown} 
        placeholder="Type something"
      />
    </div>
  );
}
```

### Focus Events

React provides focus and blur events for form elements:

```jsx
function FocusEvents() {
  const [focused, setFocused] = React.useState(false);
  
  const handleFocus = () => {
    setFocused(true);
    console.log('Input focused');
  };
  
  const handleBlur = () => {
    setFocused(false);
    console.log('Input blurred');
  };
  
  return (
    <div>
      <input 
        type="text" 
        onFocus={handleFocus} 
        onBlur={handleBlur}
        style={{ 
          border: focused ? '2px solid blue' : '1px solid gray',
          padding: '8px',
          outline: 'none'
        }}
        placeholder="Click me to focus"
      />
      <p>{focused ? 'Input is focused' : 'Input is not focused'}</p>
    </div>
  );
}
```

## Event Handler Patterns

There are several ways to define event handlers in React. Let's explore the most common patterns:

### 1. Method Defined in Function Component

The most straightforward approach is to define a function inside your component:

```jsx
function ClickCounter() {
  const [count, setCount] = React.useState(0);
  
  // Method defined in the component
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}
```

### 2. Inline Event Handlers

For simple operations, you can define handlers inline:

```jsx
function ClickCounter() {
  const [count, setCount] = React.useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

This approach is concise but can make the JSX harder to read if the handler is complex. It also creates a new function on each render, which might affect performance in some cases.

### 3. Class Component Methods

If you're working with class components, methods are defined as class methods and need to be bound to the class instance:

```jsx
class ClickCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    
    // Binding in constructor
    this.handleClick = this.handleClick.bind(this);
  }
  
  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}
```

Alternatively, you can use class properties and arrow functions to avoid binding:

```jsx
class ClickCounter extends React.Component {
  state = { count: 0 };
  
  // Arrow function for automatic binding
  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}
```

### 4. Event Handler as Prop

Components can receive event handlers as props:

```jsx
// Child component
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

// Parent component
function Counter() {
  const [count, setCount] = React.useState(0);
  
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <Button onClick={handleClick}>Increment</Button>
    </div>
  );
}
```

This pattern is useful for creating reusable components that can perform different actions based on the props they receive.

## Passing Arguments to Event Handlers

Sometimes you need to pass additional data to your event handlers. Here are different ways to achieve this:

### 1. Using Arrow Functions

The most common approach is to use an arrow function to wrap the handler call:

```jsx
function ItemList() {
  const items = ['Apple', 'Banana', 'Cherry', 'Date'];
  
  const handleItemClick = (item, index, e) => {
    console.log(`Clicked on ${item} at position ${index}`);
    console.log('Event:', e);
  };
  
  return (
    <ul>
      {items.map((item, index) => (
        <li 
          key={index} 
          onClick={(e) => handleItemClick(item, index, e)}
        >
          {item}
        </li>
      ))}
    </ul>
  );
}
```

### 2. Using bind

For class components, you can use `bind` to pass arguments:

```jsx
class ItemList extends React.Component {
  handleItemClick(item, index, e) {
    console.log(`Clicked on ${item} at position ${index}`);
    console.log('Event:', e);
  }
  
  render() {
    const items = ['Apple', 'Banana', 'Cherry', 'Date'];
    
    return (
      <ul>
        {items.map((item, index) => (
          <li 
            key={index} 
            onClick={this.handleItemClick.bind(this, item, index)}
          >
            {item}
          </li>
        ))}
      </ul>
    );
  }
}
```

The event object is automatically passed as the last argument when using `bind`.

### 3. Using Data Attributes

Another approach is to use data attributes and access them from the event:

```jsx
function ItemList() {
  const items = ['Apple', 'Banana', 'Cherry', 'Date'];
  
  const handleItemClick = (e) => {
    const item = e.target.getAttribute('data-item');
    const index = parseInt(e.target.getAttribute('data-index'));
    console.log(`Clicked on ${item} at position ${index}`);
  };
  
  return (
    <ul>
      {items.map((item, index) => (
        <li 
          key={index} 
          data-item={item}
          data-index={index}
          onClick={handleItemClick}
        >
          {item}
        </li>
      ))}
    </ul>
  );
}
```

## Synthetic Events in Detail

React's SyntheticEvent is a cross-browser wrapper around the browser's native event. It has the same interface as the browser's native event, with some additional features:

### Properties and Methods

SyntheticEvents include all standard event properties and methods:

```jsx
function EventDemo() {
  const handleClick = (e) => {
    // Common properties
    console.log('Target:', e.target); // The DOM element that triggered the event
    console.log('Type:', e.type); // The type of event (e.g., 'click')
    console.log('Timestamp:', e.timeStamp); // When the event occurred
    
    // Methods
    e.preventDefault(); // Prevents the default action
    e.stopPropagation(); // Stops the event from bubbling up
    
    // Getting the native event
    console.log('Native event:', e.nativeEvent);
    
    // Checking modifier keys
    console.log('Ctrl key:', e.ctrlKey);
    console.log('Shift key:', e.shiftKey);
    console.log('Alt key:', e.altKey);
    
    // For mouse events
    console.log('Mouse position:', e.clientX, e.clientY);
    
    // For keyboard events
    console.log('Key pressed:', e.key);
    console.log('Key code:', e.keyCode);
  };
  
  return <button onClick={handleClick}>Click Me</button>;
}
```

### Event Pooling (Historical Note)

In versions of React prior to 17, React used event pooling to improve performance. Event objects were reused, and all properties were nullified after the event callback was invoked. If you needed to access event properties asynchronously, you had to call `e.persist()`.

In React 17 and later, event pooling was removed, so you no longer need to call `e.persist()`.

```jsx
// React <17 (needed e.persist())
function EventExample() {
  const handleClick = (e) => {
    e.persist(); // Was necessary for async access
    setTimeout(() => {
      console.log(e.type); // 'click'
    }, 1000);
  };
  
  return <button onClick={handleClick}>Click Me</button>;
}

// React 17+ (e.persist() not needed)
function EventExample() {
  const handleClick = (e) => {
    setTimeout(() => {
      console.log(e.type); // 'click'
    }, 1000);
  };
  
  return <button onClick={handleClick}>Click Me</button>;
}
```

## Event Propagation and Bubbling

Like in standard DOM, React events also bubble up through the component hierarchy.

### Event Bubbling

Events in React bubble up from the target element to the root of the document:

```jsx
function BubblingDemo() {
  const handleDivClick = () => {
    console.log('Div clicked');
  };
  
  const handleButtonClick = (e) => {
    console.log('Button clicked');
    // The event will bubble up to the div without this line
    // e.stopPropagation();
  };
  
  return (
    <div 
      onClick={handleDivClick} 
      style={{ padding: '20px', backgroundColor: 'lightblue' }}
    >
      <p>Click the button or the surrounding div</p>
      <button onClick={handleButtonClick}>Click Me</button>
    </div>
  );
}
```

When you click the button, both `handleButtonClick` and `handleDivClick` will be called because the event bubbles up from the button to the div.

### Stopping Propagation

You can stop event bubbling with `stopPropagation()`:

```jsx
function StopPropagationDemo() {
  const handleDivClick = () => {
    console.log('Div clicked');
  };
  
  const handleButtonClick = (e) => {
    console.log('Button clicked');
    e.stopPropagation(); // Prevents the event from reaching the div
  };
  
  return (
    <div 
      onClick={handleDivClick} 
      style={{ padding: '20px', backgroundColor: 'lightblue' }}
    >
      <p>Click the button or the surrounding div</p>
      <button onClick={handleButtonClick}>Click Me</button>
    </div>
  );
}
```

Now when you click the button, only `handleButtonClick` will be called.

### Capture Phase

React also supports the capture phase, which happens before bubbling. To listen for events during the capture phase, add 'Capture' to the event name:

```jsx
function CapturePhaseDemo() {
  const handleDivCapturePhase = () => {
    console.log('Div - Capture Phase');
  };
  
  const handleDivBubblePhase = () => {
    console.log('Div - Bubble Phase');
  };
  
  const handleButtonCapturePhase = () => {
    console.log('Button - Capture Phase');
  };
  
  const handleButtonBubblePhase = () => {
    console.log('Button - Bubble Phase');
  };
  
  return (
    <div 
      onClickCapture={handleDivCapturePhase} 
      onClick={handleDivBubblePhase}
      style={{ padding: '20px', backgroundColor: 'lightblue' }}
    >
      <button 
        onClickCapture={handleButtonCapturePhase}
        onClick={handleButtonBubblePhase}
      >
        Click Me
      </button>
    </div>
  );
}
```

When you click the button, the events will fire in this order:
1. Div - Capture Phase
2. Button - Capture Phase
3. Button - Bubble Phase
4. Div - Bubble Phase

This matches how native DOM events work, with the capture phase going down the tree and the bubbling phase going up.

## Common Event Types

React supports all the standard DOM events. Here's a reference for some of the most commonly used ones:

### Mouse Events

- `onClick`: Triggered when an element is clicked
- `onDoubleClick`: Triggered when an element is double-clicked
- `onMouseDown`: Triggered when a mouse button is pressed down over an element
- `onMouseUp`: Triggered when a mouse button is released over an element
- `onMouseMove`: Triggered when the mouse moves over an element
- `onMouseEnter`: Triggered when the mouse enters an element (doesn't bubble)
- `onMouseLeave`: Triggered when the mouse leaves an element (doesn't bubble)
- `onMouseOver`: Triggered when the mouse moves onto an element (bubbles)
- `onMouseOut`: Triggered when the mouse moves out of an element (bubbles)

### Form Events

- `onChange`: Triggered when the value of an input element changes
- `onSubmit`: Triggered when a form is submitted
- `onFocus`: Triggered when an element receives focus
- `onBlur`: Triggered when an element loses focus
- `onInput`: Triggered when the value of an `<input>` or `<textarea>` changes
- `onReset`: Triggered when a form is reset
- `onSelect`: Triggered when text is selected in an input field

### Keyboard Events

- `onKeyDown`: Triggered when a key is pressed down
- `onKeyUp`: Triggered when a key is released
- `onKeyPress`: Triggered when a character is generated by a keypress (deprecated in favor of onKeyDown)

### Clipboard Events

- `onCopy`: Triggered when the user copies content
- `onCut`: Triggered when the user cuts content
- `onPaste`: Triggered when the user pastes content

### Touch Events

- `onTouchStart`: Triggered when a touch starts on an element
- `onTouchMove`: Triggered when a touch moves on an element
- `onTouchEnd`: Triggered when a touch ends on an element
- `onTouchCancel`: Triggered when a touch is cancelled

### Media Events

- `onPlay`: Triggered when media starts playing
- `onPause`: Triggered when media is paused
- `onEnded`: Triggered when media playback ends
- `onVolumeChange`: Triggered when the volume changes
- `onTimeUpdate`: Triggered when the playback position changes

### Animation Events

- `onAnimationStart`: Triggered when a CSS animation starts
- `onAnimationEnd`: Triggered when a CSS animation completes
- `onAnimationIteration`: Triggered when a CSS animation iteration completes
- `onTransitionEnd`: Triggered when a CSS transition completes

## Advanced Event Handling Patterns

### 1. Custom Event Handlers

You can create reusable custom event handlers:

```jsx
// Custom hook for form input handling
function useFormInput(initialValue) {
  const [value, setValue] = React.useState(initialValue);
  
  const handleChange = (e) => {
    setValue(e.target.value);
  };
  
  return {
    value,
    onChange: handleChange
  };
}

// Using the custom hook
function LoginForm() {
  const username = useFormInput('');
  const password = useFormInput('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', username.value, password.value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        <input type="text" {...username} />
      </div>
      <div>
        <label>Password:</label>
        <input type="password" {...password} />
      </div>
      <button type="submit">Login</button>
    </form>
  );
}
```

### 2. Debouncing and Throttling

For events that fire frequently (like scroll or resize), it's good practice to debounce or throttle event handlers:

```jsx
// Debouncing example (execute the latest call after a delay)
function SearchInput() {
  const [searchTerm, setSearchTerm] = React.useState('');
  const [debouncedTerm, setDebouncedTerm] = React.useState('');
  
  // Update debounced value after delay
  React.useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 500); // 500ms delay
    
    // Clean up on unmount or if searchTerm changes
    return () => {
      clearTimeout(timeoutId);
    };
  }, [searchTerm]);
  
  // Effect for API call
  React.useEffect(() => {
    if (debouncedTerm) {
      console.log('Making API call with:', debouncedTerm);
      // Make API call here
    }
  }, [debouncedTerm]);
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <p>Current search term: {searchTerm}</p>
      <p>Debounced term: {debouncedTerm}</p>
    </div>
  );
}
```

### 3. Event Delegation

React uses event delegation under the hood for performance, but you can leverage it explicitly in your code:

```jsx
function TodoList() {
  const [todos, setTodos] = React.useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build an app', completed: false },
    { id: 3, text: 'Deploy to production', completed: false }
  ]);
  
  // Using event delegation - we only need one event handler
  const handleClick = (e) => {
    if (e.target.tagName === 'LI') {
      const todoId = parseInt(e.target.dataset.id);
      
      setTodos(todos.map(todo => 
        todo.id === todoId 
          ? { ...todo, completed: !todo.completed } 
          : todo
      ));
    }
  };
  
  return (
    <ul onClick={handleClick}>
      {todos.map(todo => (
        <li 
          key={todo.id}
          data-id={todo.id}
          style={{ 
            textDecoration: todo.completed ? 'line-through' : 'none',
            cursor: 'pointer'
          }}
        >
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

### 4. Custom Events with React Portal

You can create custom event systems, especially useful with React Portal:

```jsx
function ModalSystem() {
  const [isOpen, setIsOpen] = React.useState(false);
  const [modalContent, setModalContent] = React.useState(null);
  
  // Event bus for modal communication
  const openModal = (content) => {
    setModalContent(content);
    setIsOpen(true);
  };
  
  const closeModal = () => {
    setIsOpen(false);
    // Clear content after animation completes
    setTimeout(() => setModalContent(null), 300);
  };
  
  return (
    <ModalContext.Provider value={{ openModal, closeModal }}>
      {/* Your app content */}
      <App />
      
      {/* Modal portal */}
      {isOpen && ReactDOM.createPortal(
        <div className="modal-overlay" onClick={closeModal}>
          <div 
            className="modal-content" 
            onClick={(e) => e.stopPropagation()}
          >
            {modalContent}
            <button onClick={closeModal}>Close</button>
          </div>
        </div>,
        document.getElementById('modal-root')
      )}
    </ModalContext.Provider>
  );
}
```

## Best Practices for Event Handling

### 1. Keep Event Handlers Simple

Event handlers should be focused on a single responsibility. Extract complex logic into separate functions:

```jsx
// Not ideal
function Form() {
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Too much happening in one handler
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData);
    
    // Validation
    if (!data.name) {
      alert('Name is required');
      return;
    }
    
    // API call
    fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data),
      headers: {
        'Content-Type': 'application/json'
      }
    })
      .then(response => response.json())
      .then(result => {
        console.log('Success:', result);
        e.target.reset();
      })
      .catch(error => {
        console.error('Error:', error);
      });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
}

// Better approach
function Form() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    const data = extractFormData(e.target);
    
    if (!validateFormData(data)) {
      return;
    }
    
    try {
      const result = await submitFormData(data);
      handleSubmitSuccess(result, e.target);
    } catch (error) {
      handleSubmitError(error);
    }
  };
  
  // Helper functions
  const extractFormData = (form) => {
    const formData = new FormData(form);
    return Object.fromEntries(formData);
  };
  
  const validateFormData = (data) => {
    if (!data.name) {
      alert('Name is required');
      return false;
    }
    return true;
  };
  
  const submitFormData = async (data) => {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data),
      headers: {
        'Content-Type': 'application/json'
      }
    });
    return response.json();
  };
  
  const handleSubmitSuccess = (result, form) => {
    console.log('Success:', result);
    form.reset();
  };
  
  const handleSubmitError = (error) => {
    console.error('Error:', error);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
}
```

### 2. Use Named Functions Instead of Anonymous Functions

Named functions make your code more readable and help with debugging:

```jsx
// Less ideal
function Counter() {
  const [count, setCount] = React.useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Better approach
function Counter() {
  const [count, setCount] = React.useState(0);
  
  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### 3. Avoid Inline Function Definitions for Performance-Critical Components

Creating functions inline in the render method creates a new function on each render, which can affect performance for frequently re-rendered components:

```jsx
// May cause performance issues in performance-critical components
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id} 
          onClick={() => handleItemClick(item.id)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// Better for performance-critical components
function ItemList({ items }) {
  // Use a Map to store handlers for each item
  const handlersMap = React.useMemo(() => {
    const map = new Map();
    
    items.forEach(item => {
      map.set(item.id, () => handleItemClick(item.id));
    });
    
    return map;
  }, [items]);
  
  const handleItemClick = (id) => {
    console.log(`Item ${id} clicked`);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id} 
          onClick={handlersMap.get(item.id)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

### 4. Clean Up Event Listeners

When adding DOM event listeners directly, remember to clean them up:

```jsx
function WindowResizeListener() {
  const [windowWidth, setWindowWidth] = React.useState(window.innerWidth);
  
  React.useEffect(() => {
    const handleResize = () => {
      setWindowWidth(window.innerWidth);
    };
    
    // Add event listener
    window.addEventListener('resize', handleResize);
    
    // Clean up event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return <div>Window width: {windowWidth}px</div>;
}
```

### 5. Use Event Pooling Wisely

For asynchronous operations that need event data, clone or extract the needed data:

```jsx
function AsyncEventHandler() {
  const handleClick = (e) => {
    // Extract needed data immediately
    const { target, clientX, clientY } = e;
    
    // Async operation
    setTimeout(() => {
      console.log('Target was:', target);
      console.log('Click position was:', clientX, clientY);
    }, 1000);
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

## Accessibility in Event Handling

Ensuring your event handling is accessible is crucial for users with disabilities:

### 1. Use Keyboard Events for Interactive Elements

Make sure interactive elements can be used with a keyboard:

```jsx
function AccessibleButton() {
  const [isActive, setIsActive] = React.useState(false);
  
  const handleClick = () => {
    setIsActive(!isActive);
  };
  
  const handleKeyDown = (e) => {
    // Activate on Enter or Space key
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault(); // Prevent scrolling on space key
      handleClick();
    }
  };
  
  return (
    <div 
      role="button"
      tabIndex={0} // Makes it focusable
      onClick={handleClick}
      onKeyDown={handleKeyDown}
      style={{
        padding: '10px',
        backgroundColor: isActive ? 'lightblue' : 'white',
        border: '1px solid gray',
        cursor: 'pointer'
      }}
    >
      {isActive ? 'Active' : 'Inactive'}
    </div>
  );
}
```

### 2. Use Appropriate ARIA Attributes

When creating custom interactive components, use ARIA attributes to communicate state:

```jsx
function AccessibleToggle() {
  const [isOn, setIsOn] = React.useState(false);
  
  const toggle = () => {
    setIsOn(!isOn);
  };
  
  return (
    <div>
      <button 
        onClick={toggle}
        aria-pressed={isOn}
        aria-label="Toggle feature"
      >
        {isOn ? 'ON' : 'OFF'}
      </button>
      <p>Feature is {isOn ? 'enabled' : 'disabled'}</p>
    </div>
  );
}
```

### 3. Announce Dynamic Changes

For dynamic content changes, consider using ARIA live regions:

```jsx
function NotificationSystem() {
  const [notifications, setNotifications] = React.useState([]);
  
  const addNotification = (message) => {
    const newNotification = { 
      id: Date.now(), 
      message, 
      time: new Date().toLocaleTimeString() 
    };
    
    setNotifications([...notifications, newNotification]);
    
    // Auto-remove after 5 seconds
    setTimeout(() => {
      setNotifications(current => 
        current.filter(n => n.id !== newNotification.id)
      );
    }, 5000);
  };
  
  return (
    <div>
      <button onClick={() => addNotification('Something happened!')}>
        Trigger Notification
      </button>
      
      {/* Visually hidden live region for screen readers */}
      <div 
        className="sr-only" 
        aria-live="polite"
        aria-atomic="true"
      >
        {notifications.length > 0 && 
          `New notification: ${notifications[notifications.length - 1].message}`
        }
      </div>
      
      {/* Visual notifications */}
      <div className="notifications-container">
        {notifications.map(notification => (
          <div key={notification.id} className="notification">
            <p>{notification.message}</p>
            <small>{notification.time}</small>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Summary

Event handling is a core aspect of building interactive React applications. By understanding React's event system, you can create responsive and user-friendly interfaces.

Key takeaways:
- React events use camelCase naming (onClick, onChange)
- Event handlers receive a SyntheticEvent object
- Use preventDefault() to prevent default browser behavior
- Events bubble up through the component hierarchy
- Keep event handlers simple and focused
- Clean up event listeners when components unmount
- Consider accessibility when implementing event handling

By following these patterns and best practices, you'll be able to build React applications with robust and maintainable event handling.

## Practice Exercise

Build a simple form with the following features:

1. Input fields for name, email, and message
2. Form validation that checks:
   - Name is not empty
   - Email is valid (contains @ and .)
   - Message is at least 10 characters long
3. Display validation errors next to each field
4. Submit button that's disabled until all validation passes
5. Success message when the form is successfully submitted
6. Make sure the form is keyboard accessible

This exercise will help you practice various event handling techniques in a realistic scenario.

## Additional Resources

- [React Documentation on Events](https://react.dev/reference/react-dom/components/common#react-event-object)
- [MDN Web Docs: Introduction to events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)
- [WAI-ARIA Authoring Practices](https://www.w3.org/TR/wai-aria-practices-1.1/) for accessibility guidelines
- [JavaScript.info: Browser Events](https://javascript.info/event-details) for detailed information on native browser events
