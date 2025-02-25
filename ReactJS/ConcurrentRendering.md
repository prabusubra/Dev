# Concurrent Rendering:-

- Concurrent Rendering in React is an optimized way of updating the UI without blocking the main thread.

-  It allows React to pause, prioritize, and resume rendering based on user interactions and system performance.

## Key Features:-

- **React Batching** ➡️ Groups multiple state updates to avoid unnecessary re-renders.

- **useTransition()** ➡️ Marks a state update as low-priority (non-blocking updates).

- **useDefferedValue()** ➡️ Allows a UI update to be deferred when higher-priority work is pending.

- **Suspense** ➡️ Helps in lazy-loading and streaming UI updates.
