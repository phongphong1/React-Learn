# 04 - State và useState Hook

## 📚 Mục Lục

1. [State là gì?](#state-là-gì)
2. [useState Hook](#usestate-hook)
3. [Cơ chế Re-render](#cơ-chế-re-render)
4. [State vs Props](#state-vs-props)
5. [State với Object và Array](#state-với-object-và-array)
6. [Multiple State Variables](#multiple-state-variables)
7. [State Lifting (Nâng State lên)](#state-lifting)
8. [Ví dụ thực tế](#ví-dụ-thực-tế)
9. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🎯 State là gì?

**State** là dữ liệu nội bộ của một component, có thể **thay đổi theo thời gian** và **trigger re-render** khi thay đổi.

### Đặc điểm của State

| **Đặc điểm** | **Giải thích** |
|-------------|----------------|
| **Mutable** | Có thể thay đổi (khác với Props là immutable) |
| **Private** | Chỉ component sở hữu mới quản lý được |
| **Reactive** | Khi state thay đổi → Component re-render |
| **Local** | Mỗi instance component có state riêng |

> 💡 **So sánh với Java**: State giống như **instance variable** trong Java Class. Mỗi object có state riêng, và khi state thay đổi, UI tương ứng cũng thay đổi.

```java
// Java Class
public class Counter {
    private int count = 0; // Instance variable (giống State)

    public void increment() {
        this.count++; // Thay đổi state
        render();     // Update UI
    }
}
```

```jsx
// React Component
function Counter() {
  const [count, setCount] = useState(0); // State

  const increment = () => {
    setCount(count + 1); // Thay đổi state → Auto re-render
  };

  return <button onClick={increment}>Count: {count}</button>;
}
```

---

## 🪝 useState Hook

`useState` là một Hook cho phép bạn thêm state vào functional component.

### Cú pháp cơ bản

```jsx
import { useState } from 'react';

function MyComponent() {
  // Syntax: const [state, setState] = useState(initialValue);
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Phân tích useState

```jsx
const [count, setCount] = useState(0);
//     ↓       ↓              ↓
//   State  Setter      Initial Value
//  Variable Function
```

| **Thành phần** | **Vai trò** |
|---------------|------------|
| `count` | Biến state (giá trị hiện tại) |
| `setCount` | Function để update state |
| `useState(0)` | Hook với giá trị khởi tạo = 0 |

### Các kiểu dữ liệu State

```jsx
function StateExamples() {
  // Number
  const [age, setAge] = useState(25);

  // String
  const [name, setName] = useState("John");

  // Boolean
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  // Array
  const [todos, setTodos] = useState([]);

  // Object
  const [user, setUser] = useState({ name: "", email: "" });

  // Null/Undefined
  const [data, setData] = useState(null);

  return <div>State Examples</div>;
}
```

---

## 🔄 Cơ chế Re-render

### Khi nào Component re-render?

1. **State thay đổi** (qua setState function)
2. **Props thay đổi** (từ parent component)
3. **Parent component re-render**

### Flow của Re-render

```
[User Action (Click button)]
         ↓
[Call setState(newValue)]
         ↓
[React schedule re-render]
         ↓
[Component function chạy lại]
         ↓
[Generate new Virtual DOM]
         ↓
[Compare với Virtual DOM cũ (Diffing)]
         ↓
[Update Real DOM (chỉ phần thay đổi)]
         ↓
[Browser render UI mới]
```

### Ví dụ minh họa

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  console.log("Component rendered, count =", count);
  // Log này chạy MỖI LẦN component re-render

  const handleClick = () => {
    setCount(count + 1); // Trigger re-render
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

// Output console:
// Component rendered, count = 0  (Lần mount đầu tiên)
// Component rendered, count = 1  (Click lần 1)
// Component rendered, count = 2  (Click lần 2)
```

### setState là Asynchronous

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
    console.log(count); // ⚠️ Vẫn là giá trị CŨ, chưa update!
    // Output: 0 (không phải 1)
  };

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

### setState với Callback Function

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // ❌ SAI: Nếu click nhanh, có thể bị mất update
    setCount(count + 1);
    setCount(count + 1); // Cả 2 đều dùng count = 0 → Kết quả = 1

    // ✅ ĐÚNG: Dùng callback để đảm bảo dùng giá trị mới nhất
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1); // Kết quả = 2
  };

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

---

## ⚖️ State vs Props

| **Tiêu chí** | **State** | **Props** |
|-------------|-----------|-----------|
| **Định nghĩa** | Dữ liệu nội bộ component | Dữ liệu truyền từ parent |
| **Mutable** | ✅ Có thể thay đổi (qua setState) | ❌ Immutable (read-only) |
| **Owner** | Component tự quản lý | Parent component quản lý |
| **Trigger re-render** | ✅ Có | ✅ Có (khi props mới khác cũ) |
| **Initial Value** | Đặt trong useState() | Truyền từ parent |

### So sánh với Java

```java
// Java Class
public class UserProfile {
    // State (instance variable)
    private int viewCount = 0;

    // Props (constructor parameters)
    private String username;
    private String email;

    public UserProfile(String username, String email) {
        this.username = username; // Props - không thay đổi
        this.email = email;
    }

    public void incrementView() {
        this.viewCount++; // State - có thể thay đổi
    }
}
```

```jsx
// React Component
function UserProfile({ username, email }) { // Props
  const [viewCount, setViewCount] = useState(0); // State

  const incrementView = () => {
    setViewCount(viewCount + 1); // Update state
  };

  return (
    <div>
      <h2>{username}</h2>       {/* Props - read-only */}
      <p>{email}</p>            {/* Props - read-only */}
      <p>Views: {viewCount}</p> {/* State - mutable */}
      <button onClick={incrementView}>Increment</button>
    </div>
  );
}
```

---

## 📦 State với Object và Array

### Update Object State

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: "",
    email: "",
    age: 0
  });

  // ❌ SAI: Mutate trực tiếp
  const handleChange = (field, value) => {
    user[field] = value; // Không trigger re-render!
    setUser(user);       // React không biết object đã thay đổi
  };

  // ✅ ĐÚNG: Tạo object mới (immutable update)
  const handleChange = (field, value) => {
    setUser({
      ...user,        // Copy tất cả properties cũ
      [field]: value  // Override property cụ thể
    });
  };

  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => handleChange('name', e.target.value)}
      />
      <input
        value={user.email}
        onChange={(e) => handleChange('email', e.target.value)}
      />
      <input
        type="number"
        value={user.age}
        onChange={(e) => handleChange('age', parseInt(e.target.value))}
      />

      <p>User: {JSON.stringify(user)}</p>
    </div>
  );
}
```

### Update Array State

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: "Learn React", completed: false }
  ]);

  // Thêm item mới
  const addTodo = (text) => {
    const newTodo = {
      id: Date.now(),
      text: text,
      completed: false
    };

    setTodos([...todos, newTodo]); // Spread operator
    // Hoặc: setTodos(prevTodos => [...prevTodos, newTodo]);
  };

  // Xóa item
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  // Update item
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };

  return (
    <div>
      <button onClick={() => addTodo("New Task")}>Add Todo</button>

      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

> ⚠️ **Quan trọng**: Luôn tạo **bản copy mới** khi update object/array state. Đừng mutate trực tiếp!

---

## 🔢 Multiple State Variables

### Cách 1: Nhiều useState

```jsx
function UserProfile() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [age, setAge] = useState(0);
  const [isActive, setIsActive] = useState(true);

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input value={age} onChange={(e) => setAge(e.target.value)} />
    </div>
  );
}
```

### Cách 2: Object State (Nhóm lại)

```jsx
function UserProfile() {
  const [user, setUser] = useState({
    name: "",
    email: "",
    age: 0,
    isActive: true
  });

  const updateField = (field, value) => {
    setUser(prev => ({ ...prev, [field]: value }));
  };

  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => updateField('name', e.target.value)}
      />
      <input
        value={user.email}
        onChange={(e) => updateField('email', e.target.value)}
      />
    </div>
  );
}
```

### Nên dùng cách nào?

| **Trường hợp** | **Nên dùng** |
|---------------|-------------|
| Các state độc lập nhau | Multiple useState |
| Các state liên quan (form data) | Object State |
| State phức tạp, nhiều tầng | useReducer (học sau) |

---

## ⬆️ State Lifting (Nâng State lên)

Khi **nhiều component cần chia sẻ cùng 1 state**, hãy nâng state lên **parent component gần nhất**.

### Vấn đề

```jsx
// ❌ Mỗi component có state riêng → Không đồng bộ
function ComponentA() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>A: {count}</button>;
}

