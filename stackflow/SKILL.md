# Stackflow Navigation Framework

## Metadata
---
name: stackflow
description: Create a product detail activity with Stackflow
---

## Overview
Stackflow is a mobile-first navigation framework created by Karrot (당근마켓) that enables native app-like navigation experiences in React web applications. It provides stack-based transitions and animations similar to iOS and Android native apps.

## When to Use This Skill
- Building mobile web applications with React
- Implementing native-like screen transitions and navigation
- Creating stack-based navigation flows (push/pop)
- Developing mobile-first user experiences with smooth animations
- Working with React projects that require iOS/Android-style navigation

## Core Concepts

### Stack-Based Navigation
Stackflow implements a stack-based navigation model where:
- **Activities** are individual screens/pages in your application
- Screens are pushed onto and popped from a navigation stack
- Each activity can have its own state and lifecycle
- Browser history integrates seamlessly with the stack

### Key Features
- **Native-like Transitions**: iOS and Android style screen transitions
- **Type Safety**: Full TypeScript support with type-safe navigation
- **History Integration**: Works naturally with browser back/forward buttons
- **Plugin System**: Extensible architecture for custom behaviors
- **React-First**: Built specifically for React applications

## Official Documentation

Stackflow provides comprehensive LLM-optimized documentation:

- **[llms-full.txt](https://stackflow.so/llms-full.txt)**: Complete documentation of Stackflow
- **[llms-changelog.txt](https://stackflow.so/llms-changelog.txt)**: Latest updates and version history

**Important**: Always fetch the latest documentation from these URLs before implementing features to ensure accuracy and up-to-date information.

## Installation

```bash
npm install @stackflow/react @stackflow/plugin-basic-ui
# or
yarn add @stackflow/react @stackflow/plugin-basic-ui
# or
pnpm add @stackflow/react @stackflow/plugin-basic-ui
```

## Quick Start

### 1. Define Activities
Activities are React components that represent screens:

```typescript
// activities/HomePage.tsx
import { ActivityComponentType } from '@stackflow/react';

const HomePage: ActivityComponentType = () => {
  const { push } = useFlow();
  
  return (
    <div>
      <h1>Home</h1>
      <button onClick={() => push('DetailPage', { id: '123' })}>
        Go to Detail
      </button>
    </div>
  );
};

export default HomePage;
```

### 2. Create Stack Configuration
```typescript
// stackflow.ts
import { stackflow } from '@stackflow/react';
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

import HomePage from './activities/HomePage';
import DetailPage from './activities/DetailPage';

export const { Stack, useFlow } = stackflow({
  transitionDuration: 350,
  activities: {
    HomePage,
    DetailPage,
  },
  plugins: [
    basicUIPlugin({
      theme: 'cupertino', // or 'android'
    }),
  ],
});
```

### 3. Use in Application
```typescript
// App.tsx
import { Stack } from './stackflow';

function App() {
  return <Stack />;
}

export default App;
```

## Navigation API

### Basic Navigation
```typescript
import { useFlow } from './stackflow';

const MyComponent = () => {
  const { push, pop, replace } = useFlow();
  
  // Push new activity onto stack
  push('ActivityName', { param1: 'value' });
  
  // Pop current activity from stack
  pop();
  
  // Replace current activity
  replace('ActivityName', { param1: 'value' });
};
```

### Type-Safe Parameters
```typescript
import { ActivityComponentType } from '@stackflow/react';

type DetailPageParams = {
  id: string;
  tab?: 'info' | 'reviews';
};

const DetailPage: ActivityComponentType<DetailPageParams> = ({ params }) => {
  const { id, tab = 'info' } = params;
  
  return <div>Detail for {id} - {tab}</div>;
};

export default DetailPage;
```

### Activity Lifecycle
```typescript
import { useActivity } from '@stackflow/react';

const MyActivity = () => {
  const activity = useActivity();
  
  useEffect(() => {
    if (activity.isActive) {
      // Activity is currently visible and interactive
      console.log('Activity is active');
    }
  }, [activity.isActive]);
  
  return <div>My Activity</div>;
};
```

## Plugins

### Basic UI Plugin
Provides iOS (Cupertino) and Android (Material) style transitions:

```typescript
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

basicUIPlugin({
  theme: 'cupertino', // iOS-style transitions
  // or
  theme: 'android',   // Android-style transitions
})
```

### History Plugin
Syncs with browser history for proper back/forward button support:

```typescript
import { basicRendererPlugin } from '@stackflow/plugin-renderer-basic';

basicRendererPlugin()
```

## Additional Documentation

For detailed implementation guidance, refer to these supplementary documents:

- [BEST_PRACTICES.md](BEST_PRACTICES.md): Comprehensive guide on architecture, performance, and patterns
- [EXAMPLES.md](EXAMPLES.md): Real-world code examples and common use cases
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md): Solutions to common issues and problems

## Project Structure

Recommended structure for Stackflow projects:

```
src/
├── stackflow.ts          # Stack configuration
├── activities/           # Screen components (Activities)
│   ├── HomePage.tsx
│   ├── DetailPage.tsx
│   └── SettingsPage.tsx
├── components/           # Reusable UI components
│   ├── Header.tsx
│   ├── Button.tsx
│   └── Card.tsx
├── hooks/               # Custom React hooks
├── utils/               # Utility functions
└── App.tsx              # Root component
```

## Resources

- **Official Website**: https://stackflow.so
- **GitHub Repository**: https://github.com/daangn/stackflow
- **LLM Documentation**: 
  - Full docs: https://stackflow.so/llms-full.txt
  - Changelog: https://stackflow.so/llms-changelog.txt

## Integration Notes for Claude Code

When working with Stackflow in Claude Code:

1. **Always fetch latest documentation** from llms-full.txt before major implementations
2. **Check changelog** for recent updates or breaking changes
3. **Follow React and TypeScript best practices**
4. **Test navigation flows** in mobile viewports (375px, 414px width)
5. **Use type-safe navigation** for compile-time safety
6. **Refer to BEST_PRACTICES.md** for architecture guidance
7. **Check EXAMPLES.md** for implementation patterns
8. **Consult TROUBLESHOOTING.md** if issues arise
---