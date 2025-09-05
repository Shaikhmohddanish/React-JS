# Styling in React - A Complete Guide

## Introduction to Styling in React

Styling is a crucial aspect of any React application that directly impacts user experience and interface aesthetics. Unlike traditional web development where styles are completely separated from the markup, React enables several approaches to styling that each offer unique advantages. This guide will cover all major styling techniques in React to help you choose the right approach for your projects.

## CSS Approaches in React

React supports several styling paradigms, each with its own strengths and use cases:

1. **Regular CSS Files**: Traditional approach with external stylesheets
2. **Inline Styles**: Styling directly in JSX using JavaScript objects
3. **CSS Modules**: Locally scoped CSS files
4. **CSS-in-JS Libraries**: JavaScript-based styling solutions (styled-components, Emotion)
5. **Utility-First CSS**: Using libraries like Tailwind CSS
6. **Sass/SCSS Integration**: Using preprocessors for enhanced CSS capabilities
7. **CSS Custom Properties**: Using CSS variables with React
8. **CSS Frameworks**: Integration with Bootstrap, Material-UI, etc.

Let's explore each approach in detail:

## 1. Regular CSS Files

The simplest approach is to use traditional CSS files imported directly into your React components.

### Example:

**styles.css**
```css
.button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.button:hover {
  background-color: #45a049;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}
```

**App.js**
```jsx
import React from 'react';
import './styles.css';

function App() {
  return (
    <div className="container">
      <header className="header">
        <h1>My React App</h1>
        <button className="button">Click Me</button>
      </header>
      <main>
        <p>This is styled using regular CSS imports.</p>
      </main>
    </div>
  );
}

export default App;
```

### Pros:
- Familiar to most developers
- No additional libraries required
- Good for smaller applications
- Separation of concerns (CSS and JS)

### Cons:
- Global namespace (potential naming conflicts)
- No direct connection between components and styles
- CSS specificity issues in larger applications
- No built-in way to conditionally apply styles based on props

## 2. Inline Styles

React allows you to define styles directly in your JSX using JavaScript objects.

### Example:

```jsx
import React from 'react';

function Button({ primary, children }) {
  const buttonStyle = {
    backgroundColor: primary ? '#4CAF50' : '#f8f9fa',
    color: primary ? 'white' : '#212529',
    padding: '10px 15px',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    fontSize: '16px',
  };

  return (
    <button style={buttonStyle}>
      {children}
    </button>
  );
}

function App() {
  const containerStyle = {
    maxWidth: '1200px',
    margin: '0 auto',
    padding: '20px',
  };

  const headerStyle = {
    display: 'flex',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: '10px 0',
    borderBottom: '1px solid #eee',
  };

  return (
    <div style={containerStyle}>
      <header style={headerStyle}>
        <h1>My React App</h1>
        <Button primary>Primary Button</Button>
        <Button>Secondary Button</Button>
      </header>
      <main>
        <p>This is styled using inline styles.</p>
      </main>
    </div>
  );
}

export default App;
```

### Pros:
- No additional setup required
- Styles are scoped to the component
- Can easily use JavaScript variables and props
- No CSS class name conflicts

### Cons:
- No support for CSS pseudo-classes (:hover, :focus, etc.) without extra libraries
- No support for media queries without extra libraries
- No CSS reuse across components
- Can make JSX harder to read with complex styles
- Performance overhead in some cases

## 3. CSS Modules

CSS Modules allow you to write traditional CSS files but with local scoping by default, preventing style leakage and naming conflicts.

### Example:

**Button.module.css**
```css
.button {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.primary {
  background-color: #4CAF50;
  color: white;
}

.primary:hover {
  background-color: #45a049;
}

.secondary {
  background-color: #f8f9fa;
  color: #212529;
}

.secondary:hover {
  background-color: #e2e6ea;
}
```

**Button.js**
```jsx
import React from 'react';
import styles from './Button.module.css';

function Button({ primary, children }) {
  const buttonClass = primary ? styles.primary : styles.secondary;
  
  return (
    <button className={`${styles.button} ${buttonClass}`}>
      {children}
    </button>
  );
}

export default Button;
```

