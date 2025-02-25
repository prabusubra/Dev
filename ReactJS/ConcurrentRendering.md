# Concurrent Rendering:-

- Concurrent Rendering in React is an optimized way of updating the UI without blocking the main thread.

-  It allows React to pause, prioritize, and resume rendering based on user interactions and system performance.

## Key Features:-

- **React Batching** ➡️ Groups multiple state updates to avoid unnecessary re-renders.

- **useTransition()** ➡️ Marks a state update as low-priority (non-blocking updates).

- **useDefferedValue()** ➡️ Allows a UI update to be deferred when higher-priority work is pending.

- **Suspense** ➡️ Helps in lazy-loading and streaming UI updates.

## How to enable react concurrent mode?

1. Upgrade to React 18+

2. Use createRoot() Instead of ReactDOM.render()

   ### Before (React 17 - Legacy Rendering)
     ```
      import React from "react";
      import ReactDOM from "react-dom";
      import App from "./App";

      ReactDOM.render(<App />, document.getElementById("root"));

     ```

   ### After (React 18 - Concurrent Rendering Enabled)

     ```
      import React from "react";
      import ReactDOM from "react-dom/client";
      import App from "./App";

      const root = ReactDOM.createRoot(document.getElementById("root"));
      root.render(<App />);

     ```

## Note:-
   ***✅ createRoot() enables concurrent rendering automatically.*** \
   ***❌ If you still use ReactDOM.render(), concurrent features will not work.***
