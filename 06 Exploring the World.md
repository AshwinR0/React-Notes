

### 1. Monolith vs. Microservices Architecture

This is a high-level system design question. As a senior frontend developer, you are expected to understand the architecture you're building against, as it directly impacts your work.

#### **What is Monolith Architecture?**

A monolith ("mono" meaning one) is a traditional software architecture where an entire application is built as a **single, unified unit**. The frontend, backend, business logic, and data access layer are all contained within one large codebase and deployed together.

**Analogy:** A large department store. Everything is in one building. If you want to renovate the shoe department, you might disrupt the entire store's operation.

#### **What is a Microservice Architecture?**

A microservice architecture is an approach where a large application is broken down into a collection of **small, independent, and loosely coupled services**. Each service is self-contained, has its own codebase, manages its own data, and can be deployed independently. These services communicate with each other over a network, typically via HTTP/REST APIs.

**Analogy:** A shopping mall. Each store (service) is independent. The shoe store can renovate without affecting the food court. They all operate within the mall (the system) but can be managed and updated separately.

#### **Key Differences: Monolith vs. Microservices**

| Feature | Monolith | Microservices |
| :--- | :--- | :--- |
| **Codebase** | Single, large, unified codebase. | Multiple small, independent codebases. |
| **Deployment** | The entire application is deployed as one unit. A small change requires redeploying everything. | Services can be deployed independently. A change in one service doesn't affect others. |
| **Technology Stack** | Locked into a single technology stack (e.g., all Node.js, all Java). | Polyglot. Each service can use the best technology for its specific job (e.g., Go for performance, Python for data science). |
| **Scalability** | Must scale the entire application, even if only one small part is a bottleneck. | Can scale individual services as needed, which is more resource-efficient. |
| **Fault Isolation** | A critical bug or failure in one module can bring down the entire application. | A failure in one service will likely not affect the others, leading to a more resilient system. |
| **Development** | Simpler to start. Easier to manage in the early stages. | More complex upfront (DevOps, networking, service discovery). Teams can work in parallel on different services with less conflict. |

**Senior Frontend Perspective:** How does this affect you?
*   **In a Monolith:** You typically interact with a single backend API. State management might be simpler.
*   **With Microservices:** You might need to orchestrate calls to multiple services. An **API Gateway** is often used to aggregate these services into a single endpoint for the frontend. You have to be more aware of network latency and handling failures in individual service calls.

---

### 2. Why do we need the `useEffect` Hook?

This question probes for a deeper, conceptual understanding beyond just "it's for side effects."

**The Core Problem:** React's rendering model is fundamentally pure. A component's output (its UI) should be a direct, predictable result of its props and state: `(props, state) => UI`. This pure rendering logic should not have any "side effects" like modifying things outside of the component.

However, real-world applications are not pure. We constantly need to interact with the "outside world"—the browser, the network, timers, etc.

**The Solution:** `useEffect` is the official **bridge** between React's pure, declarative world and the imperative, effectful world of the browser. It provides a safe, declarative way to run side effects *after* React has finished rendering and committed the changes to the DOM. It lets you **synchronize** your component's state with an external system.

---

### 3. What is Optional Chaining (`?.`)?

**Direct Answer:** Optional Chaining is a modern JavaScript operator (`?.`) that provides a safe way to access deeply nested properties of an object without having to write a long chain of explicit null checks.

**The Problem it Solves:** It prevents the common runtime error: `TypeError: Cannot read properties of undefined`.

**Before Optional Chaining:**
```javascript
// To safely access user.address.street, you had to do this:
const street = user && user.address && user.address.street;
```

**With Optional Chaining:**
```javascript
const street = user?.address?.street;
```
If `user` or `user.address` is `null` or `undefined`, the expression **short-circuits** and immediately returns `undefined`, preventing the error. It can also be used with function calls: `user.getName?.()`.

---

### 4. What is a Shimmer UI?

A Shimmer UI, also known as a **loading skeleton**, is a user experience pattern where a placeholder version of the UI is shown while the actual content is being loaded. This placeholder typically mimics the size and shape of the content, often with a subtle shimmering or pulsing animation to indicate that something is happening.

**Why is it important? (Senior-level insight)**
*   **Improves Perceived Performance:** It makes the application feel faster. A blank screen or a generic spinner tells the user nothing about the upcoming content. A shimmer UI sets expectations and makes the wait feel shorter.
*   **Reduces Layout Shift:** It helps prevent **Cumulative Layout Shift (CLS)**, a Google Core Web Vitals metric. By rendering a skeleton that has the same dimensions as the final content, you avoid the jarring effect of content "jumping" around the page as it loads.

---

### 5. JS Expression vs. JS Statement

This is a fundamental computer science concept applied to JavaScript.

