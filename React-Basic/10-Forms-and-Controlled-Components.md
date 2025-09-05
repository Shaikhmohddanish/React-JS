# Forms and Controlled Components in React

## Introduction to Forms in React

Forms are a fundamental part of web applications, allowing users to input data and interact with your application. In React, forms work differently than in traditional HTML due to React's component-based architecture and state management approach. This comprehensive guide will cover everything you need to know about handling forms in React, from basic input fields to complex form validation.

## Why Forms Are Different in React

In traditional HTML, form elements maintain their own state and update it based on user input. However, in React, we prefer to have a single source of truth for all data, which means we need to control form elements through React state. This approach offers several benefits:

1. **Predictable State**: All form data is managed in one place
2. **Validation**: Easy to implement real-time validation
3. **Dynamic Forms**: Forms can change based on user input or external data
4. **Debugging**: Form state is visible in React DevTools
5. **Testing**: Easier to test form behavior programmatically

## Controlled vs Uncontrolled Components

There are two main approaches to handling form inputs in React:

### Controlled Components

In controlled components, form data is handled by React state. The React component that renders the form also controls what happens in that form on subsequent user input.

```jsx
function ControlledInput() {
  const [value, setValue] = useState('');
  
  const handleChange = (event) => {
    setValue(event.target.value);
  };
  
  return (
    <div>
      <label htmlFor="controlled-input">Controlled Input:</label>
      <input
        id="controlled-input"
        type="text"
        value={value}
        onChange={handleChange}
      />
      <p>Current value: {value}</p>
    </div>
  );
}
```

In this example:
- The input's value is controlled by React state (`value`)
- When the user types, `handleChange` updates the state
- The input always reflects the current state value

### Uncontrolled Components

In uncontrolled components, form data is handled by the DOM itself. You use refs to get form values from the DOM when you need them.

