# 08 - React Router DOM

## 📚 Mục Lục

1. [SPA và Routing](#spa-và-routing)
2. [React Router DOM](#react-router-dom)
3. [Setup và Basic Routes](#setup-và-basic-routes)
4. [Dynamic Routes và URL Parameters](#dynamic-routes-và-url-parameters)
5. [Nested Routes](#nested-routes)
6. [Navigation và Link](#navigation-và-link)
7. [Protected Routes](#protected-routes)
8. [Query Parameters và Search](#query-parameters-và-search)
9. [Ví dụ thực tế](#ví-dụ-thực-tế)
10. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🌐 SPA và Routing

### Single Page Application (SPA)

**SPA** là ứng dụng web chỉ load **1 HTML page duy nhất**, sau đó dynamically update content khi user navigate.

| **Traditional Web** | **SPA (React)** |
|--------------------|-----------------|
| Mỗi page = 1 request mới | 1 request ban đầu, sau đó client-side routing |
| Page reload mỗi lần navigate | Không reload page |
| Server-side routing | Client-side routing |
| Slower | Faster, mượt hơn |

> 💡 **So sánh với Spring Boot**: Spring MVC có **@RequestMapping** cho server-side routing, React Router là **client-side routing** - không cần gửi request lên server mỗi lần đổi trang.

### Client-side Routing

```
User click link
     ↓
React Router intercept
     ↓
Update URL (không reload page)
     ↓
Render component tương ứng
     ↓
Update UI
```

---

## 🛣️ React Router DOM

**React Router DOM** là thư viện routing phổ biến nhất cho React web applications.

### Cài đặt

```bash
npm install react-router-dom
```

### Core Components

| **Component** | **Chức năng** |
|--------------|---------------|
| `<BrowserRouter>` | Wrapper cho toàn bộ app |
| `<Routes>` | Container cho các route |
| `<Route>` | Định nghĩa một route |
| `<Link>` | Navigate giữa các trang (thay `<a>`) |
| `<Navigate>` | Redirect programmatically |
| `<Outlet>` | Render nested routes |

---

## 🚀 Setup và Basic Routes

### 1. Setup Router

```jsx
// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

### 2. Define Routes

```jsx
// App.jsx
import { Routes, Route } from 'react-router-dom';
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ContactPage from './pages/ContactPage';
import NotFoundPage from './pages/NotFoundPage';

function App() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
      </nav>

      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/contact" element={<ContactPage />} />
        <Route path="*" element={<NotFoundPage />} />  {/* 404 */}
      </Routes>
    </div>
  );
}
```

### 3. Page Components

```jsx
// pages/HomePage.jsx
function HomePage() {
  return (
    <div>
      <h1>Home Page</h1>
      <p>Welcome to our website!</p>
    </div>
  );
}

export default HomePage;
```

### So sánh với Spring Boot

```java
// Spring Boot Controller
@Controller
public class PageController {

    @GetMapping("/")
    public String home() {
        return "home";  // home.html
    }

    @GetMapping("/about")
    public String about() {
        return "about";  // about.html
    }
}
```

---

## 🎯 Dynamic Routes và URL Parameters

### 1. URL Parameters

```jsx
// App.jsx
<Routes>
  <Route path="/users/:userId" element={<UserProfile />} />
  <Route path="/products/:productId" element={<ProductDetail />} />
</Routes>

// UserProfile.jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams();  // Lấy parameter từ URL

  return <h1>User Profile: {userId}</h1>;
}

// URL: /users/123 → userId = "123"
// URL: /users/abc → userId = "abc"
```

### 2. Multiple Parameters

```jsx
<Route path="/posts/:postId/comments/:commentId" element={<CommentDetail />} />

function CommentDetail() {
  const { postId, commentId } = useParams();

  return (
    <div>
      <h1>Post: {postId}</h1>
      <h2>Comment: {commentId}</h2>
    </div>
  );
}

// URL: /posts/5/comments/10 → postId=5, commentId=10
```

### 3. Fetch Data với URL Params

```jsx
import { useParams } from 'react-router-dom';
import { useState, useEffect } from 'react';
import axios from 'axios';

function ProductDetail() {
  const { productId } = useParams();
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    axios.get(`/api/products/${productId}`)
      .then(response => setProduct(response.data))
      .catch(error => console.error(error))
      .finally(() => setLoading(false));
  }, [productId]);

  if (loading) return <div>Loading...</div>;
  if (!product) return <div>Product not found</div>;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>Price: ${product.price}</p>
    </div>
  );
}
```

---

## 🏗️ Nested Routes

Nested routes cho phép tạo layout với sub-routes.

### 1. Setup Nested Routes

```jsx
// App.jsx
import { Routes, Route } from 'react-router-dom';
import DashboardLayout from './layouts/DashboardLayout';
import DashboardHome from './pages/DashboardHome';
import DashboardProfile from './pages/DashboardProfile';
import DashboardSettings from './pages/DashboardSettings';

function App() {
  return (
    <Routes>
      <Route path="/" element={<HomePage />} />

      {/* Nested Routes */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} />  {/* /dashboard */}
        <Route path="profile" element={<DashboardProfile />} />  {/* /dashboard/profile */}
        <Route path="settings" element={<DashboardSettings />} />  {/* /dashboard/settings */}
      </Route>
    </Routes>
  );
}
```

### 2. Layout với Outlet

```jsx
// layouts/DashboardLayout.jsx
import { Outlet, Link } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div className="dashboard">
      <aside className="sidebar">
        <nav>
          <Link to="/dashboard">Home</Link>
          <Link to="/dashboard/profile">Profile</Link>
          <Link to="/dashboard/settings">Settings</Link>
        </nav>
      </aside>

      <main className="content">
        {/* Nested route sẽ render ở đây */}
        <Outlet />
      </main>
    </div>
  );
}
```

### Cấu trúc URL

```
/dashboard              → DashboardLayout + DashboardHome
/dashboard/profile      → DashboardLayout + DashboardProfile
/dashboard/settings     → DashboardLayout + DashboardSettings
```

---

## 🔗 Navigation và Link

### 1. Link Component

```jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      {/* ✅ ĐÚNG: Dùng Link (không reload page) */}
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>

      {/* ❌ SAI: Dùng <a> (reload page) */}
      <a href="/">Home</a>
    </nav>
  );
}
```

### 2. NavLink (Active Styling)

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink
        to="/"
        className={({ isActive }) => isActive ? "active" : ""}
      >
        Home
      </NavLink>

      <NavLink
        to="/about"
        style={({ isActive }) => ({
          color: isActive ? 'red' : 'black',
          fontWeight: isActive ? 'bold' : 'normal'
        })}
      >
        About
      </NavLink>
    </nav>
  );
}
```

