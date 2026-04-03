# Stackflow Examples

This document provides real-world code examples and common use cases for Stackflow.

## Table of Contents
- [Basic Navigation Flow](#basic-navigation-flow)
- [E-commerce App](#e-commerce-app)
- [Social Media Feed](#social-media-feed)
- [Modal-like Flows](#modal-like-flows)
- [Tab Navigation with Stackflow](#tab-navigation-with-stackflow)
- [Form Wizard](#form-wizard)
- [Authentication Flow](#authentication-flow)
- [Deep Linking](#deep-linking)

## Basic Navigation Flow

### Simple List to Detail Navigation

```typescript
// stackflow.ts
import { stackflow } from '@stackflow/react';
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

type TypedActivities = {
  ItemList: {};
  ItemDetail: { itemId: string };
};

export const { Stack, useFlow } = stackflow<TypedActivities>({
  transitionDuration: 350,
  activities: {
    ItemList: () => import('./activities/ItemListActivity'),
    ItemDetail: () => import('./activities/ItemDetailActivity'),
  },
  plugins: [
    basicUIPlugin({
      theme: 'cupertino',
    }),
  ],
});

// activities/ItemListActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';

const ItemListActivity: ActivityComponentType = () => {
  const { push } = useFlow();
  const items = useItems(); // Custom hook to fetch items
  
  return (
    <div className="item-list">
      <h1>Items</h1>
      <div className="grid">
        {items.map(item => (
          <div
            key={item.id}
            className="item-card"
            onClick={() => push('ItemDetail', { itemId: item.id })}
          >
            <img src={item.thumbnail} alt={item.name} />
            <h3>{item.name}</h3>
            <p>${item.price}</p>
          </div>
        ))}
      </div>
    </div>
  );
};

export default ItemListActivity;

// activities/ItemDetailActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';

type ItemDetailParams = {
  itemId: string;
};

const ItemDetailActivity: ActivityComponentType<ItemDetailParams> = ({ params }) => {
  const { pop } = useFlow();
  const item = useItem(params.itemId); // Custom hook to fetch item
  
  if (!item) return <div>Loading...</div>;
  
  return (
    <div className="item-detail">
      <button onClick={() => pop()} className="back-button">
        ← Back
      </button>
      <img src={item.image} alt={item.name} />
      <h1>{item.name}</h1>
      <p className="price">${item.price}</p>
      <p className="description">{item.description}</p>
    </div>
  );
};

export default ItemDetailActivity;
```

## E-commerce App

### Complete Shopping Flow

```typescript
// stackflow.ts
import { stackflow } from '@stackflow/react';
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

type TypedActivities = {
  Home: {};
  ProductList: { category?: string };
  ProductDetail: { productId: string };
  Cart: {};
  Checkout: {};
  OrderConfirmation: { orderId: string };
};

export const { Stack, useFlow } = stackflow<TypedActivities>({
  transitionDuration: 350,
  activities: {
    Home: () => import('./activities/HomeActivity'),
    ProductList: () => import('./activities/ProductListActivity'),
    ProductDetail: () => import('./activities/ProductDetailActivity'),
    Cart: () => import('./activities/CartActivity'),
    Checkout: () => import('./activities/CheckoutActivity'),
    OrderConfirmation: () => import('./activities/OrderConfirmationActivity'),
  },
  plugins: [
    basicUIPlugin({
      theme: 'cupertino',
    }),
  ],
});

// activities/HomeActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';

const HomeActivity: ActivityComponentType = () => {
  const { push } = useFlow();
  const categories = ['Electronics', 'Clothing', 'Books', 'Home'];
  
  return (
    <div className="home">
      <header>
        <h1>Shop</h1>
        <button onClick={() => push('Cart')}>
          🛒 Cart
        </button>
      </header>
      
      <section className="hero">
        <h2>Welcome to Our Store</h2>
        <button onClick={() => push('ProductList', {})}>
          Browse All Products
        </button>
      </section>
      
      <section className="categories">
        <h3>Shop by Category</h3>
        <div className="category-grid">
          {categories.map(category => (
            <button
              key={category}
              onClick={() => push('ProductList', { category })}
              className="category-card"
            >
              {category}
            </button>
          ))}
        </div>
      </section>
    </div>
  );
};

export default HomeActivity;

// activities/ProductDetailActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useCart } from '../context/CartContext';
import { useState } from 'react';

type ProductDetailParams = {
  productId: string;
};

const ProductDetailActivity: ActivityComponentType<ProductDetailParams> = ({ params }) => {
  const { push, pop } = useFlow();
  const { addToCart } = useCart();
  const product = useProduct(params.productId);
  const [quantity, setQuantity] = useState(1);
  const [selectedImage, setSelectedImage] = useState(0);
  
  if (!product) return <div>Loading...</div>;
  
  const handleAddToCart = () => {
    addToCart(product, quantity);
    push('Cart');
  };
  
  return (
    <div className="product-detail">
      <button onClick={() => pop()} className="back-button">
        ← Back
      </button>
      
      <div className="image-gallery">
        <img 
          src={product.images[selectedImage]} 
          alt={product.name}
          className="main-image"
        />
        <div className="thumbnails">
          {product.images.map((image, index) => (
            <img
              key={index}
              src={image}
              alt={`${product.name} ${index + 1}`}
              onClick={() => setSelectedImage(index)}
              className={selectedImage === index ? 'active' : ''}
            />
          ))}
        </div>
      </div>
      
      <div className="product-info">
        <h1>{product.name}</h1>
        <p className="price">${product.price}</p>
        <p className="description">{product.description}</p>
        
        <div className="quantity-selector">
          <label>Quantity:</label>
          <button onClick={() => setQuantity(Math.max(1, quantity - 1))}>-</button>
          <span>{quantity}</span>
          <button onClick={() => setQuantity(quantity + 1)}>+</button>
        </div>
        
        <button onClick={handleAddToCart} className="add-to-cart">
          Add to Cart
        </button>
      </div>
    </div>
  );
};

export default ProductDetailActivity;

// activities/CartActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useCart } from '../context/CartContext';

const CartActivity: ActivityComponentType = () => {
  const { push, pop } = useFlow();
  const { items, removeFromCart, updateQuantity } = useCart();
  
  const total = items.reduce((sum, item) => sum + item.product.price * item.quantity, 0);
  
  if (items.length === 0) {
    return (
      <div className="cart empty">
        <button onClick={() => pop()}>← Back</button>
        <h1>Your Cart is Empty</h1>
        <p>Add some products to get started!</p>
        <button onClick={() => push('Home')}>
          Continue Shopping
        </button>
      </div>
    );
  }
  
  return (
    <div className="cart">
      <header>
        <button onClick={() => pop()}>← Back</button>
        <h1>Shopping Cart</h1>
      </header>
      
      <div className="cart-items">
        {items.map(item => (
          <div key={item.product.id} className="cart-item">
            <img src={item.product.thumbnail} alt={item.product.name} />
            <div className="item-info">
              <h3>{item.product.name}</h3>
              <p className="price">${item.product.price}</p>
              <div className="quantity-controls">
                <button onClick={() => updateQuantity(item.product.id, item.quantity - 1)}>
                  -
                </button>
                <span>{item.quantity}</span>
                <button onClick={() => updateQuantity(item.product.id, item.quantity + 1)}>
                  +
                </button>
              </div>
            </div>
            <button 
              onClick={() => removeFromCart(item.product.id)}
              className="remove-button"
            >
              🗑️
            </button>
          </div>
        ))}
      </div>
      
      <div className="cart-summary">
        <div className="total">
          <span>Total:</span>
          <span className="amount">${total.toFixed(2)}</span>
        </div>
        <button 
          onClick={() => push('Checkout')} 
          className="checkout-button"
        >
          Proceed to Checkout
        </button>
      </div>
    </div>
  );
};

export default CartActivity;
```

## Social Media Feed

### Feed with Comments

```typescript
// stackflow.ts
type TypedActivities = {
  Feed: {};
  PostDetail: { postId: string };
  UserProfile: { userId: string };
  CreatePost: {};
  EditPost: { postId: string };
};

// activities/FeedActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useInfiniteScroll } from '../hooks/useInfiniteScroll';

const FeedActivity: ActivityComponentType = () => {
  const { push } = useFlow();
  const { posts, loadMore, hasMore } = useInfiniteScroll();
  const lastPostRef = useRef<HTMLDivElement>(null);
  
  // Infinite scroll setup
  useEffect(() => {
    const observer = new IntersectionObserver(
      entries => {
        if (entries[0].isIntersecting && hasMore) {
          loadMore();
        }
      },
      { threshold: 0.1 }
    );
    
    if (lastPostRef.current) {
      observer.observe(lastPostRef.current);
    }
    
    return () => observer.disconnect();
  }, [hasMore, loadMore]);
  
  return (
    <div className="feed">
      <header>
        <h1>Feed</h1>
        <button onClick={() => push('CreatePost')}>
          ✏️ New Post
        </button>
      </header>
      
      <div className="posts">
        {posts.map((post, index) => (
          <div
            key={post.id}
            ref={index === posts.length - 1 ? lastPostRef : null}
            className="post-card"
            onClick={() => push('PostDetail', { postId: post.id })}
          >
            <div className="post-header">
              <img 
                src={post.author.avatar} 
                alt={post.author.name}
                onClick={(e) => {
                  e.stopPropagation();
                  push('UserProfile', { userId: post.author.id });
                }}
              />
              <div>
                <h3>{post.author.name}</h3>
                <span className="timestamp">{post.createdAt}</span>
              </div>
            </div>
            
            <p className="post-content">{post.content}</p>
            
            {post.image && (
              <img src={post.image} alt="Post" className="post-image" />
            )}
            
            <div className="post-actions">
              <button>❤️ {post.likes}</button>
              <button>💬 {post.comments}</button>
              <button>🔗 Share</button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};

export default FeedActivity;

// activities/PostDetailActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useState } from 'react';

type PostDetailParams = {
  postId: string;
};

const PostDetailActivity: ActivityComponentType<PostDetailParams> = ({ params }) => {
  const { push, pop } = useFlow();
  const post = usePost(params.postId);
  const comments = useComments(params.postId);
  const [newComment, setNewComment] = useState('');
  const { addComment } = useCommentMutations();
  
  const handleSubmitComment = async () => {
    if (!newComment.trim()) return;
    
    await addComment(params.postId, newComment);
    setNewComment('');
  };
  
  if (!post) return <div>Loading...</div>;
  
  return (
    <div className="post-detail">
      <header>
        <button onClick={() => pop()}>← Back</button>
        <h1>Post</h1>
      </header>
      
      <div className="post">
        <div className="post-header">
          <img 
            src={post.author.avatar} 
            alt={post.author.name}
            onClick={() => push('UserProfile', { userId: post.author.id })}
          />
          <div>
            <h3>{post.author.name}</h3>
            <span className="timestamp">{post.createdAt}</span>
          </div>
        </div>
        
        <p className="post-content">{post.content}</p>
        
        {post.image && (
          <img src={post.image} alt="Post" className="post-image" />
        )}
        
        <div className="post-stats">
          <span>{post.likes} likes</span>
          <span>{post.comments} comments</span>
        </div>
      </div>
      
      <div className="comments">
        <h2>Comments</h2>
        {comments.map(comment => (
          <div key={comment.id} className="comment">
            <img 
              src={comment.author.avatar} 
              alt={comment.author.name}
              onClick={() => push('UserProfile', { userId: comment.author.id })}
            />
            <div>
              <h4>{comment.author.name}</h4>
              <p>{comment.content}</p>
              <span className="timestamp">{comment.createdAt}</span>
            </div>
          </div>
        ))}
      </div>
      
      <div className="comment-input">
        <input
          type="text"
          placeholder="Write a comment..."
          value={newComment}
          onChange={(e) => setNewComment(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && handleSubmitComment()}
        />
        <button onClick={handleSubmitComment}>Post</button>
      </div>
    </div>
  );
};

export default PostDetailActivity;
```

## Modal-like Flows

### Confirmation Dialog

```typescript
// activities/ConfirmDialogActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';

type ConfirmDialogParams = {
  title: string;
  message: string;
  confirmText?: string;
  cancelText?: string;
  onConfirm?: () => void;
};

const ConfirmDialogActivity: ActivityComponentType<ConfirmDialogParams> = ({ params }) => {
  const { pop } = useFlow();
  
  const handleConfirm = () => {
    params.onConfirm?.();
    pop();
  };
  
  const handleCancel = () => {
    pop();
  };
  
  return (
    <div className="modal-overlay">
      <div className="dialog">
        <h2>{params.title}</h2>
        <p>{params.message}</p>
        <div className="dialog-actions">
          <button onClick={handleCancel} className="cancel">
            {params.cancelText || 'Cancel'}
          </button>
          <button onClick={handleConfirm} className="confirm">
            {params.confirmText || 'Confirm'}
          </button>
        </div>
      </div>
    </div>
  );
};

export default ConfirmDialogActivity;

// Usage in another activity
const ProductDetailActivity = ({ params }) => {
  const { push } = useFlow();
  const { deleteProduct } = useProductMutations();
  
  const handleDelete = () => {
    push('ConfirmDialog', {
      title: 'Delete Product',
      message: 'Are you sure you want to delete this product?',
      confirmText: 'Delete',
      onConfirm: async () => {
        await deleteProduct(params.productId);
        // Navigate back after deletion
        pop();
      },
    });
  };
  
  return (
    <div>
      {/* Product details */}
      <button onClick={handleDelete}>Delete Product</button>
    </div>
  );
};
```

### Bottom Sheet

```typescript
// activities/FilterBottomSheetActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useState } from 'react';

type FilterParams = {
  currentFilters: {
    category?: string;
    priceRange?: [number, number];
    rating?: number;
  };
  onApply: (filters: any) => void;
};

const FilterBottomSheetActivity: ActivityComponentType<FilterParams> = ({ params }) => {
  const { pop } = useFlow();
  const [filters, setFilters] = useState(params.currentFilters);
  
  const handleApply = () => {
    params.onApply(filters);
    pop();
  };
  
  return (
    <div className="bottom-sheet-overlay" onClick={() => pop()}>
      <div className="bottom-sheet" onClick={(e) => e.stopPropagation()}>
        <div className="handle" />
        
        <h2>Filters</h2>
        
        <div className="filter-section">
          <h3>Category</h3>
          <select 
            value={filters.category} 
            onChange={(e) => setFilters({ ...filters, category: e.target.value })}
          >
            <option value="">All</option>
            <option value="electronics">Electronics</option>
            <option value="clothing">Clothing</option>
            <option value="books">Books</option>
          </select>
        </div>
        
        <div className="filter-section">
          <h3>Price Range</h3>
          <input
            type="range"
            min="0"
            max="1000"
            value={filters.priceRange?.[1] || 1000}
            onChange={(e) => setFilters({ 
              ...filters, 
              priceRange: [0, parseInt(e.target.value)] 
            })}
          />
          <span>${filters.priceRange?.[0] || 0} - ${filters.priceRange?.[1] || 1000}</span>
        </div>
        
        <div className="filter-section">
          <h3>Minimum Rating</h3>
          <div className="rating-buttons">
            {[1, 2, 3, 4, 5].map(rating => (
              <button
                key={rating}
                className={filters.rating === rating ? 'active' : ''}
                onClick={() => setFilters({ ...filters, rating })}
              >
                {'⭐'.repeat(rating)}
              </button>
            ))}
          </div>
        </div>
        
        <div className="actions">
          <button onClick={() => setFilters({})}>Clear All</button>
          <button onClick={handleApply} className="primary">Apply Filters</button>
        </div>
      </div>
    </div>
  );
};

export default FilterBottomSheetActivity;
```

## Tab Navigation with Stackflow

### Multiple Stacks in Tabs

```typescript
// App.tsx
import { useState } from 'react';
import { Stack as HomeStack } from './stacks/homeStack';
import { Stack as SearchStack } from './stacks/searchStack';
import { Stack as ProfileStack } from './stacks/profileStack';

function App() {
  const [activeTab, setActiveTab] = useState<'home' | 'search' | 'profile'>('home');
  
  return (
    <div className="app">
      <div className="content">
        <div style={{ display: activeTab === 'home' ? 'block' : 'none' }}>
          <HomeStack />
        </div>
        <div style={{ display: activeTab === 'search' ? 'block' : 'none' }}>
          <SearchStack />
        </div>
        <div style={{ display: activeTab === 'profile' ? 'block' : 'none' }}>
          <ProfileStack />
        </div>
      </div>
      
      <nav className="tab-bar">
        <button 
          className={activeTab === 'home' ? 'active' : ''}
          onClick={() => setActiveTab('home')}
        >
          🏠 Home
        </button>
        <button 
          className={activeTab === 'search' ? 'active' : ''}
          onClick={() => setActiveTab('search')}
        >
          🔍 Search
        </button>
        <button 
          className={activeTab === 'profile' ? 'active' : ''}
          onClick={() => setActiveTab('profile')}
        >
          👤 Profile
        </button>
      </nav>
    </div>
  );
}

export default App;

// stacks/homeStack.ts
import { stackflow } from '@stackflow/react';
import { basicUIPlugin } from '@stackflow/plugin-basic-ui';

export const { Stack, useFlow } = stackflow({
  transitionDuration: 350,
  activities: {
    HomeFeed: () => import('../activities/home/HomeFeedActivity'),
    PostDetail: () => import('../activities/home/PostDetailActivity'),
  },
  plugins: [basicUIPlugin({ theme: 'cupertino' })],
});
```

## Form Wizard

### Multi-step Form

```typescript
// stackflow.ts
type TypedActivities = {
  FormStep1: {};
  FormStep2: { step1Data: { name: string; email: string } };
  FormStep3: { 
    step1Data: { name: string; email: string };
    step2Data: { address: string; city: string };
  };
  FormComplete: { formData: any };
};

// activities/FormStep1Activity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useState } from 'react';

const FormStep1Activity: ActivityComponentType = () => {
  const { push } = useFlow();
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  const handleNext = () => {
    if (!name || !email) {
      alert('Please fill all fields');
      return;
    }
    
    push('FormStep2', {
      step1Data: { name, email }
    });
  };
  
  return (
    <div className="form-step">
      <div className="progress-bar">
        <div className="progress" style={{ width: '33%' }} />
      </div>
      
      <h1>Step 1: Personal Information</h1>
      
      <form onSubmit={(e) => { e.preventDefault(); handleNext(); }}>
        <label>
          Name:
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
          />
        </label>
        
        <label>
          Email:
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
        </label>
        
        <button type="submit">Next →</button>
      </form>
    </div>
  );
};

export default FormStep1Activity;

// activities/FormStep2Activity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useState } from 'react';

type FormStep2Params = {
  step1Data: { name: string; email: string };
};

const FormStep2Activity: ActivityComponentType<FormStep2Params> = ({ params }) => {
  const { push, pop } = useFlow();
  const [address, setAddress] = useState('');
  const [city, setCity] = useState('');
  
  const handleNext = () => {
    if (!address || !city) {
      alert('Please fill all fields');
      return;
    }
    
    push('FormStep3', {
      step1Data: params.step1Data,
      step2Data: { address, city }
    });
  };
  
  return (
    <div className="form-step">
      <div className="progress-bar">
        <div className="progress" style={{ width: '66%' }} />
      </div>
      
      <h1>Step 2: Address</h1>
      
      <form onSubmit={(e) => { e.preventDefault(); handleNext(); }}>
        <label>
          Address:
          <input
            type="text"
            value={address}
            onChange={(e) => setAddress(e.target.value)}
            required
          />
        </label>
        
        <label>
          City:
          <input
            type="text"
            value={city}
            onChange={(e) => setCity(e.target.value)}
            required
          />
        </label>
        
        <div className="form-actions">
          <button type="button" onClick={() => pop()}>← Back</button>
          <button type="submit">Next →</button>
        </div>
      </form>
    </div>
  );
};

export default FormStep2Activity;
```

## Authentication Flow

### Login/Signup Flow

```typescript
// stackflow.ts
type TypedActivities = {
  Welcome: {};
  Login: {};
  Signup: {};
  ForgotPassword: {};
  ResetPassword: { token: string };
  Main: {};
};

// activities/WelcomeActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';

const WelcomeActivity: ActivityComponentType = () => {
  const { push, replace } = useFlow();
  const { user } = useAuth();
  
  useEffect(() => {
    // If already logged in, go to main app
    if (user) {
      replace('Main');
    }
  }, [user, replace]);
  
  return (
    <div className="welcome">
      <h1>Welcome to Our App</h1>
      <p>Get started by creating an account or logging in</p>
      
      <button onClick={() => push('Signup')} className="primary">
        Create Account
      </button>
      
      <button onClick={() => push('Login')} className="secondary">
        Log In
      </button>
    </div>
  );
};

export default WelcomeActivity;

// activities/LoginActivity.tsx
import { ActivityComponentType } from '@stackflow/react';
import { useFlow } from '../stackflow';
import { useState } from 'react';

const LoginActivity: ActivityComponentType = () => {
  const { push, replace, pop } = useFlow();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const { login } = useAuth();
  
  const handleLogin = async () => {
    try {
      await login(email, password);
      replace('Main'); // Replace entire stack to prevent going back
    } catch (err) {
      setError('Invalid credentials');
    }
  };
  
  return (
    <div className="auth-form">
      <button onClick={() => pop()} className="back-button">← Back</button>
      
      <h1>Log In</h1>
      
      {error && <div className="error">{error}</div>}
      
      <form onSubmit={(e) => { e.preventDefault(); handleLogin(); }}>
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
        
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
        
        <button type="submit">Log In</button>
      </form>
      
      <button 
        onClick={() => push('ForgotPassword')}
        className="link"
      >
        Forgot Password?
      </button>
      
      <p>
        Don't have an account?{' '}
        <button onClick={() => replace('Signup')} className="link">
          Sign Up
        </button>
      </p>
    </div>
  );
};

export default LoginActivity;
```

## Deep Linking

### Handle URL Parameters

```typescript
// App.tsx
import { useEffect } from 'react';
import { Stack, useFlow } from './stackflow';

function App() {
  const { push } = useFlow();
  
  useEffect(() => {
    // Handle initial deep link
    const path = window.location.pathname;
    const params = new URLSearchParams(window.location.search);
    
    if (path.startsWith('/product/')) {
      const productId = path.split('/')[2];
      push('ProductDetail', { productId });
    } else if (path === '/cart') {
      push('Cart', {});
    }
  }, [push]);
  
  return <Stack />;
}

export default App;

// Also update URL when navigating
const ProductListActivity = () => {
  const { push } = useFlow();
  
  const handleProductClick = (productId: string) => {
    // Update URL
    window.history.pushState({}, '', `/product/${productId}`);
    
    // Navigate in stack
    push('ProductDetail', { productId });
  };
  
  return (
    <div>
      {products.map(product => (
        <div key={product.id} onClick={() => handleProductClick(product.id)}>
          {product.name}
        </div>
      ))}
    </div>
  );
};
```

---

These examples cover the most common use cases for Stackflow. Refer to the official documentation at https://stackflow.so for more advanced patterns and features.