```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);
  
  const handleSubmit = (event) => {
    event.preventDefault();
    alert('Input value: ' + inputRef.current.value);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="uncontrolled-input">Uncontrolled Input:</label>
      <input
        id="uncontrolled-input"
        type="text"
        ref={inputRef}
        defaultValue="Initial value"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

In this example:
- The input manages its own state
- We use a ref to access the current value when needed
- `defaultValue` sets the initial value (not `value`)

### When to Use Each Approach

**Use Controlled Components when:**
- You need to validate input in real-time
- You want to disable the submit button based on input validation
- You need to format the input as the user types
- You want to implement features like autocomplete or suggestions
- You need to conditionally show/hide other form elements based on input

**Use Uncontrolled Components when:**
- You have a simple form that only needs values on submit
- You're integrating with non-React code
- You want to minimize re-renders for performance reasons
- You're working with file inputs (which are always uncontrolled)

## Basic Form Elements

Let's explore how to handle different types of form elements in React.

### Text Inputs

```jsx
function TextInputExample() {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: ''
  });
  
  const handleInputChange = (event) => {
    const { name, value } = event.target;
    setFormData(prevData => ({
      ...prevData,
      [name]: value
    }));
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="firstName">First Name:</label>
        <input
          id="firstName"
          type="text"
          name="firstName"
          value={formData.firstName}
          onChange={handleInputChange}
        />
      </div>
      
      <div>
        <label htmlFor="lastName">Last Name:</label>
        <input
          id="lastName"
          type="text"
          name="lastName"
          value={formData.lastName}
          onChange={handleInputChange}
        />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
        />
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

Notice how we use a single `handleInputChange` function for all inputs by using the `name` attribute to identify which field to update.

### Textarea

Textareas work similarly to input elements:

```jsx
function TextareaExample() {
  const [message, setMessage] = useState('');
  
  const handleChange = (event) => {
    setMessage(event.target.value);
  };
  
  return (
    <div>
      <label htmlFor="message">Message:</label>
      <textarea
        id="message"
        value={message}
        onChange={handleChange}
        rows={4}
        cols={50}
        placeholder="Enter your message here..."
      />
      <p>Character count: {message.length}</p>
    </div>
  );
}
```

### Select Dropdowns

Select elements in React use the `value` prop to control the selected option:

```jsx
function SelectExample() {
  const [selectedCountry, setSelectedCountry] = useState('');
  
  const countries = [
    { value: 'us', label: 'United States' },
    { value: 'ca', label: 'Canada' },
    { value: 'uk', label: 'United Kingdom' },
    { value: 'au', label: 'Australia' }
  ];
  
  const handleChange = (event) => {
    setSelectedCountry(event.target.value);
  };
  
  return (
    <div>
      <label htmlFor="country">Country:</label>
      <select id="country" value={selectedCountry} onChange={handleChange}>
        <option value="">Select a country</option>
        {countries.map(country => (
          <option key={country.value} value={country.value}>
            {country.label}
          </option>
        ))}
      </select>
      {selectedCountry && <p>Selected: {selectedCountry}</p>}
    </div>
  );
}
```

### Multi-Select

For multiple selections, use the `multiple` attribute and handle an array of values:

```jsx
function MultiSelectExample() {
  const [selectedSkills, setSelectedSkills] = useState([]);
  
  const skills = [
    { value: 'javascript', label: 'JavaScript' },
    { value: 'react', label: 'React' },
    { value: 'node', label: 'Node.js' },
    { value: 'python', label: 'Python' },
    { value: 'sql', label: 'SQL' }
  ];
  
  const handleChange = (event) => {
    const selectedOptions = Array.from(event.target.selectedOptions, option => option.value);
    setSelectedSkills(selectedOptions);
  };
  
  return (
    <div>
      <label htmlFor="skills">Skills (hold Ctrl/Cmd to select multiple):</label>
      <select
        id="skills"
        multiple
        value={selectedSkills}
        onChange={handleChange}
        size={skills.length}
      >
        {skills.map(skill => (
          <option key={skill.value} value={skill.value}>
            {skill.label}
          </option>
        ))}
      </select>
      <p>Selected skills: {selectedSkills.join(', ')}</p>
    </div>
  );
}
```

### Checkboxes

Checkboxes require special handling because they use the `checked` prop instead of `value`:

```jsx
function CheckboxExample() {
  const [preferences, setPreferences] = useState({
    newsletter: false,
    notifications: false,
    marketing: false
  });
  
  const handleCheckboxChange = (event) => {
    const { name, checked } = event.target;
    setPreferences(prevPreferences => ({
      ...prevPreferences,
      [name]: checked
    }));
  };
  
  return (
    <div>
      <h3>Email Preferences</h3>
      
      <div>
        <input
          id="newsletter"
          type="checkbox"
          name="newsletter"
          checked={preferences.newsletter}
          onChange={handleCheckboxChange}
        />
        <label htmlFor="newsletter">Subscribe to newsletter</label>
      </div>
      
      <div>
        <input
          id="notifications"
          type="checkbox"
          name="notifications"
          checked={preferences.notifications}
          onChange={handleCheckboxChange}
        />
        <label htmlFor="notifications">Receive notifications</label>
      </div>
      
      <div>
        <input
          id="marketing"
          type="checkbox"
          name="marketing"
          checked={preferences.marketing}
          onChange={handleCheckboxChange}
        />
        <label htmlFor="marketing">Receive marketing emails</label>
      </div>
      
      <div>
        <h4>Current preferences:</h4>
        <pre>{JSON.stringify(preferences, null, 2)}</pre>
      </div>
    </div>
  );
}
```

### Radio Buttons

Radio buttons are grouped by the `name` attribute and use the `checked` prop:

```jsx
function RadioButtonExample() {
  const [selectedPlan, setSelectedPlan] = useState('');
  
  const plans = [
    { value: 'basic', label: 'Basic - $9/month', description: 'Essential features' },
    { value: 'pro', label: 'Pro - $19/month', description: 'Advanced features' },
    { value: 'enterprise', label: 'Enterprise - $49/month', description: 'All features' }
  ];
  
  const handleRadioChange = (event) => {
    setSelectedPlan(event.target.value);
  };
  
  return (
    <div>
      <h3>Choose a plan:</h3>
      
      {plans.map(plan => (
        <div key={plan.value}>
          <input
            id={plan.value}
            type="radio"
            name="plan"
            value={plan.value}
            checked={selectedPlan === plan.value}
            onChange={handleRadioChange}
          />
          <label htmlFor={plan.value}>
            <strong>{plan.label}</strong>
            <br />
            <small>{plan.description}</small>
          </label>
        </div>
      ))}
      
      {selectedPlan && (
        <p>You selected: {plans.find(plan => plan.value === selectedPlan)?.label}</p>
      )}
    </div>
  );
}
```

### File Inputs

File inputs are always uncontrolled in React because you cannot set their value programmatically for security reasons:

```jsx
function FileInputExample() {
  const [selectedFiles, setSelectedFiles] = useState([]);
  const [uploadStatus, setUploadStatus] = useState('');
  
  const handleFileChange = (event) => {
    const files = Array.from(event.target.files);
    setSelectedFiles(files);
  };
  
  const handleUpload = async () => {
    if (selectedFiles.length === 0) {
      setUploadStatus('Please select files to upload');
      return;
    }
    
    setUploadStatus('Uploading...');
    
    // Simulate file upload
    try {
      for (const file of selectedFiles) {
        console.log('Uploading:', file.name, file.size, file.type);
        // In a real app, you would upload to a server here
      }
      setUploadStatus('Upload successful!');
    } catch (error) {
      setUploadStatus('Upload failed: ' + error.message);
    }
  };
  
  return (
    <div>
      <div>
        <label htmlFor="fileInput">Choose files:</label>
        <input
          id="fileInput"
          type="file"
          multiple
          accept=".jpg,.jpeg,.png,.pdf"
          onChange={handleFileChange}
        />
      </div>
      
      {selectedFiles.length > 0 && (
        <div>
          <h4>Selected files:</h4>
          <ul>
            {selectedFiles.map((file, index) => (
              <li key={index}>
                {file.name} ({Math.round(file.size / 1024)}KB)
              </li>
            ))}
          </ul>
          <button onClick={handleUpload}>Upload Files</button>
        </div>
      )}
      
      {uploadStatus && <p>{uploadStatus}</p>}
    </div>
  );
}
```

## Form Validation

Validation is crucial for ensuring data quality and providing good user experience. Let's explore different validation approaches.

### Basic Client-Side Validation

Here's a comprehensive example with various validation rules:

```jsx
function ValidationExample() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    age: '',
    website: ''
  });
  
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const validateField = (name, value) => {
    let error = '';
    
    switch (name) {
      case 'username':
        if (!value) {
          error = 'Username is required';
        } else if (value.length < 3) {
          error = 'Username must be at least 3 characters';
        } else if (!/^[a-zA-Z0-9_]+$/.test(value)) {
          error = 'Username can only contain letters, numbers, and underscores';
        }
        break;
        
      case 'email':
        if (!value) {
          error = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(value)) {
          error = 'Email is not valid';
        }
        break;
        
      case 'password':
        if (!value) {
          error = 'Password is required';
        } else if (value.length < 8) {
          error = 'Password must be at least 8 characters';
        } else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          error = 'Password must contain at least one lowercase letter, one uppercase letter, and one number';
        }
        break;
        
      case 'confirmPassword':
        if (!value) {
          error = 'Please confirm your password';
        } else if (value !== formData.password) {
          error = 'Passwords do not match';
        }
        break;
        
      case 'age':
        if (!value) {
          error = 'Age is required';
        } else if (isNaN(value) || value < 18 || value > 120) {
          error = 'Age must be a number between 18 and 120';
        }
        break;
        
      case 'website':
        if (value && !/^https?:\/\/.+\..+/.test(value)) {
          error = 'Website must be a valid URL (starting with http:// or https://)';
        }
        break;
        
      default:
        break;
    }
    
    return error;
  };
  
  const handleInputChange = (event) => {
    const { name, value } = event.target;
    
    setFormData(prevData => ({
      ...prevData,
      [name]: value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prevErrors => ({
        ...prevErrors,
        [name]: ''
      }));
    }
  };
  
  const handleBlur = (event) => {
    const { name, value } = event.target;
    
    setTouched(prevTouched => ({
      ...prevTouched,
      [name]: true
    }));
    
    const error = validateField(name, value);
    setErrors(prevErrors => ({
      ...prevErrors,
      [name]: error
    }));
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    
    // Validate all fields
    const newErrors = {};
    Object.keys(formData).forEach(key => {
      const error = validateField(key, formData[key]);
      if (error) {
        newErrors[key] = error;
      }
    });
    
    // Mark all fields as touched
    const allTouched = {};
    Object.keys(formData).forEach(key => {
      allTouched[key] = true;
    });
    setTouched(allTouched);
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Form is valid, submit the data
    console.log('Form submitted successfully:', formData);
    alert('Registration successful!');
  };
  
  const isFormValid = Object.keys(formData).every(key => {
    const error = validateField(key, formData[key]);
    return !error;
  });
  
  return (
    <form onSubmit={handleSubmit} className="validation-form">
      <h2>User Registration</h2>
      
      <div className="form-group">
        <label htmlFor="username">Username *</label>
        <input
          id="username"
          type="text"
          name="username"
          value={formData.username}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.username && touched.username ? 'error' : ''}
        />
        {errors.username && touched.username && (
          <span className="error-message">{errors.username}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          type="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.email && touched.email ? 'error' : ''}
        />
        {errors.email && touched.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="password">Password *</label>
        <input
          id="password"
          type="password"
          name="password"
          value={formData.password}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.password && touched.password ? 'error' : ''}
        />
        {errors.password && touched.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="confirmPassword">Confirm Password *</label>
        <input
          id="confirmPassword"
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.confirmPassword && touched.confirmPassword ? 'error' : ''}
        />
        {errors.confirmPassword && touched.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="age">Age *</label>
        <input
          id="age"
          type="number"
          name="age"
          value={formData.age}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.age && touched.age ? 'error' : ''}
        />
        {errors.age && touched.age && (
          <span className="error-message">{errors.age}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="website">Website</label>
        <input
          id="website"
          type="url"
          name="website"
          value={formData.website}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.website && touched.website ? 'error' : ''}
          placeholder="https://example.com"
        />
        {errors.website && touched.website && (
          <span className="error-message">{errors.website}</span>
        )}
      </div>
      
      <button type="submit" disabled={!isFormValid}>
        Register
      </button>
    </form>
  );
}
```

This example demonstrates:
- Real-time validation on blur
- Different validation rules for different field types
- Error state management
- Form submission prevention when invalid
- Visual feedback with CSS classes

### Custom Validation Hook

For reusability, you can create a custom hook for form validation:

```jsx
function useFormValidation(initialValues, validationRules) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const validateField = (name, value) => {
    const rule = validationRules[name];
    if (!rule) return '';
    
    return rule(value, values);
  };
  
  const handleChange = (event) => {
    const { name, value, type, checked } = event.target;
    const newValue = type === 'checkbox' ? checked : value;
    
    setValues(prevValues => ({
      ...prevValues,
      [name]: newValue
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prevErrors => ({
        ...prevErrors,
        [name]: ''
      }));
    }
  };
  
  const handleBlur = (event) => {
    const { name, value } = event.target;
    
    setTouched(prevTouched => ({
      ...prevTouched,
      [name]: true
    }));
    
    const error = validateField(name, value);
    setErrors(prevErrors => ({
      ...prevErrors,
      [name]: error
    }));
  };
  
  const validateForm = () => {
    const newErrors = {};
    
    Object.keys(validationRules).forEach(key => {
      const error = validateField(key, values[key]);
      if (error) {
        newErrors[key] = error;
      }
    });
    
    setErrors(newErrors);
    
    // Mark all fields as touched
    const allTouched = {};
    Object.keys(validationRules).forEach(key => {
      allTouched[key] = true;
    });
    setTouched(allTouched);
    
    return Object.keys(newErrors).length === 0;
  };
  
  const isValid = Object.keys(validationRules).every(key => {
    const error = validateField(key, values[key]);
    return !error;
  });
  
  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validateForm,
    isValid,
    setValues,
    setErrors,
    setTouched
  };
}

// Usage example
function LoginForm() {
  const validationRules = {
    email: (value) => {
      if (!value) return 'Email is required';
      if (!/\S+@\S+\.\S+/.test(value)) return 'Email is not valid';
      return '';
    },
    password: (value) => {
      if (!value) return 'Password is required';
      if (value.length < 6) return 'Password must be at least 6 characters';
      return '';
    }
  };
  
  const {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validateForm,
    isValid
  } = useFormValidation({ email: '', password: '' }, validationRules);
  
  const handleSubmit = (event) => {
    event.preventDefault();
    
    if (validateForm()) {
      console.log('Login attempt:', values);
      // Perform login logic here
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {errors.email && touched.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          id="password"
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {errors.password && touched.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>
      
      <button type="submit" disabled={!isValid}>
        Login
      </button>
    </form>
  );
}
```

## Dynamic Forms

Sometimes you need forms that can change structure based on user input or external data.

### Conditional Fields

Fields that appear or disappear based on other field values:

```jsx
function ConditionalFieldsForm() {
  const [formData, setFormData] = useState({
    accountType: '',
    companyName: '',
    personalTitle: '',
    hasExperience: false,
    experienceYears: '',
    skills: []
  });
  
  const handleInputChange = (event) => {
    const { name, value, type, checked } = event.target;
    const newValue = type === 'checkbox' ? checked : value;
    
    setFormData(prevData => ({
      ...prevData,
      [name]: newValue
    }));
  };
  
  const handleSkillsChange = (event) => {
    const selectedOptions = Array.from(event.target.selectedOptions, option => option.value);
    setFormData(prevData => ({
      ...prevData,
      skills: selectedOptions
    }));
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Dynamic Registration Form</h2>
      
      <div>
        <label htmlFor="accountType">Account Type:</label>
        <select
          id="accountType"
          name="accountType"
          value={formData.accountType}
          onChange={handleInputChange}
        >
          <option value="">Select account type</option>
          <option value="personal">Personal</option>
          <option value="business">Business</option>
        </select>
      </div>
      
      {/* Conditional field for business accounts */}
      {formData.accountType === 'business' && (
        <div>
          <label htmlFor="companyName">Company Name:</label>
          <input
            id="companyName"
            type="text"
            name="companyName"
            value={formData.companyName}
            onChange={handleInputChange}
            required
          />
        </div>
      )}
      
      {/* Conditional field for personal accounts */}
      {formData.accountType === 'personal' && (
        <div>
          <label htmlFor="personalTitle">Title:</label>
          <select
            id="personalTitle"
            name="personalTitle"
            value={formData.personalTitle}
            onChange={handleInputChange}
          >
            <option value="">Select title</option>
            <option value="mr">Mr.</option>
            <option value="ms">Ms.</option>
            <option value="mrs">Mrs.</option>
            <option value="dr">Dr.</option>
          </select>
        </div>
      )}
      
      <div>
        <input
          id="hasExperience"
          type="checkbox"
          name="hasExperience"
          checked={formData.hasExperience}
          onChange={handleInputChange}
        />
        <label htmlFor="hasExperience">I have professional experience</label>
      </div>
      
      {/* Conditional fields based on experience */}
      {formData.hasExperience && (
        <>
          <div>
            <label htmlFor="experienceYears">Years of Experience:</label>
            <input
              id="experienceYears"
              type="number"
              name="experienceYears"
              value={formData.experienceYears}
              onChange={handleInputChange}
              min="0"
              max="50"
            />
          </div>
          
          <div>
            <label htmlFor="skills">Skills (hold Ctrl to select multiple):</label>
            <select
              id="skills"
              multiple
              value={formData.skills}
              onChange={handleSkillsChange}
            >
              <option value="javascript">JavaScript</option>
              <option value="react">React</option>
              <option value="nodejs">Node.js</option>
              <option value="python">Python</option>
              <option value="sql">SQL</option>
              <option value="design">UI/UX Design</option>
            </select>
          </div>
        </>
      )}
      
      <button type="submit">Submit</button>
      
      {/* Debug info */}
      <details>
        <summary>Current form state:</summary>
        <pre>{JSON.stringify(formData, null, 2)}</pre>
      </details>
    </form>
  );
}
```

### Dynamic Field Arrays

Forms where users can add or remove fields dynamically:

```jsx
function DynamicFieldArrayForm() {
  const [formData, setFormData] = useState({
    name: '',
    contacts: [{ type: 'email', value: '' }]
  });
  
  const handleNameChange = (event) => {
    setFormData(prevData => ({
      ...prevData,
      name: event.target.value
    }));
  };
  
  const handleContactChange = (index, field, value) => {
    const newContacts = [...formData.contacts];
    newContacts[index] = {
      ...newContacts[index],
      [field]: value
    };
    
    setFormData(prevData => ({
      ...prevData,
      contacts: newContacts
    }));
  };
  
  const addContact = () => {
    setFormData(prevData => ({
      ...prevData,
      contacts: [...prevData.contacts, { type: 'email', value: '' }]
    }));
  };
  
  const removeContact = (index) => {
    if (formData.contacts.length > 1) {
      const newContacts = formData.contacts.filter((_, i) => i !== index);
      setFormData(prevData => ({
        ...prevData,
        contacts: newContacts
      }));
    }
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Contact Information Form</h2>
      
      <div>
        <label htmlFor="name">Name:</label>
        <input
          id="name"
          type="text"
          value={formData.name}
          onChange={handleNameChange}
          required
        />
      </div>
      
      <fieldset>
        <legend>Contact Methods</legend>
        
        {formData.contacts.map((contact, index) => (
          <div key={index} className="contact-group">
            <select
              value={contact.type}
              onChange={(e) => handleContactChange(index, 'type', e.target.value)}
            >
              <option value="email">Email</option>
              <option value="phone">Phone</option>
              <option value="linkedin">LinkedIn</option>
              <option value="twitter">Twitter</option>
            </select>
            
            <input
              type={contact.type === 'email' ? 'email' : 'text'}
              value={contact.value}
              onChange={(e) => handleContactChange(index, 'value', e.target.value)}
              placeholder={`Enter ${contact.type}`}
              required
            />
            
            <button
              type="button"
              onClick={() => removeContact(index)}
              disabled={formData.contacts.length === 1}
            >
              Remove
            </button>
          </div>
        ))}
        
        <button type="button" onClick={addContact}>
          Add Contact Method
        </button>
      </fieldset>
      
      <button type="submit">Submit</button>
      
      <details>
        <summary>Current form state:</summary>
        <pre>{JSON.stringify(formData, null, 2)}</pre>
      </details>
    </form>
  );
}
```

## Form Libraries

While you can build forms from scratch, there are excellent libraries that can simplify form handling, especially for complex forms.

### React Hook Form

React Hook Form is a popular library that minimizes re-renders and provides excellent performance:

```jsx
import { useForm } from 'react-hook-form';

function ReactHookFormExample() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isSubmitting, isValid }
  } = useForm({
    mode: 'onChange', // Validate on change
    defaultValues: {
      firstName: '',
      lastName: '',
      email: '',
      age: '',
      newsletter: false
    }
  });
  
  const onSubmit = async (data) => {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('Form submitted:', data);
    alert('Form submitted successfully!');
  };
  
  // Watch specific field
  const watchNewsletter = watch('newsletter');
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <h2>React Hook Form Example</h2>
      
      <div>
        <label htmlFor="firstName">First Name:</label>
        <input
          id="firstName"
          {...register('firstName', {
            required: 'First name is required',
            minLength: {
              value: 2,
              message: 'First name must be at least 2 characters'
            }
          })}
        />
        {errors.firstName && (
          <span className="error">{errors.firstName.message}</span>
        )}
      </div>
      
      <div>
        <label htmlFor="lastName">Last Name:</label>
        <input
          id="lastName"
          {...register('lastName', {
            required: 'Last name is required',
            minLength: {
              value: 2,
              message: 'Last name must be at least 2 characters'
            }
          })}
        />
        {errors.lastName && (
          <span className="error">{errors.lastName.message}</span>
        )}
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /\S+@\S+\.\S+/,
              message: 'Email is not valid'
            }
          })}
        />
        {errors.email && (
          <span className="error">{errors.email.message}</span>
        )}
      </div>
      
      <div>
        <label htmlFor="age">Age:</label>
        <input
          id="age"
          type="number"
          {...register('age', {
            required: 'Age is required',
            min: {
              value: 18,
              message: 'You must be at least 18 years old'
            },
            max: {
              value: 120,
              message: 'Age cannot exceed 120'
            }
          })}
        />
        {errors.age && (
          <span className="error">{errors.age.message}</span>
        )}
      </div>
      
      <div>
        <input
          id="newsletter"
          type="checkbox"
          {...register('newsletter')}
        />
        <label htmlFor="newsletter">Subscribe to newsletter</label>
      </div>
      
      {watchNewsletter && (
        <div className="newsletter-info">
          <p>Thanks for subscribing! You'll receive weekly updates.</p>
        </div>
      )}
      
      <button type="submit" disabled={!isValid || isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

### Formik

Formik is another popular form library with a different API approach:

```jsx
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object({
  firstName: Yup.string()
    .min(2, 'Too Short!')
    .max(50, 'Too Long!')
    .required('Required'),
  lastName: Yup.string()
    .min(2, 'Too Short!')
    .max(50, 'Too Long!')
    .required('Required'),
  email: Yup.string()
    .email('Invalid email')
    .required('Required'),
  age: Yup.number()
    .min(18, 'Must be at least 18')
    .max(120, 'Must be less than 120')
    .required('Required')
});

function FormikExample() {
  const initialValues = {
    firstName: '',
    lastName: '',
    email: '',
    age: ''
  };
  
  const handleSubmit = async (values, { setSubmitting }) => {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('Form submitted:', values);
    alert('Form submitted successfully!');
    setSubmitting(false);
  };
  
  return (
    <Formik
      initialValues={initialValues}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      {({ isSubmitting, isValid }) => (
        <Form>
          <h2>Formik Example</h2>
          
          <div>
            <label htmlFor="firstName">First Name:</label>
            <Field id="firstName" name="firstName" type="text" />
            <ErrorMessage name="firstName" component="div" className="error" />
          </div>
          
          <div>
            <label htmlFor="lastName">Last Name:</label>
            <Field id="lastName" name="lastName" type="text" />
            <ErrorMessage name="lastName" component="div" className="error" />
          </div>
          
          <div>
            <label htmlFor="email">Email:</label>
            <Field id="email" name="email" type="email" />
            <ErrorMessage name="email" component="div" className="error" />
          </div>
          
          <div>
            <label htmlFor="age">Age:</label>
            <Field id="age" name="age" type="number" />
            <ErrorMessage name="age" component="div" className="error" />
          </div>
          
          <button type="submit" disabled={!isValid || isSubmitting}>
            {isSubmitting ? 'Submitting...' : 'Submit'}
          </button>
        </Form>
      )}
    </Formik>
  );
}
```

## Form Accessibility

Making forms accessible is crucial for creating inclusive web applications.

### Accessible Form Example

```jsx
function AccessibleForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
    urgency: 'medium'
  });
  
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleInputChange = (event) => {
    const { name, value } = event.target;
    setFormData(prevData => ({
      ...prevData,
      [name]: value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prevErrors => ({
        ...prevErrors,
        [name]: ''
      }));
    }
  };
  
  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Please enter a valid email address';
    }
    
    if (!formData.message.trim()) {
      newErrors.message = 'Message is required';
    } else if (formData.message.length < 10) {
      newErrors.message = 'Message must be at least 10 characters long';
    }
    
    return newErrors;
  };
  
  const handleSubmit = async (event) => {
    event.preventDefault();
    
    const newErrors = validateForm();
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      
      // Focus on first error field
      const firstErrorField = Object.keys(newErrors)[0];
      const firstErrorElement = document.getElementById(firstErrorField);
      if (firstErrorElement) {
        firstErrorElement.focus();
      }
      
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));
      alert('Message sent successfully!');
      setFormData({ name: '', email: '', message: '', urgency: 'medium' });
    } catch (error) {
      alert('Error sending message. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} noValidate>
      <fieldset>
        <legend>Contact Form</legend>
        
        <div className="form-group">
          <label htmlFor="name">
            Name <span aria-label="required">*</span>
          </label>
          <input
            id="name"
            name="name"
            type="text"
            value={formData.name}
            onChange={handleInputChange}
            required
            aria-describedby={errors.name ? 'name-error' : undefined}
            aria-invalid={errors.name ? 'true' : 'false'}
          />
          {errors.name && (
            <div id="name-error" role="alert" className="error">
              {errors.name}
            </div>
          )}
        </div>
        
        <div className="form-group">
          <label htmlFor="email">
            Email Address <span aria-label="required">*</span>
          </label>
          <input
            id="email"
            name="email"
            type="email"
            value={formData.email}
            onChange={handleInputChange}
            required
            aria-describedby={errors.email ? 'email-error' : 'email-help'}
            aria-invalid={errors.email ? 'true' : 'false'}
          />
          <div id="email-help" className="help-text">
            We'll never share your email with anyone else.
          </div>
          {errors.email && (
            <div id="email-error" role="alert" className="error">
              {errors.email}
            </div>
          )}
        </div>
        
        <div className="form-group">
          <label htmlFor="urgency">Urgency Level</label>
          <select
            id="urgency"
            name="urgency"
            value={formData.urgency}
            onChange={handleInputChange}
          >
            <option value="low">Low - General inquiry</option>
            <option value="medium">Medium - Business related</option>
            <option value="high">High - Urgent assistance needed</option>
          </select>
        </div>
        
        <div className="form-group">
          <label htmlFor="message">
            Message <span aria-label="required">*</span>
          </label>
          <textarea
            id="message"
            name="message"
            value={formData.message}
            onChange={handleInputChange}
            required
            rows={4}
            aria-describedby={errors.message ? 'message-error' : 'message-help'}
            aria-invalid={errors.message ? 'true' : 'false'}
          />
          <div id="message-help" className="help-text">
            Please provide as much detail as possible (minimum 10 characters).
          </div>
          {errors.message && (
            <div id="message-error" role="alert" className="error">
              {errors.message}
            </div>
          )}
        </div>
        
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? (
            <>
              <span aria-hidden="true">⏳</span>
              <span className="sr-only">Sending message...</span>
              Sending...
            </>
          ) : (
            'Send Message'
          )}
        </button>
      </fieldset>
    </form>
  );
}
```

This accessible form includes:
- Proper labeling with `htmlFor` attributes
- ARIA attributes for screen readers
- Error messages with `role="alert"`
- Help text associated with form fields
- Focus management for error states
- Loading states with appropriate ARIA labels

## Best Practices

### 1. Use Semantic HTML

Always use proper HTML form elements and attributes:

```jsx
// Good
<form onSubmit={handleSubmit}>
  <fieldset>
    <legend>Personal Information</legend>
    <label htmlFor="name">Name:</label>
    <input id="name" type="text" name="name" required />
  </fieldset>
