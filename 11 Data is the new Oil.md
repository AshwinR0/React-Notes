### 1. What is Prop Drilling?

**Direct Answer:**
Prop drilling is the process of passing data down from a high-level parent component to a deeply nested child component through multiple intermediate components that do not need the data themselves.

**The Problem in Detail:**
Imagine a component tree like this: `App > PageLayout > UserProfile > Avatar`.

The `App` component holds the `user` object. The `Avatar` component needs the `user.avatarUrl`. To get it there, you have to:
*   Pass `user` from `App` to `PageLayout`.
*   `PageLayout` doesn't need `user`, but it has to accept it just to pass it down to `UserProfile`.
*   `UserProfile` accepts `user` and passes it down to `Avatar`.
*   Finally, `Avatar` uses `user.avatarUrl`.

**Why is this a problem?**
*   **Maintenance Nightmare:** If you need to change the shape of the `user` object, or if `Avatar` suddenly needs a new prop from `App`, you have to modify the prop signature of every single intermediate component (`PageLayout`, `UserProfile`). This is tedious and error-prone.
*   **Tight Coupling:** The intermediate components are now unnecessarily coupled to the data they are passing. They are aware of a data structure that is irrelevant to their own function.
*   **Reduced Reusability:** It becomes harder to reuse a component like `PageLayout` in a different context because it now expects a `user` prop that it doesn't even use.
*   **Poor Readability:** It obscures where data is actually coming from and where it's being used, making the application harder to reason about.

**Prop drilling is a "code smell"** that indicates your component structure might need refactoring or that you need a more robust state management solution.

---

### 2. What is Lifting State Up?

**Direct Answer:**
"Lifting state up" is React's primary, built-in solution for sharing state between sibling components. The pattern involves moving the state from a child component to its closest common ancestor, and then passing that state down to the children that need it via props.

**The Scenario:**
Imagine you have two components, `TemperatureInputCelsius` and `TemperatureInputFahrenheit`. When you type in one, the other should update. They are siblings.

```
<App>
  <TemperatureInputCelsius />  // Needs to update Fahrenheit
  <TemperatureInputFahrenheit /> // Needs to update Celsius
</App>
```
They cannot communicate directly.

**The Solution:**
1.  **Identify the Common Ancestor:** The closest common ancestor is the `App` component.
2.  **Lift the State:** Move the state for the temperature from the individual input components up into the `App` component.
3.  **Pass State Down:** The `App` component now passes the current temperature value down to both inputs as a prop.
4.  **Pass Callbacks Up:** The `App` component also defines the function to *change* the temperature and passes this function down to the inputs as a prop (e.g., `onTemperatureChange`). When an input's value changes, it calls this function passed down from the parent.

**Code Example:**
```jsx
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  return (
    <fieldset>
      <legend>Enter temperature in {scale}:</legend>
      <input value={temperature} onChange={(e) => onTemperatureChange(e.target.value)} />
    </fieldset>
  );
}

function App() {
  // 1. State is "lifted" to the common ancestor.
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = = useState('c');

  // 4. Functions to handle changes are defined in the parent.
  const handleCelsiusChange = (temp) => {
    setTemperature(temp);
    setScale('c');
  }

  const handleFahrenheitChange = (temp) => {
    setTemperature(temp);
    setScale('f');
  }

  // Convert temperatures for the other input
  const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
  const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

  return (
    <div>
      {/* 3. State and callbacks are passed down as props. */}
      <TemperatureInput
        scale="Celsius"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange} />

      <TemperatureInput
        scale="Fahrenheit"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange} />
    </div>
  );
}
```
**Senior-Level Insight:** Lifting state up is the first and best tool for local state sharing. It follows React's principle of unidirectional data flow and keeps your components explicit and predictable. You should always reach for this solution *before* considering more complex tools like Context or Redux.

---

### 3. What are Context Provider and Context Consumer?