*   **Expression:** Any valid unit of code that **resolves to a single value**.
    *   `1 + 1` (resolves to `2`)
    *   `getUserName()` (resolves to the return value of the function)
    *   `x > 5 ? 'yes' : 'no'` (the ternary operator is an expression)
*   **Statement:** An instruction that **performs an action**. Statements do not resolve to a value.
    *   `let x = 10;` (a variable declaration statement)
    *   `if (x > 5) { ... }` (an `if` statement)
    *   `for (let i = 0; i < 5; i++) { ... }` (a `for` loop statement)

**Relevance to React:** Inside JSX curly braces `{}`, you can **only put expressions**. This is why you can use a ternary operator but not an `if` statement directly within your JSX.
```jsx
<div>
  {/* VALID: Ternary is an expression */}
  {isLoggedIn ? <p>Welcome!</p> : <p>Please log in.</p>}

  {/* INVALID: `if` is a statement */}
  {/* if (isLoggedIn) { <p>Welcome!</p> } */}
</div>
```

---

### 6. What is Conditional Rendering?

Conditional rendering in React is the practice of rendering different components or elements based on certain conditions (typically the component's state or props).

**Code Example using common patterns:**

```jsx
function UserGreeting({ isLoggedIn, username, notifications }) {
  // Pattern 1: `if/else` (good for complex logic outside of JSX)
  if (!isLoggedIn) {
    return <p>Please log in to continue.</p>;
  }

  // If logged in, we render the main content
  return (
    <div>
      {/* Pattern 2: Ternary Operator (perfect for simple if/else) */}
      <h1>{username ? `Welcome back, ${username}!` : 'Welcome!'}</h1>

      {/* Pattern 3: Logical && Operator (great for "render this OR nothing") */}
      {notifications.length > 0 && (
        <div className="notification-badge">
          You have {notifications.length} new messages.
        </div>
      )}
    </div>
  );
}
```

---

### 7. What is CORS?

**CORS** stands for **Cross-Origin Resource Sharing**. It is a browser security mechanism that allows or restricts web applications running at one origin (domain) from making requests to resources at a different origin.

**The Problem:** By default, browsers enforce the **Same-Origin Policy (SOP)**. This policy prevents a script on `https://my-cool-app.com` from making a `fetch` request to `https://api.another-domain.com`. This is a crucial security feature to prevent malicious scripts from accessing sensitive data on other sites.

**How CORS Solves It:** CORS works by having the **server** at the different origin (`api.another-domain.com`) send specific HTTP headers that tell the browser it's okay to allow the request from `my-cool-app.com`. The most important header is `Access-Control-Allow-Origin`.

*   `Access-Control-Allow-Origin: *` (Allows any origin - less secure)
*   `Access-Control-Allow-Origin: https://my-cool-app.com` (Allows only this specific origin)

**Senior-Level Detail (Preflight Request):** For requests that can modify data (like `PUT`, `POST`, `DELETE`) or have custom headers, the browser first sends an `OPTIONS` request called a **preflight request**. This is like the browser "asking for permission" from the server before sending the actual request. The server must respond to the `OPTIONS` request with the correct CORS headers for the browser to proceed.

---

### 8. What are `async` and `await`?

`async` and `await` are JavaScript keywords that provide **syntactic sugar** on top of Promises, making asynchronous code easier to write and read as if it were synchronous.

*   **`async`:** When placed before a function declaration, it ensures two things:
    1.  The function will implicitly return a Promise.
    2.  It allows the use of the `await` keyword inside that function.
*   **`await`:** Can only be used inside an `async` function. It **pauses the execution of the function** until the Promise it's waiting on either resolves or rejects. If the Promise resolves, `await` returns the resolved value. If it rejects, it throws an error which can be caught by a `try...catch` block.

---

### 9. What is the use of `const json = await data.json()`?

This is a very common line of code when using the `fetch` API. Let's break it down:

1.  **`fetch()` returns a `Response` object:** When you `await fetch('...')`, the resolved value (`data` in this case) is not the final JSON data. It's a `Response` object which is a representation of the entire HTTP response (headers, status, etc.). The actual body of the response is a **stream** that needs to be read.
2.  **`.json()` reads the stream:** The `.json()` method is a function on the `Response` object. Its job is to read the response stream to completion and parse the body text as JSON.
3.  **`.json()` is asynchronous:** Because reading a network stream can take time, the `.json()` method itself is an asynchronous operation and **returns a Promise**.
4.  **`await` waits for parsing:** The `await` keyword pauses the function until the Promise returned by `.json()` resolves. The resolved value of this promise is the fully parsed JavaScript object.
5.  **`const json = ...`:** The resulting JavaScript object is then assigned to the `json` constant.

In short, this line is a two-step process: `data` is the HTTP response container, and `await data.json()` is the asynchronous action of extracting and parsing the JSON body from that container.