**App.module.css**
```css
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}
```

**App.js**
```jsx
import React from 'react';
import styles from './App.module.css';
import Button from './Button';

function App() {
  return (
    <div className={styles.container}>
      <header className={styles.header}>
        <h1>My React App</h1>
        <div>
          <Button primary>Primary Button</Button>
          <Button>Secondary Button</Button>
        </div>
      </header>
      <main>
        <p>This is styled using CSS Modules.</p>
      </main>
    </div>
  );
}

export default App;
```

### Pros:
- Local scope by default (prevents naming conflicts)
- Full CSS feature support (media queries, animations, pseudo-selectors)
- Clear connection between component and its styles
- Supported by Create React App out of the box
- TypeScript support with typed CSS modules

### Cons:
- Requires build configuration (though included in Create React App)
- Dynamic styling can be more verbose
- Still requires managing CSS files separately

## 4. CSS-in-JS Libraries

CSS-in-JS libraries allow you to write CSS directly in your JavaScript files, often with component-based APIs.

### Example using styled-components:

First, install the library:
```bash
npm install styled-components
```

Then use it in your components:

```jsx
import React from 'react';
import styled, { ThemeProvider } from 'styled-components';

// Define the styled components
const Button = styled.button`
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
  
  /* Use props to conditionally apply styles */
  background-color: ${props => props.primary ? props.theme.primaryColor : '#f8f9fa'};
  color: ${props => props.primary ? 'white' : '#212529'};
  
  &:hover {
    background-color: ${props => props.primary ? props.theme.primaryColorDark : '#e2e6ea'};
  }
  
  /* Apply styles based on other props */
  ${props => props.large && `
    font-size: 18px;
    padding: 12px 20px;
  `}
  
  /* Media queries work too */
  @media (max-width: 768px) {
    font-size: 14px;
    padding: 8px 12px;
  }
`;

const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
`;

const Header = styled.header`
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
`;

// Define a theme
const theme = {
  primaryColor: '#4CAF50',
  primaryColorDark: '#45a049',
  secondaryColor: '#f8f9fa',
  textColor: '#212529'
};

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Container>
        <Header>
          <h1>My React App</h1>
          <div>
            <Button primary>Primary Button</Button>
            <Button>Secondary Button</Button>
            <Button primary large>Large Primary Button</Button>
          </div>
        </Header>
        <main>
          <p>This is styled using styled-components.</p>
        </main>
      </Container>
    </ThemeProvider>
  );
}

export default App;
```

### Pros:
- Fully scoped styles (no class name conflicts)
- Dynamic styling based on props
- Automatic vendor prefixing
- Theming support
- No separate CSS files to manage
- Can leverage full JavaScript capabilities
- Dead code elimination (unused styles are not included)
- Server-side rendering support

### Cons:
- Additional runtime library dependency
- Learning curve for CSS-in-JS concepts
- Potential performance overhead
- Debugging can be more difficult (generated class names)
- Different from traditional CSS workflows

### Example using Emotion:

Emotion is another popular CSS-in-JS library with similar capabilities to styled-components.

First, install the library:
```bash
npm install @emotion/react @emotion/styled
```

Then use it in your components:

```jsx
/** @jsxImportSource @emotion/react */
import { css, ThemeProvider } from '@emotion/react';
import styled from '@emotion/styled';

// Using the css prop
const buttonStyles = ({ primary, theme }) => css`
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  background-color: ${primary ? theme.colors.primary : '#f8f9fa'};
  color: ${primary ? 'white' : '#212529'};
  
  &:hover {
    background-color: ${primary ? theme.colors.primaryDark : '#e2e6ea'};
  }
`;

// Using styled API (similar to styled-components)
const Container = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
`;

const Header = styled.header`
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
`;

const theme = {
  colors: {
    primary: '#4CAF50',
    primaryDark: '#45a049',
    secondary: '#f8f9fa',
    text: '#212529'
  }
};

function Button({ primary, children }) {
  return <button css={buttonStyles({ primary, theme })}>{children}</button>;
}

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Container>
        <Header>
          <h1>My React App</h1>
          <div>
            <Button primary>Primary Button</Button>
            <Button>Secondary Button</Button>
          </div>
        </Header>
        <main>
          <p>This is styled using Emotion.</p>
        </main>
      </Container>
    </ThemeProvider>
  );
}

export default App;
```