The **React Context API** is the built-in solution for avoiding prop drilling. It allows you to share state that can be considered "global" for a tree of React components, without having to pass props down manually at every level.

It consists of three main parts:

1.  **`React.createContext()`**
    This function creates a Context object. It takes an optional `defaultValue` as an argument.
    ```javascript
    // UserContext.js
    import { createContext } from 'react';
    export const UserContext = createContext(null); // Default value is null (no user)
    ```

2.  **The Context Provider (`<UserContext.Provider>`)**
    *   **Purpose:** This component "provides" a value to all components descending in its tree. It accepts a `value` prop.
    *   **How it works:** You wrap a parent component in the Provider. Any component within that wrapper (no matter how deeply nested) can now access the value.
    ```jsx
    // App.js
    import { UserContext } from './UserContext';

    function App() {
      const [currentUser, setCurrentUser] = useState({ name: 'Alice' });

      return (
        <UserContext.Provider value={currentUser}>
          <PageLayout />
        </UserContext.Provider>
      );
    }
    ```

3.  **Consuming the Context Value**
    There are two ways for a child component to "consume" or read the value.

    *   **The `useContext` Hook (Modern & Preferred):**
        This is the simplest and most common way. You call `useContext` with the Context object, and it returns the current context `value`.
        ```jsx
        // Avatar.js (deeply nested child)
        import { useContext } from 'react';
        import { UserContext } from './UserContext';

        function Avatar() {
          const user = useContext(UserContext); // Returns { name: 'Alice' }
          return <img alt={user.name} />;
        }
        ```

    *   **The Context Consumer (`<UserContext.Consumer>`) (Legacy):**
        This is the older way, used in class components or before Hooks were introduced. It uses a pattern called "Render Props". You wrap your JSX in the Consumer component, which provides the value as an argument to a function.
        ```jsx
        // Legacy Avatar.js
        import { UserContext } from './UserContext';

        function Avatar() {
          return (
            <UserContext.Consumer>
              {user => <img alt={user.name} />}
            </UserContext.Consumer>
          );
        }
        ```

---

### 4. If you don’t pass a value to the provider, does it take the default value?

**This is a critical nuance of the Context API.**

**Direct Answer:** **No.** If a component is wrapped in a Provider, it will **always** use the `value` prop from that Provider, even if that `value` prop is `undefined`.

**The `defaultValue` from `createContext(defaultValue)` is only used in one specific scenario: when a component tries to consume the context but has no matching Provider above it in the tree.**

**Let's illustrate with code:**

```jsx
import { createContext, useContext } from 'react';

// 1. Context created with a default value of 'Guest'
const NameContext = createContext('Guest');

function DisplayName() {
  const name = useContext(NameContext);
  return <h1>Hello, {name}</h1>;
}

// --- SCENARIOS ---

// Scenario A: No Provider in the tree
function App_A() {
  return <DisplayName />; // Renders: "Hello, Guest"
  // CORRECT: It uses the default value because there's no Provider.
}

// Scenario B: A Provider with a valid value
function App_B() {
  return (
    <NameContext.Provider value="Alice">
      <DisplayName /> {/* Renders: "Hello, Alice" */}
    </NameContext.Provider>
  );
  // CORRECT: It uses the value from the Provider.
}

// Scenario C: A Provider with NO `value` prop (the tricky one)
function App_C() {
  // The `value` prop is implicitly `undefined` here
  return (
    <NameContext.Provider>
      <DisplayName /> {/* Renders: "Hello, " (an empty string) */}
    </NameContext.Provider>
  );
  // INCORRECT ASSUMPTION: It does NOT fall back to 'Guest'.
  // It uses the Provider's value, which is `undefined`.
}
```

**Senior-Level Takeaway:** The `defaultValue` is primarily useful as a fallback for testing components in isolation without having to wrap them in a Provider, or in rare cases where a part of the app can function without a Provider. A common mistake is assuming it acts as a default for an `undefined` `value` prop. Always explicitly provide a value to your providers.