### 3. Programmatic Navigation

```jsx
import { useNavigate } from 'react-router-dom';

function LoginPage() {
  const navigate = useNavigate();

  const handleLogin = async (credentials) => {
    try {
      await loginAPI(credentials);

      // Navigate to dashboard after login
      navigate('/dashboard');

      // Navigate back
      // navigate(-1);

      // Navigate with replace (không thêm vào history)
      // navigate('/dashboard', { replace: true });
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleLogin}>
      {/* Form fields */}
    </form>
  );
}
```

### 4. Navigate Component (Redirect)

```jsx
import { Navigate } from 'react-router-dom';

function OldProductPage() {
  // Redirect cũ URL sang mới
  return <Navigate to="/products" replace />;
}

// Conditional Redirect
function AdminPage() {
  const { user } = useAuth();

  if (!user?.isAdmin) {
    return <Navigate to="/" replace />;
  }

  return <div>Admin Panel</div>;
}
```

---

## 🔒 Protected Routes

Bảo vệ routes chỉ cho phép user đã đăng nhập.

### 1. ProtectedRoute Component

```jsx
// components/ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();

  if (loading) {
    return <div>Loading...</div>;
  }

  if (!user) {
    // Redirect to login if not authenticated
    return <Navigate to="/login" replace />;
  }

  return children;
}

export default ProtectedRoute;
```

### 2. Sử dụng ProtectedRoute

