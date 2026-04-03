# Stackflow Best Practices

This document provides comprehensive guidelines for building robust, maintainable, and performant applications with Stackflow.

## Architecture Patterns

### Activity Design

#### One Activity Per Screen
Each activity should represent a distinct screen or view in your application:

```typescript
// ✅ Good: Each screen is its own activity
activities: {
  ProductList,
  ProductDetail,
  ProductReview,
  Cart,
  Checkout,
}

// ❌ Bad: Trying to handle multiple screens in one activity
activities: {
  Products, // Handles both list and detail
  Shopping, // Handles both cart and checkout
}
```

#### Keep Activities Focused
Activities should be container components that compose smaller UI components:

```typescript
// ✅ Good: Activity composes smaller components
const ProductDetailActivity: ActivityComponentType<{ id: string }> = ({ params }) => {
  const product = useProduct(params.id);
  
  return (
    <ActivityLayout>
      <ProductHeader product={product} />
      <ProductImages images={product.images} />
      <ProductInfo product={product} />
      <ProductActions product={product} />
    </ActivityLayout>
  );
};

// ❌ Bad: All logic and UI in one activity
const ProductDetailActivity = ({ params }) => {
  // 500 lines of component logic and JSX...
};
```

#### Separate Business Logic from UI
Use custom hooks to encapsulate business logic:

```typescript
// hooks/useProductDetail.ts
export const useProductDetail = (productId: string) => {
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchProduct(productId).then(setProduct).finally(() => setLoading(false));
  }, [productId]);
  
  return { product, loading };
};

// activities/ProductDetailActivity.tsx
const ProductDetailActivity: ActivityComponentType<{ id: string }> = ({ params }) => {
  const { product, loading } = useProductDetail(params.id);
  const { push } = useFlow();
  
  if (loading) return <LoadingSpinner />;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <button onClick={() => push('ReviewActivity', { productId: params.id })}>
        Write Review
      </button>
    </div>
  );
};
```

### Type Safety

#### Always Define Parameter Types
```typescript
// ✅ Good: Explicit parameter types
type UserProfileParams = {
  userId: string;
  tab?: 'posts' | 'followers' | 'following';
  highlightId?: string;
};

const UserProfileActivity: ActivityComponentType<UserProfileParams> = ({ params }) => {
  // params is fully typed
};

// ❌ Bad: No parameter types
const UserProfileActivity: ActivityComponentType = ({ params }) => {
  // params is 'any'
};
```

#### Use Type-Safe Navigation
```typescript
// stackflow.ts - Create typed navigation
type TypedActivities = {
  Home: {};
  UserProfile: { userId: string; tab?: 'posts' | 'followers' };
  PostDetail: { postId: string };
};

export const { Stack, useFlow } = stackflow<TypedActivities>({
  activities: {
    Home: HomePage,
    UserProfile: UserProfileActivity,
    PostDetail: PostDetailActivity,
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});

// Usage - TypeScript will check parameters
const MyComponent = () => {
  const { push } = useFlow();
  
  // ✅ Type-safe: TypeScript validates parameters
  push('UserProfile', { userId: '123', tab: 'posts' });
  
  // ❌ Type error: missing required 'userId'
  push('UserProfile', { tab: 'posts' });
  
  // ❌ Type error: invalid tab value
  push('UserProfile', { userId: '123', tab: 'invalid' });
};
```

#### Validate Parameters at Runtime
For additional safety, validate parameters within activities:

```typescript
import { z } from 'zod';

const ParamsSchema = z.object({
  userId: z.string().min(1),
  tab: z.enum(['posts', 'followers', 'following']).optional(),
});

const UserProfileActivity: ActivityComponentType<UserProfileParams> = ({ params }) => {
  // Validate at runtime
  const validatedParams = ParamsSchema.parse(params);
  
  // Continue with validated params
};
```

## Performance Optimization

### Lazy Load Activities
Use React.lazy() for code splitting to reduce initial bundle size:

```typescript
// stackflow.ts
import { lazy } from 'react';

const HomePage = lazy(() => import('./activities/HomePage'));
const ProductDetail = lazy(() => import('./activities/ProductDetail'));
const UserProfile = lazy(() => import('./activities/UserProfile'));

export const { Stack, useFlow } = stackflow({
  activities: {
    Home: HomePage,
    ProductDetail,
    UserProfile,
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});

// App.tsx - Wrap Stack with Suspense
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

### Optimize Transition Durations
Keep transition durations reasonable for perceived performance:

```typescript
// ✅ Good: 300-400ms feels snappy
export const { Stack, useFlow } = stackflow({
  transitionDuration: 350,
  // ...
});

// ⚠️ Caution: Too slow feels sluggish
export const { Stack, useFlow } = stackflow({
  transitionDuration: 800, // Feels slow
  // ...
});

