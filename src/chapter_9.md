# 09 - Call API Backend Java (Spring Boot)

## 📚 Mục Lục

1. [Kiến trúc Frontend-Backend](#kiến-trúc-frontend-backend)
2. [HTTP Methods và REST API](#http-methods-và-rest-api)
3. [Axios vs Fetch](#axios-vs-fetch)
4. [Setup Axios](#setup-axios)
5. [CORS Configuration](#cors-configuration)
6. [Authentication với JWT](#authentication-với-jwt)
7. [Error Handling](#error-handling)
8. [API Service Layer](#api-service-layer)
9. [Ví dụ thực tế](#ví-dụ-thực-tế)
10. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🏗️ Kiến trúc Frontend-Backend

### Mô hình Client-Server

```
┌─────────────────┐           ┌──────────────────┐
│   React App     │           │  Spring Boot     │
│  (Frontend)     │           │   (Backend)      │
│                 │           │                  │
│  - Components   │  HTTP     │  - Controllers   │
│  - State        │ ◄────────►│  - Services      │
│  - Axios/Fetch  │  Requests │  - Repositories  │
│                 │           │  - Database      │
└─────────────────┘           └──────────────────┘
     Port 3000                      Port 8080
```

### Communication Flow

```
User Action
    ↓
React Component
    ↓
Call API (Axios)
    ↓
HTTP Request → Spring Boot Controller
                    ↓
                Service Layer
                    ↓
                Repository
                    ↓
                Database (MySQL)
                    ↓
Response ← ← ← ← ← ←
    ↓
Update State
    ↓
Re-render UI
```

---

## 🌐 HTTP Methods và REST API

### HTTP Methods

| **Method** | **CRUD** | **Ý nghĩa** | **Spring Annotation** |
|-----------|----------|-------------|-----------------------|
| GET | Read | Lấy dữ liệu | `@GetMapping` |
| POST | Create | Tạo mới | `@PostMapping` |
| PUT | Update | Cập nhật toàn bộ | `@PutMapping` |
| PATCH | Update | Cập nhật một phần | `@PatchMapping` |
| DELETE | Delete | Xóa | `@DeleteMapping` |

### REST API Endpoints (Spring Boot)

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    // GET /api/users - Lấy tất cả users
    @GetMapping
    public List<UserDTO> getAllUsers() {
        return userService.findAll();
    }

    // GET /api/users/123 - Lấy user theo ID
    @GetMapping("/{id}")
    public UserDTO getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }

    // POST /api/users - Tạo user mới
    @PostMapping
    public UserDTO createUser(@RequestBody UserDTO user) {
        return userService.create(user);
    }

    // PUT /api/users/123 - Update user
    @PutMapping("/{id}")
    public UserDTO updateUser(@PathVariable Long id, @RequestBody UserDTO user) {
        return userService.update(id, user);
    }

    // DELETE /api/users/123 - Xóa user
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

---

## ⚔️ Axios vs Fetch

### So sánh

| **Tiêu chí** | **Axios** | **Fetch** |
|-------------|-----------|-----------|
| **Built-in** | ❌ Cần cài (npm) | ✅ Native browser |
| **JSON parsing** | ✅ Tự động | ❌ Phải `.json()` |
| **Request/Response interceptors** | ✅ Có | ❌ Không |
| **Timeout** | ✅ Có | ❌ Không (phải tự implement) |
| **Error handling** | ✅ Tốt hơn | ⚠️ Cần check `response.ok` |
| **Cancel request** | ✅ Có | ⚠️ AbortController |
| **Browser support** | ✅ Rộng | ⚠️ IE không support |

### Ví dụ so sánh

**Fetch:**
```jsx
// Fetch API
fetch('/api/users')
  .then(response => {
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    return response.json(); // Phải parse JSON thủ công
  })
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**Axios:**
```jsx
// Axios (Đơn giản hơn)
axios.get('/api/users')
  .then(response => console.log(response.data)) // Tự động parse JSON
  .catch(error => console.error(error));
```

> 💡 **Khuyến nghị**: Dùng **Axios** cho dự án thực tế vì có nhiều features và dễ dùng hơn.

---

## 🔧 Setup Axios

### 1. Cài đặt

```bash
npm install axios
```

### 2. Axios Instance với Base Config

```jsx
// src/services/api.js
import axios from 'axios';

// Tạo axios instance với config mặc định
const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8080/api',
  timeout: 10000, // 10 seconds
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request Interceptor (chạy trước mỗi request)
api.interceptors.request.use(
  (config) => {
    // Thêm JWT token vào header
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    console.log('Request:', config.method.toUpperCase(), config.url);
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response Interceptor (chạy sau mỗi response)
api.interceptors.response.use(
  (response) => {
    console.log('Response:', response.status, response.data);
    return response;
  },
  (error) => {
    // Xử lý error global
    if (error.response) {
      // Server trả về error response
      console.error('Error Response:', error.response.status, error.response.data);

      // 401 Unauthorized → Redirect to login
      if (error.response.status === 401) {
        localStorage.removeItem('token');
        window.location.href = '/login';
      }

      // 403 Forbidden
      if (error.response.status === 403) {
        alert('You do not have permission to access this resource');
      }
    } else if (error.request) {
      // Request được gửi nhưng không nhận được response
      console.error('No response received:', error.request);
    } else {
      // Lỗi khác
      console.error('Error:', error.message);
    }

    return Promise.reject(error);
  }
);

export default api;
```

### 3. Environment Variables

```bash
# .env.development
VITE_API_URL=http://localhost:8080/api

# .env.production
VITE_API_URL=https://api.yourdomain.com/api
```

---

## 🌍 CORS Configuration

### Vấn đề CORS

CORS (Cross-Origin Resource Sharing) error xảy ra khi:
- Frontend: `http://localhost:3000`
- Backend: `http://localhost:8080`
- ❌ Browser block request vì khác origin

### Giải pháp 1: Config CORS trong Spring Boot

```java
// Spring Boot - CorsConfig.java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000", "http://localhost:5173")
                    .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}

// Hoặc dùng @CrossOrigin trên Controller
@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "http://localhost:3000")
public class UserController {
    // ...
}
```

### Giải pháp 2: Proxy trong Vite (Development)

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        // rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
});
```

Khi dùng proxy:
```jsx
// Frontend chỉ cần gọi: /api/users
// Vite tự động forward đến: http://localhost:8080/api/users
axios.get('/api/users');
```

---

## 🔐 Authentication với JWT

### Flow JWT Authentication

```
1. User login
   → POST /api/auth/login { email, password }

2. Backend validate
   → Return { token: "jwt-token", user: {...} }

3. Frontend lưu token
   → localStorage.setItem('token', token)

4. Mọi request sau đó
   → Header: Authorization: Bearer jwt-token

5. Backend verify token
   → Return data hoặc 401 Unauthorized
```

### 1. Login API

```jsx
// src/services/authService.js
import api from './api';

export const authService = {
  // Login
  async login(email, password) {
    const response = await api.post('/auth/login', { email, password });
    const { token, user } = response.data;

    // Lưu token và user info
    localStorage.setItem('token', token);
    localStorage.setItem('user', JSON.stringify(user));

    return { token, user };
  },

  // Register
  async register(userData) {
    const response = await api.post('/auth/register', userData);
    return response.data;
  },

  // Logout
  logout() {
    localStorage.removeItem('token');
    localStorage.removeItem('user');
  },

  // Get current user
  getCurrentUser() {
    const userStr = localStorage.getItem('user');
    return userStr ? JSON.parse(userStr) : null;
  },

  // Check if logged in
  isLoggedIn() {
    return !!localStorage.getItem('token');
  }
};
```

### 2. Spring Boot JWT Controller

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthService authService;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest loginRequest) {
        try {
            String token = authService.authenticate(
                loginRequest.getEmail(),
                loginRequest.getPassword()
            );

            User user = authService.getUserByEmail(loginRequest.getEmail());

            return ResponseEntity.ok(new AuthResponse(token, user));
        } catch (Exception e) {
            return ResponseEntity.status(401)
                .body(new ErrorResponse("Invalid credentials"));
        }
    }

    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody RegisterRequest request) {
        User user = authService.register(request);
        return ResponseEntity.ok(user);
    }

    @GetMapping("/me")
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<User> getCurrentUser() {
        User user = authService.getCurrentUser();
        return ResponseEntity.ok(user);
    }
}
```

### 3. Sử dụng trong Component

```jsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { authService } from '../services/authService';

function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      setLoading(true);
      setError('');

      await authService.login(email, password);

      // Navigate to dashboard after successful login
      navigate('/dashboard');
    } catch (err) {
      setError(err.response?.data?.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>Login</h2>

      {error && <div className="error">{error}</div>}

      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
        required
      />

      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
        required
      />

      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

---

## ⚠️ Error Handling

### 1. Error Types

```jsx
axios.get('/api/users')
  .catch(error => {
    if (error.response) {
      // Server trả về response với status code khác 2xx
      console.log('Status:', error.response.status);
      console.log('Data:', error.response.data);
      console.log('Headers:', error.response.headers);

      // 400 Bad Request
      // 401 Unauthorized
      // 403 Forbidden
      // 404 Not Found
      // 500 Internal Server Error

    } else if (error.request) {
      // Request được gửi nhưng không nhận được response
      console.log('Request:', error.request);
      // Network error, server down, timeout

    } else {
      // Lỗi khác (setup request, etc.)
      console.log('Error:', error.message);
    }
  });
```

### 2. Custom Error Handler

```jsx
// src/utils/errorHandler.js
export const handleAPIError = (error) => {
  if (error.response) {
    const { status, data } = error.response;

    switch (status) {
      case 400:
        return data.message || 'Bad Request';
      case 401:
        return 'Unauthorized. Please login again.';
      case 403:
        return 'You do not have permission to access this resource.';
      case 404:
        return 'Resource not found.';
      case 500:
        return 'Internal Server Error. Please try again later.';
      default:
        return data.message || 'An error occurred';
    }
  } else if (error.request) {
    return 'Network error. Please check your internet connection.';
  } else {
    return error.message || 'An unexpected error occurred';
  }
};

// Sử dụng
import { handleAPIError } from '../utils/errorHandler';

try {
  await api.get('/api/users');
} catch (error) {
  const errorMessage = handleAPIError(error);
  setError(errorMessage);
}
```

### 3. Spring Boot Error Response

```java
// ErrorResponse.java
@Data
public class ErrorResponse {
    private int status;
    private String message;
    private long timestamp;

    public ErrorResponse(int status, String message) {
        this.status = status;
        this.message = message;
        this.timestamp = System.currentTimeMillis();
    }
}

// GlobalExceptionHandler.java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(404, ex.getMessage());
        return ResponseEntity.status(404).body(error);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        ErrorResponse error = new ErrorResponse(400, ex.getMessage());
        return ResponseEntity.status(400).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(500, "Internal Server Error");
        return ResponseEntity.status(500).body(error);
    }
}
```

---

## 🏢 API Service Layer

Tổ chức code theo Service Layer pattern (giống Spring Boot).

### 1. User Service

```jsx
// src/services/userService.js
import api from './api';

export const userService = {
  // GET /api/users
  async getAllUsers() {
    const response = await api.get('/users');
    return response.data;
  },

  // GET /api/users/:id
  async getUserById(id) {
    const response = await api.get(`/users/${id}`);
    return response.data;
  },

  // POST /api/users
  async createUser(userData) {
    const response = await api.post('/users', userData);
    return response.data;
  },

  // PUT /api/users/:id
  async updateUser(id, userData) {
    const response = await api.put(`/users/${id}`, userData);
    return response.data;
  },

  // DELETE /api/users/:id
  async deleteUser(id) {
    await api.delete(`/users/${id}`);
  },

  // GET /api/users/search?q=query
  async searchUsers(query) {
    const response = await api.get('/users/search', {
      params: { q: query }
    });
    return response.data;
  }
};
```

### 2. Product Service

```jsx
// src/services/productService.js
import api from './api';

export const productService = {
  async getProducts(page = 1, limit = 10, category = null) {
    const response = await api.get('/products', {
      params: { page, limit, category }
    });
    return response.data;
  },

  async getProductById(id) {
    const response = await api.get(`/products/${id}`);
    return response.data;
  },

  async createProduct(productData) {
    // Upload ảnh
    const formData = new FormData();
    formData.append('name', productData.name);
    formData.append('price', productData.price);
    formData.append('image', productData.image); // File object

    const response = await api.post('/products', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });
    return response.data;
  },

  async updateProduct(id, productData) {
    const response = await api.put(`/products/${id}`, productData);
    return response.data;
  },

  async deleteProduct(id) {
    await api.delete(`/products/${id}`);
  }
};
```

### 3. Sử dụng trong Component

```jsx
import { useState, useEffect } from 'react';
import { userService } from '../services/userService';
import { handleAPIError } from '../utils/errorHandler';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');

  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    try {
      setLoading(true);
      const data = await userService.getAllUsers();
      setUsers(data);
    } catch (err) {
      setError(handleAPIError(err));
    } finally {
      setLoading(false);
    }
  };

  const handleDelete = async (userId) => {
    if (!window.confirm('Are you sure?')) return;

    try {
      await userService.deleteUser(userId);
      // Refresh list
      fetchUsers();
    } catch (err) {
      alert(handleAPIError(err));
    }
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>Users</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.id}</td>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>
                <button onClick={() => handleDelete(user.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 🔥 Ví dụ thực tế

### Complete CRUD với Spring Boot Backend

```jsx
// src/pages/ProductManagement.jsx
import { useState, useEffect } from 'react';
import { productService } from '../services/productService';

function ProductManagement() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [editingProduct, setEditingProduct] = useState(null);
  const [formData, setFormData] = useState({
    name: '',
    price: 0,
    category: '',
    description: ''
  });

  useEffect(() => {
    fetchProducts();
  }, []);

  const fetchProducts = async () => {
    try {
      setLoading(true);
      const data = await productService.getProducts();
      setProducts(data);
    } catch (error) {
      console.error('Fetch error:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      if (editingProduct) {
        // Update
        await productService.updateProduct(editingProduct.id, formData);
      } else {
        // Create
        await productService.createProduct(formData);
      }

      // Reset form
      setFormData({ name: '', price: 0, category: '', description: '' });
      setEditingProduct(null);

      // Refresh list
      fetchProducts();
    } catch (error) {
      alert('Error: ' + error.message);
    }
  };

  const handleEdit = (product) => {
    setEditingProduct(product);
    setFormData({
      name: product.name,
      price: product.price,
      category: product.category,
      description: product.description
    });
  };

  const handleDelete = async (id) => {
    if (!window.confirm('Delete this product?')) return;

    try {
      await productService.deleteProduct(id);
      fetchProducts();
    } catch (error) {
      alert('Delete error: ' + error.message);
    }
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Product Management</h1>

      {/* Form */}
      <form onSubmit={handleSubmit}>
        <input
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          placeholder="Name"
          required
        />

        <input
          type="number"
          value={formData.price}
          onChange={(e) => setFormData({ ...formData, price: parseFloat(e.target.value) })}
          placeholder="Price"
          required
        />

        <input
          value={formData.category}
          onChange={(e) => setFormData({ ...formData, category: e.target.value })}
          placeholder="Category"
        />

        <textarea
          value={formData.description}
          onChange={(e) => setFormData({ ...formData, description: e.target.value })}
          placeholder="Description"
        />

        <button type="submit">
          {editingProduct ? 'Update' : 'Create'} Product
        </button>

        {editingProduct && (
          <button type="button" onClick={() => {
            setEditingProduct(null);
            setFormData({ name: '', price: 0, category: '', description: '' });
          }}>
            Cancel
          </button>
        )}
      </form>

      {/* Product List */}
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Price</th>
            <th>Category</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {products.map(product => (
            <tr key={product.id}>
              <td>{product.name}</td>
              <td>${product.price}</td>
              <td>{product.category}</td>
              <td>
                <button onClick={() => handleEdit(product)}>Edit</button>
                <button onClick={() => handleDelete(product.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 📝 Bài tập thực hành

### Bài 1: User Management

Tạo trang quản lý user:
- Fetch danh sách users từ Spring Boot backend
- CRUD operations: Create, Read, Update, Delete
- Search users by name/email
- Pagination
- Error handling

### Bài 2: Blog Post với Comments

- Fetch posts với comments
- Create new post
- Add comment to post
- Edit/Delete own posts
- JWT authentication required

### Bài 3: Shopping Cart

- Add to cart (POST /api/cart)
- Get cart items (GET /api/cart)
- Update quantity (PUT /api/cart/:id)
- Remove from cart (DELETE /api/cart/:id)
- Checkout (POST /api/orders)

### Bài 4: File Upload

- Upload avatar (POST /api/users/:id/avatar)
- Upload multiple product images
- Display uploaded images
- Spring Boot handle multipart/form-data

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ Kiến trúc Frontend-Backend với React và Spring Boot
✅ HTTP Methods và REST API
✅ Setup Axios với interceptors
✅ CORS configuration
✅ JWT Authentication flow
✅ Error handling best practices
✅ API Service Layer pattern

**Bài tiếp theo:** [10 - Project Thực Hành FitHub Mini](10-project-thuc-hanh-fithub-mini.md)

---

> 💬 **Tips**: Luôn tạo **Axios instance** với interceptors để xử lý JWT token và error handling global. Tổ chức code theo **Service Layer** giống Spring Boot để dễ maintain!