```jsx
// App.jsx
import ProtectedRoute from './components/ProtectedRoute';

function App() {
  return (
    <Routes>
      <Route path="/login" element={<LoginPage />} />

      {/* Protected Routes */}
      <Route path="/dashboard" element={
        <ProtectedRoute>
          <DashboardPage />
        </ProtectedRoute>
      } />

      <Route path="/profile" element={
        <ProtectedRoute>
          <ProfilePage />
        </ProtectedRoute>
      } />
    </Routes>
  );
}
```

### 3. Role-based Protection

```jsx
function RequireRole({ children, allowedRoles }) {
  const { user } = useAuth();

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  if (!allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

// Sử dụng
<Route path="/admin" element={
  <RequireRole allowedRoles={['admin']}>
    <AdminPage />
  </RequireRole>
} />
```

---

## 🔍 Query Parameters và Search

### 1. useSearchParams Hook

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();

  // Lấy query params
  const category = searchParams.get('category');
  const sort = searchParams.get('sort');
  const page = searchParams.get('page') || 1;

  // URL: /products?category=electronics&sort=price&page=2
  // category = "electronics"
  // sort = "price"
  // page = "2"

  const handleFilterChange = (newCategory) => {
    setSearchParams({ category: newCategory, page: 1 });
    // URL becomes: /products?category=newCategory&page=1
  };

  return (
    <div>
      <h2>Products - Category: {category}</h2>

      <select onChange={(e) => handleFilterChange(e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>

      {/* Render products based on filters */}
    </div>
  );
}
```

### 2. Search với Filter

```jsx
function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const [query, setQuery] = useState(searchParams.get('q') || '');

  const handleSearch = (e) => {
    e.preventDefault();
    setSearchParams({ q: query });
  };

  useEffect(() => {
    const searchQuery = searchParams.get('q');
    if (searchQuery) {
      // Fetch search results
      fetchResults(searchQuery);
    }
  }, [searchParams]);

  return (
    <div>
      <form onSubmit={handleSearch}>
        <input
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search..."
        />
        <button type="submit">Search</button>
      </form>
    </div>
  );
}
```

---

## 🔥 Ví dụ thực tế

### Complete E-commerce Routing

```jsx
// App.jsx
import { Routes, Route } from 'react-router-dom';
import Layout from './layouts/Layout';
import HomePage from './pages/HomePage';
import ProductsPage from './pages/ProductsPage';
import ProductDetail from './pages/ProductDetail';
import CartPage from './pages/CartPage';
import CheckoutPage from './pages/CheckoutPage';
import LoginPage from './pages/LoginPage';
import RegisterPage from './pages/RegisterPage';
import ProfilePage from './pages/ProfilePage';
import OrdersPage from './pages/OrdersPage';
import NotFoundPage from './pages/NotFoundPage';
import ProtectedRoute from './components/ProtectedRoute';

function App() {
  return (
    <Routes>
      {/* Public Routes */}
      <Route path="/" element={<Layout />}>
        <Route index element={<HomePage />} />
        <Route path="products" element={<ProductsPage />} />
        <Route path="products/:productId" element={<ProductDetail />} />
        <Route path="cart" element={<CartPage />} />

        {/* Auth Routes */}
        <Route path="login" element={<LoginPage />} />
        <Route path="register" element={<RegisterPage />} />

        {/* Protected Routes */}
        <Route path="checkout" element={
          <ProtectedRoute>
            <CheckoutPage />
          </ProtectedRoute>
        } />

        <Route path="profile" element={
          <ProtectedRoute>
            <ProfilePage />
          </ProtectedRoute>
        } />

        <Route path="orders" element={
          <ProtectedRoute>
            <OrdersPage />
          </ProtectedRoute>
        } />

        {/* 404 */}
        <Route path="*" element={<NotFoundPage />} />
      </Route>
    </Routes>
  );
}

// layouts/Layout.jsx
import { Outlet } from 'react-router-dom';
import Header from '../components/Header';
import Footer from '../components/Footer';

function Layout() {
  return (
    <div className="app">
      <Header />
      <main>
        <Outlet />  {/* Nested routes render here */}
      </main>
      <Footer />
    </div>
  );
}