// ⚠️ Caution: Too fast feels jarring
export const { Stack, useFlow } = stackflow({
  transitionDuration: 100, // Too abrupt
  // ...
});
```

### Minimize Activity Re-renders
Use proper React optimization techniques:

```typescript
import { memo, useCallback, useMemo } from 'react';

// Memoize expensive computations
const ProductListActivity = () => {
  const products = useProducts();
  
  const sortedProducts = useMemo(() => {
    return products.sort((a, b) => b.rating - a.rating);
  }, [products]);
  
  return <ProductGrid products={sortedProducts} />;
};

// Memoize callbacks passed to child components
const ProductDetailActivity = ({ params }) => {
  const { push } = useFlow();
  
  const handleReviewClick = useCallback(() => {
    push('ReviewActivity', { productId: params.id });
  }, [push, params.id]);
  
  return <ProductView onReviewClick={handleReviewClick} />;
};

// Memoize components that don't need frequent updates
const ProductCard = memo(({ product, onClick }) => {
  return (
    <div onClick={onClick}>
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  );
});
```

### Preload Data for Next Activity
Improve perceived performance by preloading data:

```typescript
const ProductListActivity = () => {
  const { push } = useFlow();
  const products = useProducts();
  
  const handleProductClick = async (productId: string) => {
    // Start loading product data
    const productPromise = fetchProduct(productId);
    
    // Navigate immediately
    push('ProductDetail', { id: productId });
    
    // Data will be ready or loading when activity renders
  };
  
  return (
    <div>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onClick={() => handleProductClick(product.id)}
        />
      ))}
    </div>
  );
};
```

## Styling Best Practices

### Use Scoped Styles
Keep styles scoped to prevent conflicts:

```typescript
// ✅ Good: CSS Modules
import styles from './ProductDetail.module.css';

const ProductDetailActivity = () => {
  return <div className={styles.container}>...</div>;
};

// ✅ Good: Styled Components
import styled from 'styled-components';

const Container = styled.div`
  padding: 16px;
`;

const ProductDetailActivity = () => {
  return <Container>...</Container>;
};

// ❌ Bad: Global styles that can conflict
// styles.css
.container { padding: 16px; } // Can conflict with other .container classes
```

### Respect Safe Areas
Account for device notches and safe areas:

```css
/* ProductDetail.module.css */
.container {
  /* Use safe area insets */
  padding-top: max(16px, env(safe-area-inset-top));
  padding-bottom: max(16px, env(safe-area-inset-bottom));
  padding-left: max(16px, env(safe-area-inset-left));
  padding-right: max(16px, env(safe-area-inset-right));
}

/* For full-screen activities */
.fullscreen {
  height: 100vh;
  height: 100dvh; /* Dynamic viewport height for mobile */
}
```

### Consistent Activity Layout
Create a reusable layout component for consistent spacing:

```typescript
// components/ActivityLayout.tsx
import styled from 'styled-components';

const Container = styled.div`
  min-height: 100vh;
  min-height: 100dvh;
  padding: max(16px, env(safe-area-inset-top)) 
           max(16px, env(safe-area-inset-right)) 
           max(16px, env(safe-area-inset-bottom)) 
           max(16px, env(safe-area-inset-left));
  background: var(--background-color);
`;

export const ActivityLayout = ({ children }) => {
  return <Container>{children}</Container>;
};

// Usage in activities
const ProductDetailActivity = () => {
  return (
    <ActivityLayout>
      <ProductHeader />
      <ProductContent />
    </ActivityLayout>
  );
};
```

### Handle Different Orientations
Test and optimize for both portrait and landscape:

```css
/* Responsive layout */
.productGrid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