## 5. Utility-First CSS with Tailwind CSS

Tailwind CSS provides low-level utility classes that let you build custom designs without writing CSS.

First, install Tailwind CSS:
```bash
npm install tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Configure it in your project, then use it in your components:

```jsx
import React from 'react';
import 'tailwindcss/tailwind.css';

function Button({ primary, children }) {
  const buttonClasses = primary
    ? 'bg-green-500 hover:bg-green-600 text-white'
    : 'bg-gray-100 hover:bg-gray-200 text-gray-800';
    
  return (
    <button className={`px-4 py-2 rounded font-medium focus:outline-none focus:ring-2 ${buttonClasses}`}>
      {children}
    </button>
  );
}

function App() {
  return (
    <div className="max-w-6xl mx-auto p-6">
      <header className="flex justify-between items-center py-4 border-b border-gray-200">
        <h1 className="text-2xl font-bold">My React App</h1>
        <div className="space-x-2">
          <Button primary>Primary Button</Button>
          <Button>Secondary Button</Button>
        </div>
      </header>
      <main className="py-6">
        <p className="text-gray-700">This is styled using Tailwind CSS.</p>
      </main>
    </div>
  );
}

export default App;
```

### Pros:
- No need to write custom CSS or create new CSS files
- Consistent design system with predefined values
- Highly customizable through configuration
- Responsive design utilities built-in
- Fast development speed once you learn the classes
- Small production CSS file size with PurgeCSS integration

### Cons:
- Steep learning curve for class names
- Can lead to lengthy class strings in JSX
- HTML becomes more verbose
- Can be challenging to read and maintain complex components
- Requires build process configuration

## 6. Sass/SCSS Integration

Sass is a CSS preprocessor that adds features like variables, nesting, and mixins.

First, install Sass:
```bash
npm install sass
```

Then create and use SCSS files:

**Button.scss**
```scss
// Variables
$primary-color: #4CAF50;
$primary-dark: #45a049;
$secondary-color: #f8f9fa;
$secondary-dark: #e2e6ea;
$border-radius: 4px;

// Mixins
@mixin button-base {
  padding: 10px 15px;
  border: none;
  border-radius: $border-radius;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

// Classes
.button {
  @include button-base;
  
  &.primary {
    background-color: $primary-color;
    color: white;
    
    &:hover {
      background-color: $primary-dark;
    }
  }
  
  &.secondary {
    background-color: $secondary-color;
    color: #212529;
    
    &:hover {
      background-color: $secondary-dark;
    }
  }
  
  &.large {
    font-size: 18px;
    padding: 12px 20px;
  }
  
  // Media queries
  @media (max-width: 768px) {
    font-size: 14px;
    padding: 8px 12px;
  }
}
```

**App.scss**
```scss
@import './variables';

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}
```

**Button.js**
```jsx
import React from 'react';
import './Button.scss';

function Button({ primary, large, children }) {
  const buttonClass = primary ? 'primary' : 'secondary';
  const sizeClass = large ? 'large' : '';
  
  return (
    <button className={`button ${buttonClass} ${sizeClass}`}>
      {children}
    </button>
  );
}

export default Button;
```

**App.js**
```jsx
import React from 'react';
import './App.scss';
import Button from './Button';

function App() {
  return (
    <div className="container">
      <header className="header">
        <h1>My React App</h1>
        <div>
          <Button primary>Primary Button</Button>
          <Button>Secondary Button</Button>
          <Button primary large>Large Primary Button</Button>
        </div>
      </header>
      <main>
        <p>This is styled using Sass/SCSS.</p>
      </main>
    </div>
  );
}

