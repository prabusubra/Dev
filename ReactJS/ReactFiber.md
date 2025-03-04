# React Fiber

- React16+

- React Fiber was a complete rewrite of ***React’s reconciliation*** algorithm.

- React Fiber is the default render engine to improve performace and user experience.

- Before React 16, React used the Stack Reconciler, which was synchronous and blocking,
  causing UI freezes during heavy rendering tasks.

- React Fiber solved this problem by making rendering interruptible and prioritized,
  leading to smoother and more efficient UI updates.

## Prioritized Updates

  - React Fiber assigns priorities to different updates:

    - **High-priority updates:** User interactions (e.g., typing in an input).
    - **Low-priority updates:** Heavy computations (e.g., filtering large datasets).
      
  - This allows React to render important updates first while deferring less critical ones.

##  Error Boundaries (Improved Error Handling)

  - React 16 introduced Error Boundaries, which let components gracefully handle errors without crashing the whole app.
  - Before React 16: If an error occurred, React would unmount the entire component tree.
  - After React 16 (with Fiber): Errors can be caught inside special components called Error Boundaries.

  - ### Example:

    ```
    class ErrorBoundary extends React.Component {
      constructor(props) {
        super(props);
        this.state = { hasError: false };
      }
    
      static getDerivedStateFromError(error) {
        return { hasError: true };
      }
    
      render() {
        if (this.state.hasError) {
          return <h2>Something went wrong.</h2>;
        }
        return this.props.children;
      }
    }

    // Usage:
    <ErrorBoundary>
    <MyComponent />
    </ErrorBoundary>;
    ```

    
| Feature |	Before Fiber (React 15) | After Fiber (React 16) |
|----------|-------------------------|-----------------------|
| Rendering |	Blocking, Synchronous |	Non-blocking, Asynchronous |
| Prioritized Updates |	No |	Yes (High/Low Priority) |
| Animations |	Laggy & Janky |	Smooth & Interruptible |
| Error Handling	| Crashes Entire App	| Error Boundaries (Handles Errors Gracefully) |
| Fragments |	Extra <div> Wrappers Needed	| No Extra Wrappers (<></>) |
Portals |	Not Supported	| Supported (ReactDOM.createPortal) |

## Improvements in React17

- **Improved Event Delegation:**

  - 🌲 Before React 17: React attached event listeners to document.
    
  - 💽 React 17 with Fiber: Event listeners are now attached at the root container, improving compatibility with third-party libraries.

    ✅ Use Case: Better integration with non-React apps and micro frontends.

- **Incremental Adoption:**

  - 🥇 ***Mainly For Smooth upgrades***

  - React 17 introduced incremental adoption, meaning different parts of an app can run on different React versions.

  - React Fiber makes this possible by allowing multiple versions of React to coexist without conflicts.
  
    ✅ Use Case: Migrate old parts of your app to React 17 without rewriting everything at once.