@media (orientation: landscape) {
  .productGrid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (min-width: 768px) {
  .productGrid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

## State Management

### Activity-Level State
Use local state for activity-specific data:

```typescript
const ProductDetailActivity: ActivityComponentType<{ id: string }> = ({ params }) => {
  const [selectedImage, setSelectedImage] = useState(0);
  const [quantity, setQuantity] = useState(1);
  
  // State is scoped to this activity
  return (
    <div>
      <ProductImages 
        selected={selectedImage} 
        onSelect={setSelectedImage} 
      />
      <QuantitySelector 
        value={quantity} 
        onChange={setQuantity} 
      />
    </div>
  );
};
```

### Global State
Use context or external state management for shared data:

```typescript
// context/CartContext.tsx
export const CartProvider = ({ children }) => {
  const [items, setItems] = useState([]);
  
  const addToCart = (product, quantity) => {
    setItems(prev => [...prev, { product, quantity }]);
  };
  
  return (
    <CartContext.Provider value={{ items, addToCart }}>
      {children}
    </CartContext.Provider>
  );
};

// App.tsx
function App() {
  return (
    <CartProvider>
      <Stack />
    </CartProvider>
  );
}

// Usage in activities
const ProductDetailActivity = ({ params }) => {
  const { addToCart } = useCart();
  const { push } = useFlow();
  
  const handleAddToCart = () => {
    addToCart(product, quantity);
    push('Cart');
  };
};
```

### Persist State Across Navigation
Use activity params to pass state between activities:

```typescript
// Passing filter state
const ProductListActivity = () => {
  const [category, setCategory] = useState('all');
  const { push } = useFlow();
  
  const handleSearch = (query: string) => {
    push('SearchResults', { 
      query, 
      category, // Pass current category to search
    });
  };
};

// Receiving and using state
type SearchResultsParams = {
  query: string;
  category: string;
};

const SearchResultsActivity: ActivityComponentType<SearchResultsParams> = ({ params }) => {
  const { query, category } = params;
  const results = useSearch(query, category);
  
  return <SearchResultsList results={results} />;
};
```

## Testing

### Test Navigation Flows
```typescript
import { renderHook, act } from '@testing-library/react';
import { useFlow } from './stackflow';

describe('Navigation', () => {
  it('should navigate to product detail', () => {
    const { result } = renderHook(() => useFlow());
    
    act(() => {
      result.current.push('ProductDetail', { id: '123' });
    });
    
    // Assert navigation occurred
  });
  
  it('should go back on pop', () => {
    const { result } = renderHook(() => useFlow());
    
    act(() => {
      result.current.push('ProductDetail', { id: '123' });
      result.current.pop();
    });
    
    // Assert returned to previous activity
  });
});
```

### Test Activities
```typescript
import { render, screen } from '@testing-library/react';
import ProductDetailActivity from './ProductDetailActivity';

describe('ProductDetailActivity', () => {
  it('should render product information', () => {
    const params = { id: '123' };
    
    render(<ProductDetailActivity params={params} />);
    
    expect(screen.getByText(/product name/i)).toBeInTheDocument();
  });
});
```

### Test in Mobile Viewports
```typescript
// Use viewport testing
describe('Mobile Layout', () => {
  beforeEach(() => {
    // Set mobile viewport
    global.innerWidth = 375;
    global.innerHeight = 667;
  });
  
  it('should display mobile layout', () => {
    render(<ProductListActivity />);
    // Assert mobile-specific layout
  });
});
```

## Error Handling

### Handle Navigation Errors
```typescript
const ProductDetailActivity: ActivityComponentType<{ id: string }> = ({ params }) => {
  const { data: product, error, isLoading } = useProduct(params.id);
  const { pop } = useFlow();
  
  if (error) {
    return (
      <ErrorView 
        message="Failed to load product" 
        onRetry={() => window.location.reload()}
        onBack={() => pop()}
      />
    );
  }
  
  if (isLoading) {
    return <LoadingView />;
  }
  
  return <ProductView product={product} />;
};
```

### Validate Activity Access
```typescript
const CheckoutActivity: ActivityComponentType = () => {
  const { items } = useCart();
  const { replace } = useFlow();
  
  useEffect(() => {
    // Redirect if cart is empty
    if (items.length === 0) {
      replace('Cart');
    }
  }, [items, replace]);
  
  if (items.length === 0) {
    return null; // Will redirect
  }
  
  return <CheckoutForm items={items} />;
};
```

## Accessibility

### Keyboard Navigation
```typescript
const ProductListActivity = () => {
  const { push } = useFlow();
  
  const handleKeyPress = (e: KeyboardEvent, productId: string) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      push('ProductDetail', { id: productId });
    }
  };
  
  return (
    <div>
      {products.map(product => (
        <div
          key={product.id}
          role="button"
          tabIndex={0}
          onClick={() => push('ProductDetail', { id: product.id })}
          onKeyPress={(e) => handleKeyPress(e, product.id)}
        >
          {product.name}
        </div>
      ))}
    </div>
  );
};
```

### Screen Reader Support
```typescript
const ProductDetailActivity = ({ params }) => {
  const { push } = useFlow();
  
  return (
    <div>
      <button
        onClick={() => push('ReviewActivity', { productId: params.id })}
        aria-label={`Write a review for ${product.name}`}
      >
        Write Review
      </button>
    </div>
  );
};
```

### Focus Management
```typescript
const SearchActivity = () => {
  const inputRef = useRef<HTMLInputElement>(null);
  const activity = useActivity();
  
  useEffect(() => {
    // Focus input when activity becomes active
    if (activity.isActive && inputRef.current) {
      inputRef.current.focus();
    }
  }, [activity.isActive]);
  
  return <input ref={inputRef} type="search" />;
};
```

---

These best practices will help you build robust, maintainable, and performant Stackflow applications. Always refer to the official documentation for the latest updates and recommendations.