export default App;
```

### Pros:
- Enhanced CSS features (variables, nesting, mixins, functions)
- Improved maintainability with partials and imports
- Familiar CSS syntax with additional capabilities
- Good for large applications with many components
- Well-established in the industry

### Cons:
- Requires preprocessing
- Global namespace (unless combined with CSS Modules)
- Additional build setup (though integrated in Create React App)
- Still separate files from components

## 7. CSS Custom Properties (Variables) with React

Modern CSS supports custom properties (variables) that can be dynamically updated with JavaScript.

```jsx
import React, { useState } from 'react';
import './styles.css';

function ThemeToggler() {
  const [isDarkMode, setIsDarkMode] = useState(false);
  
  const toggleTheme = () => {
    const newDarkMode = !isDarkMode;
    setIsDarkMode(newDarkMode);
    
    // Update CSS variables at the document root
    const root = document.documentElement;
    if (newDarkMode) {
      root.style.setProperty('--bg-color', '#121212');
      root.style.setProperty('--text-color', '#ffffff');
      root.style.setProperty('--primary-color', '#90caf9');
    } else {
      root.style.setProperty('--bg-color', '#ffffff');
      root.style.setProperty('--text-color', '#121212');
      root.style.setProperty('--primary-color', '#1976d2');
    }
  };
  
  return (
    <button onClick={toggleTheme}>
      Switch to {isDarkMode ? 'Light' : 'Dark'} Mode
    </button>
  );
}

function App() {
  return (
    <div className="container">
      <header className="header">
        <h1>My React App</h1>
        <ThemeToggler />
      </header>
      <main>
        <p>This uses CSS custom properties (variables).</p>
        <button className="button">Themed Button</button>
      </main>
    </div>
  );
}

