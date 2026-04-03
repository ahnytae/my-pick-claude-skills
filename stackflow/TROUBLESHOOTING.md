# Stackflow Troubleshooting Guide

This document provides solutions to common issues and problems when working with Stackflow.

## Table of Contents
- [Installation Issues](#installation-issues)
- [Navigation Problems](#navigation-problems)
- [Transition Issues](#transition-issues)
- [TypeScript Errors](#typescript-errors)
- [Performance Problems](#performance-problems)
- [Browser History Issues](#browser-history-issues)
- [Styling Issues](#styling-issues)
- [Mobile-Specific Issues](#mobile-specific-issues)
- [Plugin Issues](#plugin-issues)

## Installation Issues

### Problem: Module not found errors after installation

```bash
Error: Cannot find module '@stackflow/react'
```

**Solution:**
1. Clear node_modules and reinstall:
```bash
rm -rf node_modules package-lock.json
npm install
```

2. Ensure all required packages are installed:
```bash
npm install @stackflow/react @stackflow/plugin-basic-ui
```

3. If using TypeScript, install type definitions:
```bash
npm install -D @types/react @types/react-dom
```

### Problem: Version conflicts with React

```bash
npm ERR! peer dep missing: react@^18.0.0
```

**Solution:**
Ensure your React version is compatible (16.8+):
```bash
npm install react@^18.0.0 react-dom@^18.0.0
```

Or use `--legacy-peer-deps` flag:
```bash
npm install --legacy-peer-deps
```

## Navigation Problems

### Problem: Navigation doesn't work (push/pop has no effect)

**Symptoms:**
- Calling `push()` or `pop()` doesn't change the screen
- No errors in console

**Solution 1: Check if Stack is rendered**
```typescript
// ✅ Correct: Stack component must be rendered
function App() {
  return <Stack />;
}

// ❌ Wrong: Stack not rendered
function App() {
  const { Stack } = stackflow({...});
  return <div>Hello</div>; // Stack not rendered!
}
```

**Solution 2: Ensure you're using the correct useFlow hook**
```typescript
// ✅ Correct: Import from your stackflow.ts
import { useFlow } from './stackflow';

// ❌ Wrong: Importing from package
import { useFlow } from '@stackflow/react'; // This won't work!
```

**Solution 3: Check if activities are registered**
```typescript
// Make sure all activities are in the config
export const { Stack, useFlow } = stackflow({
  activities: {
    Home: HomeActivity, // Must be here!
    Detail: DetailActivity,
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});
```

### Problem: Parameters not being passed correctly

**Symptoms:**
- Activity receives undefined or empty params
- TypeScript errors about parameter types

**Solution 1: Ensure parameter types match**
```typescript
// Define types
type DetailParams = {
  id: string;
};

// Use correct types when pushing
const { push } = useFlow();
push('Detail', { id: '123' }); // ✅ Correct

// ❌ Wrong: parameter name mismatch
push('Detail', { productId: '123' }); // Activity expects 'id', not 'productId'
```

**Solution 2: Check TypeScript configuration**
```typescript
// stackflow.ts
type TypedActivities = {
  Home: {};
  Detail: { id: string }; // Define parameters here
};

export const { Stack, useFlow } = stackflow<TypedActivities>({
  activities: {
    Home: HomeActivity,
    Detail: DetailActivity,
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});
```

### Problem: Cannot go back with pop()

**Symptoms:**
- `pop()` doesn't work
- Stuck on current screen

**Solution 1: Check if there's a previous activity**
```typescript
import { useActivity } from '@stackflow/react';

const MyActivity = () => {
  const activity = useActivity();
  const { pop } = useFlow();
  
  // Check if we can go back
  const canGoBack = activity.depth > 0;
  
  return (
    <button 
      onClick={() => canGoBack && pop()}
      disabled={!canGoBack}
    >
      Back
    </button>
  );
};
```

**Solution 2: Use replace() instead of push() for initial navigation**
```typescript
// If you want to prevent going back to welcome screen
const WelcomeActivity = () => {
  const { replace } = useFlow();
  
  // Use replace instead of push
  const goToMain = () => replace('Main', {}); // Can't go back to Welcome
};
```

## Transition Issues

### Problem: Transitions are janky or stuttering

**Solution 1: Reduce transition duration**
```typescript
export const { Stack, useFlow } = stackflow({
  transitionDuration: 300, // Try reducing from 350ms to 300ms
  activities: {...},
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});
```

**Solution 2: Check for expensive renders during transition**
```typescript
// ❌ Bad: Heavy computation during render
const MyActivity = () => {
  const heavyData = expensiveComputation(); // Runs on every render!
  return <div>{heavyData}</div>;
};

// ✅ Good: Memoize expensive computations
const MyActivity = () => {
  const heavyData = useMemo(() => expensiveComputation(), []);
  return <div>{heavyData}</div>;
};
```

**Solution 3: Use CSS will-change for GPU acceleration**
```css
.activity {
  will-change: transform;
  transform: translateZ(0); /* Force GPU acceleration */
}
```

**Solution 4: Lazy load activities**
```typescript
// Reduce initial bundle size
const MyActivity = lazy(() => import('./activities/MyActivity'));
```

### Problem: No transition animation appears

**Solution 1: Ensure basicUIPlugin is configured**
```typescript
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

export const { Stack, useFlow } = stackflow({
  transitionDuration: 350,
  activities: {...},
  plugins: [
    basicUIPlugin({ 
      theme: 'cupertino' // or 'android'
    })
  ],
});
```

**Solution 2: Import required CSS**
```typescript
// App.tsx or index.tsx
import '@stackflow/plugin-basic-ui/index.css'; // Don't forget this!
```

**Solution 3: Check for CSS conflicts**
```css
/* Make sure nothing is overriding Stackflow's styles */
* {
  transform: none !important; /* ❌ This would break transitions */
}
```

## TypeScript Errors

### Problem: Type errors when using push/pop

```typescript
// Error: Argument of type '{ id: string }' is not assignable to parameter
```

**Solution: Define activities with proper types**
```typescript
// stackflow.ts
type TypedActivities = {
  Home: {};
  Detail: { id: string };
  Profile: { userId: string; tab?: 'posts' | 'followers' };
};

export const { Stack, useFlow } = stackflow<TypedActivities>({
  activities: {
    Home: HomeActivity,
    Detail: DetailActivity,
    Profile: ProfileActivity,
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});

// Now TypeScript will validate your navigation
const MyComponent = () => {
  const { push } = useFlow();
  
  push('Detail', { id: '123' }); // ✅ Type-safe
  push('Detail', { productId: '123' }); // ❌ Type error
};
```

### Problem: params is 'any' type in activity

**Solution: Use ActivityComponentType with generic**
```typescript
// ❌ Wrong: params will be 'any'
const DetailActivity = ({ params }) => {
  return <div>{params.id}</div>;
};

// ✅ Correct: params is properly typed
type DetailParams = {
  id: string;
};

const DetailActivity: ActivityComponentType<DetailParams> = ({ params }) => {
  return <div>{params.id}</div>; // params.id is string
};
```

### Problem: useFlow() returns 'any' type

**Solution: Export typed useFlow from stackflow.ts**
```typescript
// stackflow.ts
type TypedActivities = {
  Home: {};
  Detail: { id: string };
};

export const { Stack, useFlow } = stackflow<TypedActivities>({...});

// ✅ Import this typed useFlow, not the one from @stackflow/react
import { useFlow } from './stackflow';
```

## Performance Problems

### Problem: Slow initial load

**Solution 1: Code split activities**
```typescript
import { lazy } from 'react';

const HomeActivity = lazy(() => import('./activities/Home'));
const DetailActivity = lazy(() => import('./activities/Detail'));

export const { Stack, useFlow } = stackflow({
  activities: {
    Home: HomeActivity,
    Detail: DetailActivity,
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});
```

**Solution 2: Add Suspense boundary**
```typescript
import { Suspense } from 'react';
import { Stack } from './stackflow';

function App() {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <Stack />
    </Suspense>
  );
}
```

### Problem: Activities re-render unnecessarily

**Solution: Memoize components and callbacks**
```typescript
import { memo, useCallback, useMemo } from 'react';

// Memoize child components
const ProductCard = memo(({ product, onClick }) => {
  return <div onClick={onClick}>{product.name}</div>;
});

// Memoize callbacks
const ProductList = () => {
  const { push } = useFlow();
  
  const handleClick = useCallback((id: string) => {
    push('Detail', { id });
  }, [push]);
  
  return products.map(p => (
    <ProductCard 
      key={p.id} 
      product={p} 
      onClick={() => handleClick(p.id)} 
    />
  ));
};
```

### Problem: Large bundle size

**Solution 1: Analyze bundle**
```bash
npm install -D webpack-bundle-analyzer
# Add to webpack config or use Vite's build analyzer
```

**Solution 2: Lazy load activities and dependencies**
```typescript
// Lazy load heavy dependencies
const ChartActivity = lazy(() => import('./activities/Chart'));

// Inside activity, lazy load heavy libraries
const ChartActivity = () => {
  const [Chart, setChart] = useState(null);
  
  useEffect(() => {
    import('chart.js').then(module => setChart(() => module.Chart));
  }, []);
  
  if (!Chart) return <div>Loading...</div>;
  
  return <ChartComponent Chart={Chart} />;
};
```

## Browser History Issues

### Problem: Browser back button doesn't work as expected

**Solution: Ensure history plugin is configured**
```typescript
import { basicRendererPlugin } from '@stackflow/plugin-renderer-basic';

export const { Stack, useFlow } = stackflow({
  activities: {...},
  plugins: [
    basicUIPlugin({ theme: 'cupertino' }),
    basicRendererPlugin(), // Add this for history support
  ],
});
```

### Problem: URL doesn't update when navigating

**Solution: Manually sync URL with navigation**
```typescript
const MyActivity = () => {
  const { push } = useFlow();
  
  const handleNavigate = (id: string) => {
    // Update URL
    window.history.pushState({}, '', `/detail/${id}`);
    
    // Navigate in Stackflow
    push('Detail', { id });
  };
  
  return <button onClick={() => handleNavigate('123')}>Go</button>;
};
```

### Problem: Deep links don't work

**Solution: Handle initial route**
```typescript
// App.tsx
import { useEffect } from 'react';
import { Stack, useFlow } from './stackflow';

function App() {
  const { push } = useFlow();
  
  useEffect(() => {
    const path = window.location.pathname;
    
    if (path.startsWith('/detail/')) {
      const id = path.split('/')[2];
      push('Detail', { id });
    } else if (path === '/profile') {
      push('Profile', {});
    }
  }, [push]);
  
  return <Stack />;
}
```

## Styling Issues

### Problem: Activity content is cut off or not scrollable

**Solution: Set proper height and overflow**
```css
.activity-container {
  height: 100vh;
  height: 100dvh; /* Dynamic viewport height for mobile */
  overflow-y: auto;
  -webkit-overflow-scrolling: touch; /* Smooth scrolling on iOS */
}
```

### Problem: Safe area not respected on notched devices

**Solution: Use safe-area-inset**
```css
.activity {
  padding-top: max(16px, env(safe-area-inset-top));
  padding-bottom: max(16px, env(safe-area-inset-bottom));
  padding-left: max(16px, env(safe-area-inset-left));
  padding-right: max(16px, env(safe-area-inset-right));
}
```

### Problem: Transitions show white flash between activities

**Solution: Set background on Stack container**
```css
/* Add this to your global styles */
[data-stackflow-root] {
  background: #ffffff; /* or your app's background color */
}
```

## Mobile-Specific Issues

### Problem: Touch events not working properly

**Solution: Use proper touch event handlers**
```typescript
const MyActivity = () => {
  const handleTouchStart = (e: TouchEvent) => {
    e.stopPropagation(); // Prevent parent handlers
  };
  
  return (
    <div onTouchStart={handleTouchStart}>
      Swipeable content
    </div>
  );
};
```

### Problem: Viewport height issues on mobile browsers

**Solution: Use dynamic viewport units**
```css
.full-height {
  /* Use dvh instead of vh */
  height: 100dvh; /* Dynamic viewport height */
  min-height: 100dvh;
}

/* Fallback for older browsers */
@supports not (height: 100dvh) {
  .full-height {
    height: 100vh;
    min-height: 100vh;
  }
}
```

### Problem: Transitions feel slow on mobile

**Solution: Reduce transition duration for mobile**
```typescript
const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);

export const { Stack, useFlow } = stackflow({
  transitionDuration: isMobile ? 300 : 350,
  activities: {...},
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});
```

## Plugin Issues

### Problem: Plugin not working after installation

**Solution 1: Check import and registration**
```typescript
// ✅ Correct: Import and register plugin
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';
import '@stackflow/plugin-basic-ui/index.css'; // Don't forget CSS!

export const { Stack, useFlow } = stackflow({
  activities: {...},
  plugins: [
    basicUIPlugin({ theme: 'cupertino' }) // Register here
  ],
});

// ❌ Wrong: Plugin imported but not registered
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

export const { Stack, useFlow } = stackflow({
  activities: {...},
  plugins: [], // Plugin not added!
});
```

**Solution 2: Check plugin configuration**
```typescript
// Some plugins require specific configuration
basicUIPlugin({
  theme: 'cupertino', // Required: must be 'cupertino' or 'android'
})
```

### Problem: Custom plugin not working

**Solution: Follow plugin structure**
```typescript
import { StackflowPlugin } from '@stackflow/react';

const myCustomPlugin: StackflowPlugin = () => ({
  key: 'my-custom-plugin',
  onInit: ({ actions }) => {
    console.log('Plugin initialized');
  },
  wrapStack: ({ stack }) => {
    // Wrap the entire stack
    return <div className="custom-wrapper">{stack}</div>;
  },
  wrapActivity: ({ activity }) => {
    // Wrap each activity
    return <div className="activity-wrapper">{activity}</div>;
  },
});

export const { Stack, useFlow } = stackflow({
  activities: {...},
  plugins: [
    basicUIPlugin({ theme: 'cupertino' }),
    myCustomPlugin(),
  ],
});
```

## Common Error Messages

### Error: "Cannot read property 'push' of undefined"

**Cause:** useFlow() called outside of Stack context

**Solution:**
```typescript
// ✅ Correct: useFlow() inside component rendered by Stack
const MyActivity = () => {
  const { push } = useFlow(); // Works!
  return <button onClick={() => push('Detail', { id: '1' })}>Go</button>;
};

// ❌ Wrong: useFlow() outside Stack context
const { push } = useFlow(); // Error! Not inside Stack

function App() {
  return <Stack />;
}
```

### Error: "Activity 'DetailActivity' is not registered"

**Cause:** Activity name in push() doesn't match registered name

**Solution:**
```typescript
// Make sure names match exactly
export const { Stack, useFlow } = stackflow({
  activities: {
    Detail: DetailActivity, // Name is "Detail"
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});

// Use the exact registered name
const { push } = useFlow();
push('Detail', { id: '123' }); // ✅ Correct
push('DetailActivity', { id: '123' }); // ❌ Wrong - not registered as "DetailActivity"
```

### Error: "Maximum update depth exceeded"

**Cause:** Infinite render loop, often from navigation in render

**Solution:**
```typescript
// ❌ Wrong: Navigation in render causes loop
const MyActivity = () => {
  const { push } = useFlow();
  push('OtherActivity', {}); // Called on every render!
  return <div>Content</div>;
};

// ✅ Correct: Navigation in event handler or useEffect
const MyActivity = () => {
  const { push } = useFlow();
  
  const handleClick = () => {
    push('OtherActivity', {});
  };
  
  return <button onClick={handleClick}>Go</button>;
};

// ✅ Also correct: Conditional navigation in useEffect
const MyActivity = () => {
  const { push } = useFlow();
  const isLoggedIn = useAuth();
  
  useEffect(() => {
    if (!isLoggedIn) {
      push('Login', {});
    }
  }, [isLoggedIn, push]);
  
  return <div>Content</div>;
};
```

## Getting More Help

If you're still experiencing issues:

1. **Check official documentation**: https://stackflow.so
2. **Read LLM docs**: https://stackflow.so/llms-full.txt
3. **Check changelog**: https://stackflow.so/llms-changelog.txt for breaking changes
4. **GitHub Issues**: https://github.com/daangn/stackflow/issues
5. **Enable verbose logging**:
```typescript
// Add this for debugging
if (process.env.NODE_ENV === 'development') {
  window.__STACKFLOW_DEBUG__ = true;
}
```

## Debug Checklist

When troubleshooting, check:
- [ ] All required packages installed
- [ ] Stack component is rendered
- [ ] Activities are registered in config
- [ ] Plugin CSS is imported
- [ ] useFlow() is from your stackflow.ts, not the package
- [ ] Parameter types match between push() and activity
- [ ] Navigation is in event handler, not render
- [ ] No CSS conflicts overriding Stackflow styles
- [ ] Console has no errors
- [ ] React DevTools shows correct component tree

---

For more detailed information, always refer to the official Stackflow documentation at https://stackflow.so