// components/Header.jsx
import { Link, NavLink } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';
import { useCartStore } from '../stores/cartStore';

function Header() {
  const { user, logout } = useAuth();
  const cartCount = useCartStore((state) => state.getItemCount());

  return (
    <header>
      <nav>
        <Link to="/" className="logo">MyShop</Link>

        <div className="nav-links">
          <NavLink to="/">Home</NavLink>
          <NavLink to="/products">Products</NavLink>

          <Link to="/cart">
            🛒 Cart ({cartCount})
          </Link>

          {user ? (
            <>
              <NavLink to="/profile">Profile</NavLink>
              <NavLink to="/orders">Orders</NavLink>
              <button onClick={logout}>Logout</button>
            </>
          ) : (
            <>
              <NavLink to="/login">Login</NavLink>
              <NavLink to="/register">Register</NavLink>
            </>
          )}
        </div>
      </nav>
    </header>
  );
}

// pages/ProductsPage.jsx
import { useState, useEffect } from 'react';
import { useSearchParams, Link } from 'react-router-dom';
import axios from 'axios';

function ProductsPage() {
  const [products, setProducts] = useState([]);
  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get('category') || 'all';
  const sort = searchParams.get('sort') || 'name';

  useEffect(() => {
    axios.get('/api/products', {
      params: { category, sort }
    })
      .then(response => setProducts(response.data))
      .catch(error => console.error(error));
  }, [category, sort]);

  return (
    <div>
      <h1>Products</h1>

      <div className="filters">
        <select
          value={category}
          onChange={(e) => setSearchParams({ category: e.target.value, sort })}
        >
          <option value="all">All Categories</option>
          <option value="electronics">Electronics</option>
          <option value="clothing">Clothing</option>
        </select>

        <select
          value={sort}
          onChange={(e) => setSearchParams({ category, sort: e.target.value })}
        >
          <option value="name">Name</option>
          <option value="price">Price</option>
          <option value="rating">Rating</option>
        </select>
      </div>

      <div className="product-grid">
        {products.map(product => (
          <Link key={product.id} to={`/products/${product.id}`}>
            <div className="product-card">
              <img src={product.image} alt={product.name} />
              <h3>{product.name}</h3>
              <p>${product.price}</p>
            </div>
          </Link>
        ))}
      </div>
    </div>
  );
}
```

---

## 📝 Bài tập thực hành

### Bài 1: Blog Application

Tạo routing cho blog app:
- `/` - Home page (list all posts)
- `/posts/:postId` - Post detail
- `/posts/:postId/edit` - Edit post (protected)
- `/categories/:category` - Posts by category
- `/search?q=query` - Search posts
- `/login` - Login page
- `/404` - Not found page

### Bài 2: Admin Dashboard

Tạo nested routes cho admin dashboard:
- `/admin` - Dashboard home
- `/admin/users` - User list
- `/admin/users/:userId` - User detail
- `/admin/products` - Product management
- `/admin/orders` - Orders
- Tất cả routes phải protected (require admin role)

### Bài 3: E-learning Platform

Tạo routing:
- `/courses` - Course list (với filter query params)
- `/courses/:courseId` - Course detail
- `/courses/:courseId/lessons/:lessonId` - Lesson viewer
- `/my-courses` - User's enrolled courses (protected)
- `/profile` - User profile (protected)

### Bài 4: Social Media App

Tạo routing:
- `/` - Feed
- `/profile/:username` - User profile
- `/posts/:postId` - Post detail
- `/messages` - Direct messages (protected)
- `/notifications` - Notifications (protected)
- `/settings` - Settings (protected)

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ SPA và client-side routing
✅ React Router DOM setup
✅ Dynamic routes với URL parameters
✅ Nested routes và layout
✅ Navigation với Link, NavLink, useNavigate
✅ Protected routes cho authentication
✅ Query parameters với useSearchParams

**Bài tiếp theo:** [09 - Call API Backend Java](09-call-api-backend-java.md)

---

> 💬 **Tips**: Dùng **NavLink** thay vì **Link** khi cần active styling. Luôn dùng **ProtectedRoute** component để bảo vệ routes thay vì kiểm tra auth trong từng page!