export default App;
```

**styles.css**
```css
:root {
  --bg-color: #ffffff;
  --text-color: #121212;
  --primary-color: #1976d2;
  --secondary-color: #f8f9fa;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
  transition: background-color 0.3s, color 0.3s;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.button {
  background-color: var(--primary-color);
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.button:hover {
  filter: brightness(110%);
}
```

### Pros:
- Native CSS feature (no libraries required)
- Ability to update styles dynamically with JavaScript
- Can be used for theming
- Scoped by CSS cascade (can set variables at different levels)

### Cons:
- Not supported in older browsers (IE11)
- Less integrated with React's component model
- Limited to string values

## 8. CSS Frameworks with React

### Material-UI Example

Material-UI is a popular React UI framework implementing Google's Material Design.

First, install Material-UI:
```bash
npm install @mui/material @mui/icons-material @emotion/react @emotion/styled
```

Then use the components:

```jsx
import React, { useState } from 'react';
import { 
  ThemeProvider, 
  createTheme, 
  CssBaseline,
  AppBar,
  Toolbar,
  Typography,
  Button,
  Container,
  Switch,
  Box,
  Card,
  CardContent,
  CardActions
} from '@mui/material';

function App() {
  const [darkMode, setDarkMode] = useState(false);
  
  const theme = createTheme({
    palette: {
      mode: darkMode ? 'dark' : 'light',
      primary: {
        main: '#1976d2',
      },
      secondary: {
        main: '#f50057',
      },
    },
  });
  
  const handleThemeChange = () => {
    setDarkMode(!darkMode);
  };
  
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <AppBar position="static">
        <Toolbar>
          <Typography variant="h6" component="div" sx={{ flexGrow: 1 }}>
            Material-UI Demo
          </Typography>
          <Box display="flex" alignItems="center">
            <Typography variant="body2">
              {darkMode ? 'Dark' : 'Light'} Mode
            </Typography>
            <Switch
              checked={darkMode}
              onChange={handleThemeChange}
              color="default"
            />
          </Box>
        </Toolbar>
      </AppBar>
      
      <Container maxWidth="sm" sx={{ mt: 4 }}>
        <Typography variant="h4" component="h1" gutterBottom>
          Welcome to My React App
        </Typography>
        
        <Card sx={{ mb: 3 }}>
          <CardContent>
            <Typography variant="h5" component="div">
              Material-UI Components
            </Typography>
            <Typography variant="body2" color="text.secondary" sx={{ mt: 1 }}>
              This example demonstrates using a CSS framework (Material-UI) with React.
              Material-UI provides pre-designed components following Material Design principles.
            </Typography>
          </CardContent>
          <CardActions>
            <Button size="small" color="primary">Learn More</Button>
          </CardActions>
        </Card>
        
        <Box sx={{ display: 'flex', gap: 2 }}>
          <Button variant="contained" color="primary">
            Primary Button
          </Button>
          <Button variant="contained" color="secondary">
            Secondary Button
          </Button>
          <Button variant="outlined" color="primary">
            Outlined Button
          </Button>
        </Box>
      </Container>
    </ThemeProvider>
  );
}

export default App;
```

### Bootstrap with React Example

Bootstrap is another popular framework that can be used with React.

First, install React Bootstrap:
```bash
npm install react-bootstrap bootstrap
```

Then import Bootstrap CSS and use the components:

```jsx
import React from 'react';
import { 
  Container, 
  Navbar, 
  Nav, 
  Button, 
  Card, 
  Row, 
  Col 
} from 'react-bootstrap';
import 'bootstrap/dist/css/bootstrap.min.css';

function App() {
  return (
    <>
      <Navbar bg="dark" variant="dark" expand="lg">
        <Container>
          <Navbar.Brand href="#home">React-Bootstrap</Navbar.Brand>
          <Navbar.Toggle aria-controls="basic-navbar-nav" />
          <Navbar.Collapse id="basic-navbar-nav">
            <Nav className="me-auto">
              <Nav.Link href="#home">Home</Nav.Link>
              <Nav.Link href="#features">Features</Nav.Link>
              <Nav.Link href="#pricing">Pricing</Nav.Link>
            </Nav>
          </Navbar.Collapse>
        </Container>
      </Navbar>
      
      <Container className="py-4">
        <h1 className="mb-4">Bootstrap with React</h1>
        
        <Row className="mb-4">
          <Col md={4} className="mb-3 mb-md-0">
            <Card>
              <Card.Img variant="top" src="https://via.placeholder.com/300x150" />
              <Card.Body>
                <Card.Title>Card Title 1</Card.Title>
                <Card.Text>
                  This example shows how to use Bootstrap components with React.
                </Card.Text>
                <Button variant="primary">Learn More</Button>
              </Card.Body>
            </Card>
          </Col>
          
          <Col md={4} className="mb-3 mb-md-0">
            <Card>
              <Card.Img variant="top" src="https://via.placeholder.com/300x150" />
              <Card.Body>
                <Card.Title>Card Title 2</Card.Title>
                <Card.Text>
                  React-Bootstrap replaces the Bootstrap JavaScript with React components.
                </Card.Text>
                <Button variant="primary">Learn More</Button>
              </Card.Body>
            </Card>
          </Col>
          
          <Col md={4}>
            <Card>
              <Card.Img variant="top" src="https://via.placeholder.com/300x150" />
              <Card.Body>
                <Card.Title>Card Title 3</Card.Title>
                <Card.Text>
                  This approach gives you more control over the component behavior.
                </Card.Text>
                <Button variant="primary">Learn More</Button>
              </Card.Body>
            </Card>
          </Col>
        </Row>
        
        <div className="d-grid gap-2 d-md-flex justify-content-md-center">
          <Button variant="primary" size="lg">Primary</Button>
          <Button variant="secondary" size="lg">Secondary</Button>
          <Button variant="success" size="lg">Success</Button>
          <Button variant="warning" size="lg">Warning</Button>
        </div>
      </Container>
    </>
  );
}

export default App;
```

### Pros of CSS Frameworks:
- Ready-to-use, professionally designed components
- Consistent UI across the application
- Built-in accessibility features
- Documentation and community support
- Often include responsive design
- Reduce development time

### Cons of CSS Frameworks:
- Larger bundle size
- Less customizable (without overriding styles)
- Similar look and feel across different applications
- Learning curve for framework-specific APIs
- Sometimes harder to implement unique designs

## Styling Organization Best Practices

### Component-Based Organization

For any styling approach, organize styles by component:

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.js
│   │   ├── Button.module.css (or Button.scss, or Button.styles.js)
│   │   └── index.js
│   ├── Card/
│   │   ├── Card.js
│   │   ├── Card.module.css
│   │   └── index.js
│   └── ...
```

### Theme Management

Create a centralized theme for consistent styling:

```jsx
// theme.js (for CSS-in-JS)
export const theme = {
  colors: {
    primary: '#1976d2',
    secondary: '#f50057',
    success: '#4caf50',
    error: '#f44336',
    warning: '#ff9800',
    info: '#2196f3',
    background: '#ffffff',
    text: '#121212',
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px',
  },
  typography: {
    fontFamily: 'Roboto, sans-serif',
    fontSize: {
      small: '0.875rem',
      medium: '1rem',
      large: '1.25rem',
      h1: '2.5rem',
      h2: '2rem',
      h3: '1.75rem',
    },
  },
  breakpoints: {
    xs: '0px',
    sm: '600px',
    md: '960px',
    lg: '1280px',
    xl: '1920px',
  },
  shadows: {
    small: '0 2px 4px rgba(0,0,0,0.1)',
    medium: '0 4px 8px rgba(0,0,0,0.1)',
    large: '0 8px 16px rgba(0,0,0,0.1)',
  },
};
```

Or with SCSS:

```scss
// _variables.scss
$colors: (
  primary: #1976d2,
  secondary: #f50057,
  success: #4caf50,
  error: #f44336,
  warning: #ff9800,
  info: #2196f3,
  background: #ffffff,
  text: #121212,
);

$spacing: (
  xs: 4px,
  sm: 8px,
  md: 16px,
  lg: 24px,
  xl: 32px,
);

$font-sizes: (
  small: 0.875rem,
  medium: 1rem,
  large: 1.25rem,
  h1: 2.5rem,
  h2: 2rem,
  h3: 1.75rem,
);

$breakpoints: (
  xs: 0px,
  sm: 600px,
  md: 960px,
  lg: 1280px,
  xl: 1920px,
);

// Usage in other files
@import 'variables';

.button {
  background-color: map-get($colors, primary);
  padding: map-get($spacing, md);
  font-size: map-get($font-sizes, medium);
  
  @media (min-width: map-get($breakpoints, md)) {
    padding: map-get($spacing, lg);
  }
}
```

## Responsive Design in React

### Media Queries in CSS/SCSS

```scss
.container {
  padding: 15px;
  
  @media (min-width: 768px) {
    padding: 30px;
  }
  
  @media (min-width: 1024px) {
    padding: 40px;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

### Responsive with CSS-in-JS

```jsx
import styled from 'styled-components';

const Container = styled.div`
  padding: 15px;
  
  @media (min-width: 768px) {
    padding: 30px;
  }
  
  @media (min-width: 1024px) {
    padding: 40px;
    max-width: 1200px;
    margin: 0 auto;
  }
`;
```

### Responsive Components with React Hooks

```jsx
import React, { useState, useEffect } from 'react';

function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });
  
  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    
    window.addEventListener('resize', handleResize);
    handleResize(); // Initial size
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}