function ComponentB() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>B: {count}</button>;
}
```

### Giải pháp: Lifting State Up

```jsx
// ✅ State ở parent, truyền xuống children qua props
function ParentComponent() {
  const [count, setCount] = useState(0); // State ở parent

  return (
    <div>
      <h2>Total: {count}</h2>
      <ComponentA count={count} setCount={setCount} />
      <ComponentB count={count} setCount={setCount} />
    </div>
  );
}

function ComponentA({ count, setCount }) {
  return <button onClick={() => setCount(count + 1)}>A: {count}</button>;
}

function ComponentB({ count, setCount }) {
  return <button onClick={() => setCount(count + 1)}>B: {count}</button>;
}
```

> 💡 **So sánh với Java**: Giống như khi bạn đưa shared variable lên class cha, thay vì để mỗi subclass có biến riêng.

---

## 🔥 Ví dụ thực tế

### 1. Shopping Cart

```jsx
import { useState } from 'react';

function ShoppingCart() {
  const [cart, setCart] = useState([]);

  const products = [
    { id: 1, name: "Laptop", price: 1000 },
    { id: 2, name: "Phone", price: 500 },
    { id: 3, name: "Tablet", price: 300 }
  ];

  const addToCart = (product) => {
    // Kiểm tra product đã có trong cart chưa
    const existingItem = cart.find(item => item.id === product.id);

    if (existingItem) {
      // Tăng quantity
      setCart(cart.map(item =>
        item.id === product.id
          ? { ...item, quantity: item.quantity + 1 }
          : item
      ));
    } else {
      // Thêm item mới
      setCart([...cart, { ...product, quantity: 1 }]);
    }
  };

  const removeFromCart = (productId) => {
    setCart(cart.filter(item => item.id !== productId));
  };

  const updateQuantity = (productId, newQuantity) => {
    if (newQuantity <= 0) {
      removeFromCart(productId);
      return;
    }

    setCart(cart.map(item =>
      item.id === productId
        ? { ...item, quantity: newQuantity }
        : item
    ));
  };

  const getTotalPrice = () => {
    return cart.reduce((total, item) => total + (item.price * item.quantity), 0);
  };

  return (
    <div className="shopping-cart">
      <h2>Products</h2>
      <div className="products">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <h3>{product.name}</h3>
            <p>${product.price}</p>
            <button onClick={() => addToCart(product)}>Add to Cart</button>
          </div>
        ))}
      </div>

      <h2>Cart ({cart.length} items)</h2>
      {cart.length === 0 ? (
        <p>Cart is empty</p>
      ) : (
        <>
          <ul>
            {cart.map(item => (
              <li key={item.id}>
                {item.name} - ${item.price} x
                <input
                  type="number"
                  value={item.quantity}
                  onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
                  min="1"
                />
                = ${item.price * item.quantity}
                <button onClick={() => removeFromCart(item.id)}>Remove</button>
              </li>
            ))}
          </ul>

          <h3>Total: ${getTotalPrice()}</h3>
        </>
      )}
    </div>
  );
}