</form>

// Avoid
<div onClick={handleSubmit}>
  <div>Name:</div>
  <div contentEditable />
</div>
```

### 2. Provide Clear Feedback

Users should always know the state of their form:

```jsx
function FormWithFeedback() {
  const [status, setStatus] = useState('idle'); // idle, submitting, success, error
  
  const handleSubmit = async (event) => {
    event.preventDefault();
    setStatus('submitting');
    
    try {
      await submitForm();
      setStatus('success');
    } catch (error) {
      setStatus('error');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      
      <button type="submit" disabled={status === 'submitting'}>
        {status === 'submitting' ? 'Submitting...' : 'Submit'}
      </button>
      
      {status === 'success' && (
        <div className="success-message" role="alert">
          Form submitted successfully!
        </div>
      )}
      
      {status === 'error' && (
        <div className="error-message" role="alert">
          There was an error submitting the form. Please try again.
        </div>
      )}
    </form>
  );
}
```

### 3. Validate Progressively

Don't overwhelm users with all validation errors at once:

```jsx
function ProgressiveValidation() {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [touchedFields, setTouchedFields] = useState({});
  
  const handleBlur = (fieldName) => {
    setTouchedFields(prev => ({ ...prev, [fieldName]: true }));
  };
  
  const getFieldError = (fieldName) => {
    if (!touchedFields[fieldName]) return '';
    
    // Only validate fields that have been touched
    if (fieldName === 'email' && !formData.email.includes('@')) {
      return 'Please enter a valid email';
    }
    
    return '';
  };
  
  return (
    <form>
      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
        onBlur={() => handleBlur('email')}
      />
      {getFieldError('email') && <span>{getFieldError('email')}</span>}
    </form>
  );
}
```

### 4. Handle Edge Cases

Consider various edge cases in your forms:

```jsx
function RobustForm() {
  const [formData, setFormData] = useState({ name: '', email: '' });
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  const handleSubmit = (event) => {
    event.preventDefault();
    
    if (!isOnline) {
      alert('You appear to be offline. Please check your connection and try again.');
      return;
    }
    
    // Handle form submission
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {!isOnline && (
        <div className="warning">
          You are currently offline. Form submission is disabled.
        </div>
      )}
      
      {/* Form fields */}
      
      <button type="submit" disabled={!isOnline}>
        Submit
      </button>
    </form>
  );
}
```

## Summary

Forms are a critical part of React applications, and mastering them requires understanding:

1. **Controlled vs Uncontrolled Components**: Know when to use each approach
2. **Form Elements**: Handle different input types properly
3. **Validation**: Implement both client-side and server-side validation
4. **Dynamic Forms**: Create forms that adapt to user input
5. **Accessibility**: Make forms usable by everyone
6. **Performance**: Optimize for good user experience
7. **Error Handling**: Provide clear feedback to users

By following the patterns and best practices outlined in this guide, you can create robust, user-friendly forms that provide excellent user experiences while maintaining clean, maintainable code.

## Practice Exercise

Build a comprehensive user registration form with the following features:

1. Multiple steps (personal info, account details, preferences)
2. Real-time validation with appropriate error messages
3. Dynamic fields based on user selections
4. File upload for profile picture
5. Accessibility features
6. Proper error handling and loading states
7. Form data persistence across steps

This exercise will help you apply all the concepts learned in this guide to create a production-ready form component.

## Additional Resources

- [React Official Documentation on Forms](https://react.dev/reference/react-dom/components#form-components)
- [React Hook Form](https://react-hook-form.com/) - Performant forms with easy validation
- [Formik](https://formik.org/) - Popular form library for React
- [Yup](https://github.com/jquense/yup) - Schema validation library
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/) - Web accessibility guidelines