function ResponsiveComponent() {
  const { width } = useWindowSize();
  
  // Different layouts based on screen size
  if (width < 768) {
    return <MobileLayout />;
  } else if (width < 1024) {
    return <TabletLayout />;
  } else {
    return <DesktopLayout />;
  }
}
```

## Animations in React

### CSS Animations

```css
.fade-in {
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.button {
  transition: transform 0.2s, background-color 0.3s;
}

.button:hover {
  transform: scale(1.05);
  background-color: #0056b3;
}
```

### React Transition Group

```jsx
import React, { useState } from 'react';
import { CSSTransition, TransitionGroup } from 'react-transition-group';
import './animations.css';

function AnimatedList() {
  const [items, setItems] = useState([
    { id: 1, text: 'Item 1' },
    { id: 2, text: 'Item 2' },
    { id: 3, text: 'Item 3' },
  ]);
  
  const [nextId, setNextId] = useState(4);
  
  const addItem = () => {
    setItems([...items, { id: nextId, text: `Item ${nextId}` }]);
    setNextId(nextId + 1);
  };
  
  const removeItem = (id) => {
    setItems(items.filter(item => item.id !== id));
  };
  
  return (
    <div>
      <button onClick={addItem}>Add Item</button>
      
      <TransitionGroup className="item-list">
        {items.map(item => (
          <CSSTransition
            key={item.id}
            timeout={500}
            classNames="item"
          >
            <div className="item">
              <span>{item.text}</span>
              <button onClick={() => removeItem(item.id)}>Remove</button>
            </div>
          </CSSTransition>
        ))}
      </TransitionGroup>
    </div>
  );
}
```

**animations.css**
```css
.item-list {
  list-style: none;
  padding: 0;
}

.item {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f9f9f9;
}

/* Enter animations */
.item-enter {
  opacity: 0;
  transform: translateX(-30px);
}

.item-enter-active {
  opacity: 1;
  transform: translateX(0);
  transition: opacity 500ms, transform 500ms;
}

/* Exit animations */
.item-exit {
  opacity: 1;
}

.item-exit-active {
  opacity: 0;
  transform: translateX(30px);
  transition: opacity 500ms, transform 500ms;
}
```

### Framer Motion

```jsx
import React, { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

function MotionDemo() {
  const [isVisible, setIsVisible] = useState(true);
  
  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        {isVisible ? 'Hide' : 'Show'}
      </button>
      
      <AnimatePresence>
        {isVisible && (
          <motion.div
            initial={{ opacity: 0, y: -20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: 20 }}
            transition={{ duration: 0.5 }}
            style={{
              padding: '20px',
              margin: '20px 0',
              backgroundColor: '#f0f0f0',
              borderRadius: '8px',
            }}
          >
            <h3>Animated Content</h3>
            <p>This content animates in and out smoothly using Framer Motion.</p>
          </motion.div>
        )}
      </AnimatePresence>
      
      <motion.button
        whileHover={{ scale: 1.1 }}
        whileTap={{ scale: 0.95 }}
        style={{
          backgroundColor: '#1976d2',
          color: 'white',
          border: 'none',
          padding: '10px 20px',
          borderRadius: '4px',
          cursor: 'pointer',
        }}
      >
        Interactive Button
      </motion.button>
    </div>
  );
}
```

## Accessibility Considerations for Styling

### High Contrast and Color Blindness

```jsx
// Ensure sufficient color contrast
const Button = styled.button`
  background-color: #1976d2; // Good contrast with white text
  color: white;
  
  // Provide additional visual cues besides color
  &:disabled {
    background-color: #cccccc;
    color: #666666;
    cursor: not-allowed;
    opacity: 0.7; // Visual cue beyond just color
    text-decoration: line-through; // Additional visual cue
  }
`;
```

### Focus Styles

```css
/* Don't remove focus outlines without providing alternatives */
.button:focus {
  outline: 2px solid #1976d2;
  outline-offset: 2px;
}

/* Only hide outline for mouse users, keep for keyboard navigation */
.button:focus:not(:focus-visible) {
  outline: none;
}

.button:focus-visible {
  outline: 2px solid #1976d2;
  outline-offset: 2px;
}
```

### Reduced Motion

```css
/* Respect user preferences for reduced motion */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

```jsx
// In React with CSS-in-JS
import { useState, useEffect } from 'react';

function usePrefersReducedMotion() {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);
  
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);
    
    const handleChange = () => setPrefersReducedMotion(mediaQuery.matches);
    mediaQuery.addEventListener('change', handleChange);
    
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);
  
  return prefersReducedMotion;
}

function AnimatedComponent() {
  const prefersReducedMotion = usePrefersReducedMotion();
  
  const animationProps = prefersReducedMotion
    ? { initial: {}, animate: {}, exit: {} } // No animation
    : {
        initial: { opacity: 0, y: -20 },
        animate: { opacity: 1, y: 0 },
        exit: { opacity: 0, y: 20 },
        transition: { duration: 0.5 }
      };
  
  return (
    <motion.div {...animationProps}>
      Content that may or may not animate
    </motion.div>
  );
}
```

## Performance Optimization for Styling

### Dynamic Style Optimization

```jsx
// Bad: Recreates the styles object on every render
function Button({ primary, children }) {
  const buttonStyle = {
    backgroundColor: primary ? '#4CAF50' : '#f8f9fa',
    color: primary ? 'white' : '#212529',
    padding: '10px 15px',
    border: 'none',
    borderRadius: '4px',
  };
  
  return <button style={buttonStyle}>{children}</button>;
}

// Better: Memoize the styles object
function Button({ primary, children }) {
  const buttonStyle = useMemo(() => ({
    backgroundColor: primary ? '#4CAF50' : '#f8f9fa',
    color: primary ? 'white' : '#212529',
    padding: '10px 15px',
    border: 'none',
    borderRadius: '4px',
  }), [primary]);
  
  return <button style={buttonStyle}>{children}</button>;
}

// Best: Use CSS classes instead of inline styles for static parts
function Button({ primary, children }) {
  return (
    <button 
      className={`button ${primary ? 'button-primary' : 'button-secondary'}`}
    >
      {children}
    </button>
  );
}
```

### Bundle Size Optimization

1. Use CSS Modules or CSS-in-JS with code splitting
2. Consider importing only the CSS you need from libraries:

```jsx
// Bad: Imports the entire library
import 'bootstrap/dist/css/bootstrap.min.css';

// Better: Import only what you need
import 'bootstrap/dist/css/bootstrap-grid.min.css';
import 'bootstrap/dist/css/bootstrap-utilities.min.css';
```

3. For CSS-in-JS libraries, use their production-optimized versions

## Choosing the Right Styling Approach

### Decision Factors

1. **Team Experience**: What is your team most familiar with?
2. **Project Size**: Small project vs. large enterprise application
3. **Design Requirements**: Custom design vs. standard UI
4. **Performance Needs**: Is optimal performance critical?
5. **Browser Support**: Do you need to support older browsers?
6. **Development Speed**: How quickly do you need to iterate?

### Recommendations

- **Small Project or Quick Prototype**: CSS Modules or Tailwind CSS
- **Large Application**: CSS-in-JS or CSS Modules with a design system
- **Design System Implementation**: Styled-components or Emotion
- **Team with CSS Expertise**: SCSS + CSS Modules
- **Need for Rapid Development**: UI Framework (Material-UI, Chakra UI)
- **Maximum Performance**: CSS Modules or vanilla CSS with minimal JavaScript

## Summary

React offers multiple styling approaches, each with its own advantages:

1. **Regular CSS**: Simple, familiar, but global scope
2. **Inline Styles**: Component-scoped, but limited features
3. **CSS Modules**: Scoped CSS with full CSS features
4. **CSS-in-JS**: JavaScript-powered styling with component integration
5. **Utility-First CSS**: Rapid development with predefined classes
6. **Sass/SCSS**: Enhanced CSS features with preprocessing
7. **CSS Custom Properties**: Dynamic styling with native CSS variables
8. **CSS Frameworks**: Pre-designed components and styling systems

The best approach depends on your project needs, team expertise, and specific requirements. Many projects use a combination of these approaches for different parts of the application.

Remember to prioritize:
- Component-based organization
- Consistent theming
- Responsive design
- Accessibility
- Performance optimization

## Practice Exercise

Create a simple React application that demonstrates at least three different styling approaches:

1. Create a button component using CSS Modules
2. Create a card component using styled-components or Emotion
3. Create a navigation bar using a CSS framework or Tailwind CSS
4. Add responsive designs to all components
5. Implement a theme toggle (light/dark mode) using your preferred method
6. Ensure all components are accessible

This exercise will help you understand the practical differences between various styling approaches and develop preferences for your future projects.

## Additional Resources

- [CSS Modules Documentation](https://github.com/css-modules/css-modules)
- [styled-components Documentation](https://styled-components.com/docs)
- [Emotion Documentation](https://emotion.sh/docs/introduction)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Material-UI Documentation](https://mui.com/material-ui/getting-started/)
- [Sass Documentation](https://sass-lang.com/documentation/)
- [React Transition Group](https://reactcommunity.org/react-transition-group/)
- [Framer Motion](https://www.framer.com/motion/)
- [CSS Custom Properties MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)
