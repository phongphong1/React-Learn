# 07 - Context API và Zustand

## 📚 Mục Lục

1. [State Management là gì?](#state-management-là-gì)
2. [Props Drilling Problem](#props-drilling-problem)
3. [Context API](#context-api)
4. [useContext Hook](#usecontext-hook)
5. [Context API Best Practices](#context-api-best-practices)
6. [Zustand - Simple State Management](#zustand-simple-state-management)
7. [Context API vs Zustand](#context-api-vs-zustand)
8. [Ví dụ thực tế](#ví-dụ-thực-tế)
9. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🎯 State Management là gì?

**State Management** là cách quản lý và chia sẻ dữ liệu (state) giữa nhiều component trong ứng dụng.

### Vấn đề

Khi ứng dụng lớn lên:
- State cần được chia sẻ giữa nhiều component
- Props drilling (truyền props qua nhiều tầng)
- Khó maintain và debug

> 💡 **So sánh với Java**: Giống như bạn cần **Singleton Pattern** hoặc **Application-scoped Bean** trong Spring để chia sẻ data giữa các class.

### Các giải pháp

| **Giải pháp** | **Độ phức tạp** | **Use case** |
|--------------|----------------|-------------|
| **Props** | Thấp | State local, 1-2 tầng |
| **Context API** | Trung bình | State global đơn giản |
| **Zustand** | Trung bình | State global, hiệu năng cao |
| **Redux** | Cao | Ứng dụng lớn, phức tạp |

---

## 🔗 Props Drilling Problem

### Vấn đề: Truyền props qua nhiều tầng

```jsx
// ❌ Props Drilling - Phải truyền qua nhiều component
function App() {
  const [user, setUser] = useState({ name: "John", role: "admin" });

  return <Dashboard user={user} />;
}

function Dashboard({ user }) {
  return (
    <div>
      <Sidebar user={user} />
      <MainContent user={user} />
    </div>
  );
}

function Sidebar({ user }) {
  return (
    <div>
      <UserMenu user={user} />
    </div>
  );
}

function UserMenu({ user }) {
  // Component này mới dùng user, nhưng phải truyền qua 3 tầng!
  return <div>Welcome, {user.name}</div>;
}
```

### Vấn đề

- ❌ Code dài dòng
- ❌ Khó maintain khi thêm/bớt props
- ❌ Component trung gian không cần props nhưng vẫn phải nhận
- ❌ Khó refactor

---

## 🌐 Context API

**Context API** cho phép chia sẻ data giữa các component **mà không cần truyền props**.

### Cách hoạt động

```
┌─────────────┐
│   Provider  │ ← Cung cấp data
└──────┬──────┘
       │
   ┌───┴────┐
   │        │
┌──▼──┐  ┌─▼───┐
│Child1│  │Child2│ ← Consume data (dù ở tầng sâu)
└─────┘  └──────┘
```

### 3 bước sử dụng Context

```jsx
// 1️⃣ Create Context
import { createContext } from 'react';

const UserContext = createContext();

// 2️⃣ Provide Context
function App() {
  const [user, setUser] = useState({ name: "John", role: "admin" });

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Dashboard />
    </UserContext.Provider>
  );
}

// 3️⃣ Consume Context
import { useContext } from 'react';

function UserMenu() {
  const { user } = useContext(UserContext);

  return <div>Welcome, {user.name}</div>;
}
```

### So sánh với Spring

```java
// Java - Application-scoped Bean (giống Context)
@Component
@Scope("application")
public class UserContext {
    private User currentUser;

    public User getCurrentUser() {
        return currentUser;
    }
}

// Inject vào bất kỳ đâu
@Service
public class UserService {
    @Autowired
    private UserContext userContext;

    public void doSomething() {
        User user = userContext.getCurrentUser();
    }
}
```

---

## 🪝 useContext Hook

### Cú pháp

```jsx
const value = useContext(MyContext);
```

### Ví dụ đầy đủ

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create Context
const ThemeContext = createContext();

// 2. Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Custom Hook (Optional, nhưng khuyến nghị)
function useTheme() {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }

  return context;
}

// 4. App Component
function App() {
  return (
    <ThemeProvider>
      <Header />
      <MainContent />
      <Footer />
    </ThemeProvider>
  );
}

// 5. Consumer Component (bất kỳ đâu trong tree)
function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header className={theme}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

function MainContent() {
  const { theme } = useTheme();

  return (
    <main className={theme}>
      <p>Content in {theme} mode</p>
    </main>
  );
}
```

---

## ✅ Context API Best Practices

### 1. Tách Context theo chức năng

```jsx
// ❌ SAI: Một Context chứa tất cả
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');
  const [notifications, setNotifications] = useState([]);

  // Khi bất kỳ state nào thay đổi → TẤT CẢ consumer re-render!

  return (
    <AppContext.Provider value={{ user, theme, language, notifications, ... }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ ĐÚNG: Tách thành nhiều Context
const UserContext = createContext();
const ThemeContext = createContext();
const LanguageContext = createContext();

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <LanguageProvider>
          <AppContent />
        </LanguageProvider>
      </ThemeProvider>
    </UserProvider>
  );
}
```

### 2. Custom Hook cho mỗi Context

```jsx
// UserContext.jsx
import { createContext, useContext, useState } from 'react';

const UserContext = createContext();

export function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

// Custom Hook - Export này thay vì export Context
export function useUser() {
  const context = useContext(UserContext);

  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }

  return context;
}

// Sử dụng
import { useUser } from './UserContext';

function Profile() {
  const { user, logout } = useUser();

  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

### 3. Optimization với useMemo

```jsx
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  // ❌ SAI: Mỗi lần render tạo object mới → Tất cả consumer re-render
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );

  // ✅ ĐÚNG: Dùng useMemo để cache value
  const value = useMemo(() => ({ user, setUser }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

---

## ⚡ Zustand - Simple State Management

**Zustand** là thư viện state management đơn giản, nhẹ, không dùng Context, hiệu năng cao.

### Cài đặt

```bash
npm install zustand
```

### 1. Basic Usage

```jsx
import { create } from 'zustand';

// 1. Create Store (giống Redux store)
const useUserStore = create((set) => ({
  // State
  user: null,

  // Actions
  login: (userData) => set({ user: userData }),
  logout: () => set({ user: null }),
  updateUser: (updates) => set((state) => ({
    user: { ...state.user, ...updates }
  }))
}));

// 2. Sử dụng trong Component
function Profile() {
  // Chỉ subscribe vào "user" → Chỉ re-render khi user thay đổi
  const user = useUserStore((state) => state.user);
  const logout = useUserStore((state) => state.logout);

  if (!user) return <div>Please login</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

function LoginButton() {
  const login = useUserStore((state) => state.login);

  const handleLogin = () => {
    login({ id: 1, name: "John Doe", email: "john@example.com" });
  };

  return <button onClick={handleLogin}>Login</button>;
}
```

### 2. Complex Store

```jsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

// Store với middleware
const useStore = create(
  devtools(
    persist(
      (set, get) => ({
        // State
        user: null,
        cart: [],
        theme: 'light',

        // Actions
        login: (userData) => set({ user: userData }),
        logout: () => set({ user: null, cart: [] }),

        // Cart actions
        addToCart: (product) => set((state) => ({
          cart: [...state.cart, product]
        })),

        removeFromCart: (productId) => set((state) => ({
          cart: state.cart.filter(item => item.id !== productId)
        })),

        clearCart: () => set({ cart: [] }),

        // Computed value (giống getter)
        getTotalPrice: () => {
          const cart = get().cart;
          return cart.reduce((total, item) => total + item.price, 0);
        },

        // Theme
        toggleTheme: () => set((state) => ({
          theme: state.theme === 'light' ? 'dark' : 'light'
        }))
      }),
      {
        name: 'app-storage', // LocalStorage key
        partialPersist: ['user', 'theme'] // Chỉ persist user và theme
      }
    )
  )
);

// Sử dụng
function ShoppingCart() {
  const cart = useStore((state) => state.cart);
  const removeFromCart = useStore((state) => state.removeFromCart);
  const getTotalPrice = useStore((state) => state.getTotalPrice);

  return (
    <div>
      <h2>Cart ({cart.length} items)</h2>

      {cart.map(item => (
        <div key={item.id}>
          {item.name} - ${item.price}
          <button onClick={() => removeFromCart(item.id)}>Remove</button>
        </div>
      ))}

      <h3>Total: ${getTotalPrice()}</h3>
    </div>
  );
}
```

### 3. Slice Pattern (Tách Store)

```jsx
// stores/userSlice.js
export const createUserSlice = (set) => ({
  user: null,
  login: (userData) => set({ user: userData }),
  logout: () => set({ user: null })
});

// stores/cartSlice.js
export const createCartSlice = (set) => ({
  cart: [],
  addToCart: (product) => set((state) => ({
    cart: [...state.cart, product]
  })),
  clearCart: () => set({ cart: [] })
});

// stores/index.js
import { create } from 'zustand';
import { createUserSlice } from './userSlice';
import { createCartSlice } from './cartSlice';

export const useStore = create((...a) => ({
  ...createUserSlice(...a),
  ...createCartSlice(...a)
}));
```

---

## ⚖️ Context API vs Zustand

| **Tiêu chí** | **Context API** | **Zustand** |
|-------------|----------------|-------------|
| **Setup** | Nhiều boilerplate | Ít code hơn |
| **Performance** | Re-render nhiều hơn | Tối ưu hơn |
| **Learning Curve** | Dễ (built-in React) | Dễ (đơn giản hơn Redux) |
| **DevTools** | Không có | Có support |
| **Middleware** | Tự implement | Built-in |
| **TypeScript** | OK | Excellent |
| **Bundle Size** | 0 (built-in) | ~1KB |
| **Use Case** | State đơn giản | State phức tạp |

### Khi nào dùng cái nào?

#### Dùng Context API khi:
- ✅ State đơn giản (theme, language, auth)
- ✅ Ít thay đổi
- ✅ Không cần DevTools
- ✅ Muốn dùng built-in React

#### Dùng Zustand khi:
- ✅ State phức tạp (shopping cart, user data)
- ✅ Cần hiệu năng cao
- ✅ Cần DevTools
- ✅ Cần persist state (LocalStorage)
- ✅ Nhiều actions

---

## 🔥 Ví dụ thực tế

### 1. Authentication với Context API

```jsx
// AuthContext.jsx
import { createContext, useContext, useState, useEffect } from 'react';
import axios from 'axios';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Check if user is logged in (on mount)
  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      axios.get('/api/auth/me', {
        headers: { Authorization: `Bearer ${token}` }
      })
        .then(response => setUser(response.data))
        .catch(() => localStorage.removeItem('token'))
        .finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);

  const login = async (email, password) => {
    const response = await axios.post('/api/auth/login', { email, password });
    const { token, user } = response.data;

    localStorage.setItem('token', token);
    setUser(user);

    return user;
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  const value = { user, login, logout, loading };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();

  if (loading) return <div>Loading...</div>;
  if (!user) return <Navigate to="/login" />;

  return children;
}

// Sử dụng
function App() {
  return (
    <AuthProvider>
      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/dashboard" element={
          <ProtectedRoute>
            <Dashboard />
          </ProtectedRoute>
        } />
      </Routes>
    </AuthProvider>
  );
}
```

### 2. Shopping Cart với Zustand

```jsx
// stores/cartStore.js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useCartStore = create(
  persist(
    (set, get) => ({
      cart: [],

      addToCart: (product) => {
        const cart = get().cart;
        const existingItem = cart.find(item => item.id === product.id);

        if (existingItem) {
          set({
            cart: cart.map(item =>
              item.id === product.id
                ? { ...item, quantity: item.quantity + 1 }
                : item
            )
          });
        } else {
          set({ cart: [...cart, { ...product, quantity: 1 }] });
        }
      },

      removeFromCart: (productId) => {
        set({
          cart: get().cart.filter(item => item.id !== productId)
        });
      },

      updateQuantity: (productId, quantity) => {
        if (quantity <= 0) {
          get().removeFromCart(productId);
          return;
        }

        set({
          cart: get().cart.map(item =>
            item.id === productId ? { ...item, quantity } : item
          )
        });
      },

      clearCart: () => set({ cart: [] }),

      getTotal: () => {
        return get().cart.reduce(
          (total, item) => total + (item.price * item.quantity),
          0
        );
      },

      getItemCount: () => {
        return get().cart.reduce((count, item) => count + item.quantity, 0);
      }
    }),
    {
      name: 'shopping-cart'
    }
  )
);

// Components
function ProductCard({ product }) {
  const addToCart = useCartStore((state) => state.addToCart);

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => addToCart(product)}>
        Add to Cart
      </button>
    </div>
  );
}

function CartIcon() {
  const itemCount = useCartStore((state) => state.getItemCount());

  return (
    <button className="cart-icon">
      🛒 <span className="badge">{itemCount}</span>
    </button>
  );
}

function CartPage() {
  const cart = useCartStore((state) => state.cart);
  const updateQuantity = useCartStore((state) => state.updateQuantity);
  const removeFromCart = useCartStore((state) => state.removeFromCart);
  const getTotal = useCartStore((state) => state.getTotal);
  const clearCart = useCartStore((state) => state.clearCart);

  return (
    <div>
      <h2>Shopping Cart</h2>

      {cart.map(item => (
        <div key={item.id} className="cart-item">
          <h4>{item.name}</h4>
          <p>${item.price} x {item.quantity} = ${item.price * item.quantity}</p>

          <input
            type="number"
            value={item.quantity}
            onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
            min="1"
          />

          <button onClick={() => removeFromCart(item.id)}>Remove</button>
        </div>
      ))}

      <h3>Total: ${getTotal()}</h3>

      <button onClick={clearCart}>Clear Cart</button>
      <button>Checkout</button>
    </div>
  );
}
```

---

## 📝 Bài tập thực hành

### Bài 1: Theme Switcher

Tạo ThemeContext:
- State: theme ('light' | 'dark')
- Action: toggleTheme()
- Apply theme cho toàn bộ app (background, text color)
- Persist theme trong localStorage

### Bài 2: Todo App với Zustand

Tạo Zustand store cho Todo app:
- State: todos array
- Actions: addTodo, removeTodo, toggleTodo, clearCompleted
- Computed: getCompletedCount, getPendingCount
- Persist todos trong localStorage

### Bài 3: Multi-language Support

Tạo LanguageContext:
- State: language ('en' | 'vi')
- translations object
- Action: changeLanguage()
- Custom hook: useTranslation() để translate text

### Bài 4: Notification System

Tạo NotificationStore (Zustand):
- State: notifications array
- Actions: addNotification, removeNotification, clearAll
- Auto-remove notification sau 5s
- Hiển thị notifications ở góc màn hình

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ Props drilling problem và cách giải quyết
✅ Context API để chia sẻ state globally
✅ useContext Hook
✅ Context API best practices
✅ Zustand - state management đơn giản, hiệu năng cao
✅ Khi nào dùng Context API vs Zustand

**Bài tiếp theo:** [08 - React Router DOM](08-react-router-dom.md)

---

> 💬 **Tips**: Dùng **Context API** cho state ít thay đổi (theme, auth), dùng **Zustand** cho state phức tạp (cart, user data). Tránh đặt tất cả state vào một Context vì sẽ gây re-render không cần thiết!
