### 1. Order of Lifecycle Method Calls in Class Components

Understanding the lifecycle of class components is crucial, especially when working with older codebases or using features like Error Boundaries.

**Phase 1: Mounting (Component is created and inserted into the DOM)**
1.  `constructor()` - Initialize state, bind event handlers.
2.  `static getDerivedStateFromProps()` - A rare use case for updating state based on prop changes before rendering.
3.  `render()` - The only required method. Returns the JSX/UI. Must be a pure function of props and state.
4.  `componentDidMount()` - The component is now in the DOM. This is the place for side effects like API calls and subscriptions.

**Phase 2: Updating (Component is re-rendered due to prop or state changes)**
1.  `static getDerivedStateFromProps()` - Called on every re-render.
2.  `shouldComponentUpdate()` - Performance optimization. Can return `false` to prevent a re-render.
3.  `render()` - Re-renders the UI.
4.  `getSnapshotBeforeUpdate()` - Captures information from the DOM (e.g., scroll position) right before it's about to change.
5.  `componentDidUpdate()` - The component has updated in the DOM. Perfect for side effects that need to happen after a DOM update.

**Phase 3: Unmounting (Component is removed from the DOM)**
1.  `componentWillUnmount()` - The component is about to be destroyed. This is the place for cleanup.

**Phase 4: Error Handling**
1.  `static getDerivedStateFromError()` - Renders a fallback UI after an error has been thrown.
2.  `componentDidCatch()` - Logs the error information.

---

### 2. Why do we use `componentDidMount`?

We use `componentDidMount` to run **side effects** that need to happen **after** the component has been successfully rendered to the DOM for the first time.

The key reason is that before `componentDidMount`, the component's DOM nodes **do not exist yet**. Any attempt to interact with the DOM or perform actions that rely on the component being "live" would fail.

**Primary Use Cases:**
*   **Data Fetching:** Initiating API calls to fetch data needed for the component.
*   **Subscriptions:** Setting up event listeners, WebSockets, or timers (`setInterval`).
*   **Direct DOM Manipulation:** As an escape hatch to integrate with third-party libraries that need a direct DOM node reference.

---

### 3. Why do we use `componentWillUnmount`? Show with an example.

We use `componentWillUnmount` to **clean up** any side effects that were created in `componentDidMount`. This is absolutely critical for **preventing memory leaks** and buggy behavior.

If you set up a subscription or a timer when a component mounts, but fail to tear it down when the component unmounts, that subscription will continue to run in the background, consuming memory and potentially trying to update a non-existent component, which will throw errors.

**Example: Cleaning up a Timer**
This component displays a ticking clock. It sets up a `setInterval` when it mounts and *must* clear it when it unmounts.

```jsx
import React from 'react';

class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
    this.timerID = null; // Good practice to store the timer ID on the instance
  }

  componentDidMount() {
    console.log("Clock mounted. Setting up timer...");
    // The side effect: a timer that ticks every second
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    // The CLEANUP: clearing the timer
    console.log("Clock will unmount. Cleaning up timer...");
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({ date: new Date() });
  }

  render() {
    return <h2>It is {this.state.date.toLocaleTimeString()}.</h2>;
  }
}
```
Without `componentWillUnmount`, if you were to navigate away from the page with this `Clock`, the `setInterval` would continue running forever, causing a memory leak.

---

### 4. (Research) Why do we use `super(props)` in the constructor?

**The Short Answer:** You must call `super(props)` if you want to access `this.props` **inside the constructor**.

**The Detailed Explanation:**

1.  **JavaScript Class Inheritance:** In JavaScript, when you have a subclass (your component) that extends a parent class (`React.Component`), the subclass's constructor must call `super()` to properly initialize the parent class. This is a JavaScript rule, not a React-specific one.
2.  **Initializing `this`:** The call to `super()` is what establishes the `this` context for the class instance. Before `super()` is called, `this` is uninitialized.
3.  **The Role of `React.Component`:** The constructor of the `React.Component` parent class is responsible for taking the `props` object and setting it up on the instance as `this.props`.
4.  **Connecting the Dots:** By calling `super(props)`, you are passing the `props` from your component's constructor up to the `React.Component` constructor. The parent constructor then does its job and initializes `this.props` with the `props` you passed in.

**What happens if you use `super()` or forget it?**
*   **`super()` without `props`:** `this.props` will be **undefined** *inside the constructor*. This can lead to bugs if you try to initialize state based on props.
*   **Forgetting `super()`:** This will cause a JavaScript reference error because `this` cannot be used before `super()` is called.

**Interesting Nuance:** React actually assigns `this.props` to the instance *after* the constructor has run, so `this.props` will still be available in `render()` and other lifecycle methods even if you forgot to pass it to `super()`. However, it is a convention and best practice to always use `super(props)` to avoid confusion and ensure `this.props` is available everywhere, including the constructor.

---

### 7. (Research) Why can't the callback function of `useEffect` be `async`?

**The Short Answer:** Because an `async` function implicitly returns a **Promise**, but the `useEffect` hook expects the return value to be either **nothing** (`undefined`) or a **cleanup function**.

**The Detailed Explanation:**

1.  **`useEffect`'s Signature:** The `useEffect` hook is designed to handle an optional cleanup mechanism. The function you pass to `useEffect` can return another function. React will execute this returned "cleanup" function when the component unmounts or before the effect runs again.
    ```javascript
    useEffect(() => {
      // Effect code...
      return () => {
        // Cleanup code...
      };
    }, []);
    ```
2.  **`async/await`'s Signature:** An `async` function, by its very definition in JavaScript, always returns a Promise. When the function completes, the promise resolves with the function's return value.
    ```javascript
    const myAsyncFunction = async () => {
      // No explicit return statement
    };
    console.log(myAsyncFunction()); // Logs: Promise {<fulfilled>: undefined}
    ```
3.  **The Conflict:** If you make the `useEffect` callback `async`, you are returning a Promise from it.
    ```javascript
    // THIS IS THE ANTI-PATTERN
    useEffect(async () => {
      await someAsyncOperation();
    }, []);
    ```
    React sees that the return value is a Promise, not a function. It doesn't know how to use a Promise for cleanup, so this can lead to race conditions and unexpected behavior. React will show a warning in the console about this.

**The Correct Pattern:**
To use async/await inside `useEffect`, define a separate `async` function *inside* the effect and then call it immediately. This way, the effect's callback itself doesn't return a promise.

```jsx
useEffect(() => {
  // 1. Define an async function inside the effect
  const fetchData = async () => {
    try {
      const response = await fetch('...');
      const data = await response.json();
      // Do something with the data
    } catch (error) {
      console.error("Failed to fetch data:", error);
    }
  };

  // 2. Call the async function
  fetchData();

  // You can still return a proper cleanup function here if needed
  return () => {
    // Cleanup logic
  };
}, [/* dependencies */]);
```