export default ShoppingCart;
```

### 2. Form với Multiple Inputs

```jsx
function RegistrationForm() {
  const [formData, setFormData] = useState({
    username: "",
    email: "",
    password: "",
    confirmPassword: "",
    agreeTerms: false
  });

  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;

    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const validate = () => {
    const newErrors = {};

    if (!formData.username) newErrors.username = "Username is required";
    if (!formData.email) newErrors.email = "Email is required";
    if (!formData.password) newErrors.password = "Password is required";
    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = "Passwords don't match";
    }
    if (!formData.agreeTerms) newErrors.agreeTerms = "You must agree to terms";

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();

    if (validate()) {
      console.log("Form submitted:", formData);
      alert("Registration successful!");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          value={formData.username}
          onChange={handleChange}
          placeholder="Username"
        />
        {errors.username && <span className="error">{errors.username}</span>}
      </div>

      <div>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <input
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>

      <div>
        <input
          name="confirmPassword"
          type="password"
          value={formData.confirmPassword}
          onChange={handleChange}
          placeholder="Confirm Password"
        />
        {errors.confirmPassword && <span className="error">{errors.confirmPassword}</span>}
      </div>

      <div>
        <label>
          <input
            name="agreeTerms"
            type="checkbox"
            checked={formData.agreeTerms}
            onChange={handleChange}
          />
          I agree to terms and conditions
        </label>
        {errors.agreeTerms && <span className="error">{errors.agreeTerms}</span>}
      </div>

      <button type="submit">Register</button>
    </form>
  );
}
```

---

## 📝 Bài tập thực hành

### Bài 1: Counter với nhiều chức năng

Tạo Counter component với:
- State: `count` (khởi tạo = 0)
- Buttons: Increment (+1), Decrement (-1), Reset (về 0), Increment by 5 (+5)
- Hiển thị màu đỏ nếu count < 0, màu xanh nếu count > 10

### Bài 2: Todo List

Tạo Todo List với các chức năng:
- Thêm todo mới (input + button)
- Hiển thị danh sách todos
- Checkbox để đánh dấu completed (có gạch ngang)
- Button Delete để xóa todo
- Hiển thị tổng số todo và số todo đã hoàn thành

### Bài 3: Search và Filter

Tạo component với:
- Array products: `[{ id, name, price, category }, ...]`
- Input search (tìm theo name)
- Select filter category
- Hiển thị danh sách đã được filter

### Bài 4: Multi-step Form

Tạo form đăng ký 3 bước:
- Step 1: Username, Email
- Step 2: Password, Confirm Password
- Step 3: Phone, Address
- Buttons: Next, Previous, Submit
- State để track: currentStep, formData

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ State là dữ liệu nội bộ component, có thể thay đổi
✅ `useState` Hook để thêm state vào functional component
✅ setState trigger re-render, React tự động update UI
✅ State vs Props: State mutable, Props immutable
✅ Cách update object và array state (immutable)
✅ State Lifting để chia sẻ state giữa components

**Bài tiếp theo:** [05 - useEffect và Lifecycle](05-useeffect-va-lifecycle.md)

---

> 💬 **Tips**: setState là **asynchronous**. Nếu cần dùng state mới ngay sau khi set, hãy dùng **callback function** `setState(prev => prev + 1)` thay vì `setState(state + 1)`!
