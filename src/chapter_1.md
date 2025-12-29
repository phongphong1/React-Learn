# 01 - Kiến Thức Nền Tảng React

## 📚 Mục Lục

1. [Giới thiệu về React](#giới-thiệu-về-react)
2. [Virtual DOM là gì?](#virtual-dom-là-gì)
3. [Tại sao nên dùng React thay vì JavaScript thuần?](#tại-sao-nên-dùng-react)
4. [So sánh MVC (Spring Boot) vs Component-based (React)](#so-sánh-mvc-vs-component-based)
5. [Ví dụ thực tế](#ví-dụ-thực-tế)
6. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🎯 Giới thiệu về React

React là một **thư viện JavaScript** được phát triển bởi Facebook (Meta) để xây dựng giao diện người dùng (UI). React tập trung vào việc:

- **Xây dựng UI theo component**: Chia nhỏ giao diện thành các thành phần độc lập, có thể tái sử dụng
- **Quản lý state hiệu quả**: Tự động cập nhật UI khi dữ liệu thay đổi
- **Hiệu năng cao**: Sử dụng Virtual DOM để tối ưu hóa việc render

> 💡 **So sánh với Java**: Nếu Spring Boot giúp bạn xây dựng backend với các **Bean**, **Service**, **Controller** độc lập, thì React giúp bạn xây dựng frontend với các **Component** độc lập.

---

## 🌳 Virtual DOM là gì?

### Định nghĩa

**Virtual DOM** (DOM ảo) là một bản sao nhẹ của DOM thực (Real DOM) được lưu trong bộ nhớ. React sử dụng Virtual DOM để:

1. **Tính toán sự khác biệt** (diffing algorithm)
2. **Chỉ cập nhật những phần thay đổi** thay vì render lại toàn bộ trang

### Cách hoạt động

```
[State thay đổi]
    ↓
[Tạo Virtual DOM mới]
    ↓
[So sánh với Virtual DOM cũ (Diffing)]
    ↓
[Xác định những gì cần thay đổi]
    ↓
[Cập nhật Real DOM (chỉ phần thay đổi)]
```

### So sánh với JavaScript thuần

| **JavaScript Thuần** | **React với Virtual DOM** |
|----------------------|---------------------------|
| Thao tác trực tiếp với DOM | Thao tác với Virtual DOM |
| Phải tự quản lý update | React tự động quản lý update |
| Re-render toàn bộ hoặc phức tạp | Chỉ re-render phần cần thiết |
| Hiệu năng kém khi ứng dụng lớn | Hiệu năng tối ưu |

### Ví dụ minh họa

```jsx
// JavaScript thuần - Phải thao tác DOM thủ công
function updateCounter() {
  const counterElement = document.getElementById('counter');
  let count = parseInt(counterElement.innerText);
  count++;
  counterElement.innerText = count; // Thao tác trực tiếp với DOM
}

// React - Chỉ cần thay đổi state, React tự update DOM
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Tăng</button>
    </div>
  );
  // React tự động phát hiện thay đổi và update DOM
}
```

---

## 💡 Tại sao nên dùng React?

### 1. **Component-based Architecture**
- Chia nhỏ UI thành các component có thể tái sử dụng
- Dễ bảo trì và mở rộng
- Tương tự việc bạn chia nhỏ code Java thành các Class/Interface

### 2. **Declarative (Khai báo) thay vì Imperative (Mệnh lệnh)**

```javascript
// JavaScript thuần (Imperative - Mệnh lệnh)
const button = document.createElement('button');
button.innerText = 'Click me';
button.addEventListener('click', handleClick);
document.body.appendChild(button);

// React (Declarative - Khai báo)
<button onClick={handleClick}>Click me</button>
```

> 💡 **Giống Java Stream API**: Imperative như `for loop` cổ điển, Declarative như `stream().map().filter()`

### 3. **Hệ sinh thái mạnh mẽ**
- React Router (Routing)
- Redux/Zustand (State Management)
- React Query (Data Fetching)
- Next.js (Framework)

### 4. **Hiệu năng cao**
- Virtual DOM
- Code Splitting
- Lazy Loading

---

## 🏗️ So sánh MVC vs Component-based

### Mô hình MVC (Spring Boot)

```
┌─────────────┐
│   View      │ ← Hiển thị (Thymeleaf, JSP)
│  (JSP/HTML) │
└──────┬──────┘
       │
┌──────▼──────┐
│ Controller  │ ← Xử lý request, gọi Service
│ (@RestCtrl) │
└──────┬──────┘
       │
┌──────▼──────┐
│   Service   │ ← Business Logic
│ (@Service)  │
└──────┬──────┘
       │
┌──────▼──────┐
│   Model     │ ← Dữ liệu (Entity)
│  (Entity)   │
└─────────────┘
```

### Mô hình Component-based (React)

```
┌─────────────────────────────┐
│      App Component          │
│  (Quản lý state + logic)    │
└────────────┬────────────────┘
             │
     ┌───────┴────────┐
     │                │
┌────▼─────┐   ┌─────▼────┐
│ Header   │   │  Content │
│Component │   │Component │
└──────────┘   └────┬─────┘
                    │
            ┌───────┴────────┐
            │                │
      ┌─────▼─────┐   ┌─────▼─────┐
      │   List    │   │   Detail  │
      │ Component │   │ Component │
      └───────────┘   └───────────┘
```

### Sự khác biệt chính

| **MVC (Spring Boot)** | **Component-based (React)** |
|-----------------------|----------------------------|
| Tách biệt rõ ràng View, Controller, Model | UI = Component (chứa cả logic lẫn view) |
| Server-side rendering | Client-side rendering (chủ yếu) |
| Page reload khi chuyển trang | Single Page Application (SPA) |
| Stateless (REST API) | Stateful (State management) |

### Ví dụ so sánh

**Spring Boot Controller:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        UserDTO user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
}
```

**React Component tương đương:**
```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Gọi API giống như gọi Service trong Spring
    axios.get(`/api/users/${userId}`)
      .then(response => setUser(response.data))
      .catch(error => console.error(error));
  }, [userId]);

  if (!user) return <div>Loading...</div>;

  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

---

## 🔥 Ví dụ thực tế

### Tạo Counter App đơn giản

```jsx
import { useState } from 'react';

function Counter() {
  // State giống như biến instance trong Java Class
  const [count, setCount] = useState(0);

  // Method xử lý logic
  const handleIncrement = () => {
    setCount(count + 1);
  };

  const handleDecrement = () => {
    setCount(count - 1);
  };

  const handleReset = () => {
    setCount(0);
  };

  return (
    <div className="counter">
      <h1>Counter: {count}</h1>

      <div className="buttons">
        <button onClick={handleIncrement}>➕ Tăng</button>
        <button onClick={handleDecrement}>➖ Giảm</button>
        <button onClick={handleReset}>🔄 Reset</button>
      </div>

      {/* Conditional Rendering */}
      {count > 10 && (
        <p style={{ color: 'red' }}>
          ⚠️ Số đếm đã vượt quá 10!
        </p>
      )}
    </div>
  );
}

export default Counter;
```

### So sánh với Java Class

```java
// Tương đương trong Java
public class Counter {
    private int count = 0; // Giống state trong React

    public void increment() {
        this.count++;
        render(); // React tự động làm việc này
    }

    public void decrement() {
        this.count--;
        render();
    }

    public void reset() {
        this.count = 0;
        render();
    }

    private void render() {
        // Cập nhật UI - React làm tự động qua Virtual DOM
        System.out.println("Counter: " + count);
    }
}
```

---

## 📝 Bài tập thực hành

### Bài 1: Phân tích kiến trúc

Hãy so sánh một ứng dụng quản lý sản phẩm:

**Spring Boot:**
- ProductController
- ProductService
- ProductRepository
- Product Entity

**React sẽ có:**
- ??? Component nào?
- ??? State nào cần quản lý?

### Bài 2: Tư duy Component

Hãy vẽ sơ đồ component tree cho một trang Blog đơn giản gồm:
- Header (Logo + Menu)
- Danh sách bài viết (List)
- Chi tiết bài viết (Detail)
- Footer

### Bài 3: Code Challenge

Hãy tạo một Component `TodoCounter` hiển thị:
- Tổng số todo
- Số todo đã hoàn thành
- Số todo chưa hoàn thành
- Nút để reset tất cả

**Gợi ý cấu trúc:**
```jsx
function TodoCounter() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Học React', completed: false },
    { id: 2, text: 'Làm bài tập', completed: true }
  ]);

  // Viết logic của bạn ở đây

  return (
    // JSX của bạn ở đây
  );
}
```

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ Virtual DOM giúp React render hiệu quả hơn JS thuần
✅ React sử dụng Component-based architecture thay vì MVC truyền thống
✅ Declarative programming giúp code dễ đọc, dễ maintain hơn
✅ React phù hợp cho SPA, trong khi Spring Boot phù hợp cho backend API

**Bài tiếp theo:** [02 - Setup môi trường với Vite](02-setup-moi-truong-vite.md)

---

> 💬 **Tips**: Hãy nghĩ về React Component như một **Java Bean** - nó encapsulate cả data (state) và behavior (methods), nhưng thay vì trả về JSON, nó trả về UI!
