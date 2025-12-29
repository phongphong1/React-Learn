# 03 - JSX và Props

## 📚 Mục Lục

1. [JSX là gì?](#jsx-là-gì)
2. [Cú pháp JSX cơ bản](#cú-pháp-jsx-cơ-bản)
3. [Props - Truyền dữ liệu giữa Component](#props-truyền-dữ-liệu-giữa-component)
4. [Props vs Parameters trong Java](#props-vs-parameters-trong-java)
5. [Props Destructuring](#props-destructuring)
6. [Props Validation với PropTypes](#props-validation-với-proptypes)
7. [Children Props](#children-props)
8. [Ví dụ thực tế](#ví-dụ-thực-tế)
9. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🎨 JSX là gì?

**JSX** (JavaScript XML) là một syntax extension của JavaScript, cho phép bạn viết HTML trong code JavaScript.

### JSX không phải là HTML

```jsx
// JSX (React)
const element = <h1 className="title">Hello React</h1>;

// Được compile thành JavaScript thuần:
const element = React.createElement(
  'h1',
  { className: 'title' },
  'Hello React'
);
```

### Lợi ích của JSX

| **Lợi ích** | **Giải thích** |
|------------|----------------|
| **Trực quan** | Viết UI giống HTML, dễ đọc, dễ hiểu |
| **Type-safe** | JavaScript compiler kiểm tra lỗi ngay |
| **Powerful** | Có thể nhúng biểu thức JavaScript |
| **Component-based** | Tái sử dụng component dễ dàng |

> 💡 **So sánh với Java**: JSX giống như **Thymeleaf** hoặc **JSP**, nhưng thay vì chạy trên server, nó chạy trên client và được compile thành JavaScript.

---

## 📝 Cú pháp JSX cơ bản

### 1. Nhúng biểu thức JavaScript

Sử dụng `{}` để nhúng JavaScript expression:

```jsx
function Welcome() {
  const name = "John";
  const age = 25;
  const isAdmin = true;

  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Age: {age}</p>
      <p>Role: {isAdmin ? "Admin" : "User"}</p>
      <p>Year: {2024 + 1}</p>
    </div>
  );
}
```

### 2. Attributes trong JSX

```jsx
function Image() {
  const imageUrl = "https://example.com/avatar.jpg";
  const altText = "User Avatar";

  return (
    <div>
      {/* Lưu ý: className thay vì class */}
      <img
        src={imageUrl}
        alt={altText}
        className="avatar"
        style={{ width: '100px', borderRadius: '50%' }}
      />

      {/* camelCase cho HTML attributes */}
      <input
        type="text"
        maxLength={50}
        autoComplete="off"
      />
    </div>
  );
}
```

### 3. Conditional Rendering

```jsx
function UserGreeting({ isLoggedIn, username }) {
  // Cách 1: if-else (trong function)
  if (!isLoggedIn) {
    return <h1>Please login</h1>;
  }

  return (
    <div>
      <h1>Welcome back, {username}!</h1>

      {/* Cách 2: Ternary operator */}
      {isLoggedIn ? (
        <button>Logout</button>
      ) : (
        <button>Login</button>
      )}

      {/* Cách 3: Logical && */}
      {isLoggedIn && <p>You have 5 notifications</p>}
    </div>
  );
}
```

### 4. Render Lists (Arrays)

```jsx
function ProductList() {
  const products = [
    { id: 1, name: "Laptop", price: 1000 },
    { id: 2, name: "Phone", price: 500 },
    { id: 3, name: "Tablet", price: 300 }
  ];

  return (
    <div>
      <h2>Products</h2>
      <ul>
        {products.map(product => (
          // Key là bắt buộc khi render list
          <li key={product.id}>
            {product.name} - ${product.price}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

> ⚠️ **Lưu ý**: Luôn cần `key` prop khi render list để React track được từng item.

### 5. JSX Rules (Quy tắc JSX)

```jsx
function MyComponent() {
  // ❌ SAI: Không thể return nhiều element gốc
  // return (
  //   <h1>Title</h1>
  //   <p>Description</p>
  // );

  // ✅ ĐÚNG: Wrap trong 1 parent element
  return (
    <div>
      <h1>Title</h1>
      <p>Description</p>
    </div>
  );

  // ✅ ĐÚNG: Hoặc dùng Fragment <>...</>
  return (
    <>
      <h1>Title</h1>
      <p>Description</p>
    </>
  );
}
```

---

## 🎁 Props - Truyền dữ liệu giữa Component

**Props** (Properties) là cách truyền dữ liệu từ **Parent Component** xuống **Child Component**.

### Cách truyền Props

```jsx
// Parent Component
function App() {
  return (
    <div>
      <UserCard
        name="John Doe"
        age={25}
        email="john@example.com"
        isActive={true}
      />
    </div>
  );
}

// Child Component
function UserCard(props) {
  return (
    <div className="user-card">
      <h2>{props.name}</h2>
      <p>Age: {props.age}</p>
      <p>Email: {props.email}</p>
      <p>Status: {props.isActive ? "Active" : "Inactive"}</p>
    </div>
  );
}
```

### Props là Read-only (Immutable)

```jsx
function UserCard(props) {
  // ❌ SAI: Không thể thay đổi props
  // props.name = "Jane Doe"; // Error!

  // ✅ ĐÚNG: Chỉ được đọc
  const displayName = props.name.toUpperCase();

  return <h2>{displayName}</h2>;
}
```

---

## 🔄 Props vs Parameters trong Java

### So sánh trực quan

**Java Method:**
```java
public class UserService {
    // Parameters trong method
    public void displayUser(String name, int age, String email) {
        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
        System.out.println("Email: " + email);
    }

    // Gọi method
    public void run() {
        displayUser("John", 25, "john@example.com");
    }
}
```

**React Component:**
```jsx
// Props trong component
function UserCard(props) {
  return (
    <div>
      <p>Name: {props.name}</p>
      <p>Age: {props.age}</p>
      <p>Email: {props.email}</p>
    </div>
  );
}

// Sử dụng component
function App() {
  return <UserCard name="John" age={25} email="john@example.com" />;
}
```

### Bảng so sánh

| **Java** | **React** |
|----------|-----------|
| Method parameters | Component props |
| `methodName(param1, param2)` | `<Component prop1={} prop2={} />` |
| Pass by value/reference | Props are immutable |
| Type checking by compiler | PropTypes (optional) |
| Overloading methods | Default props |

---

## 🎯 Props Destructuring

Thay vì viết `props.name`, `props.age`, bạn có thể destructure props:

### Cách 1: Destructuring trong parameters

```jsx
// Before
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.age}</p>
    </div>
  );
}

// After (Destructuring)
function UserCard({ name, age, email, isActive }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      <p>Status: {isActive ? "Active" : "Inactive"}</p>
    </div>
  );
}
```

### Cách 2: Destructuring trong body

```jsx
function UserCard(props) {
  const { name, age, email, isActive } = props;

  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
}
```

### Default Props (Giá trị mặc định)

```jsx
function Button({ text, color = "blue", size = "medium" }) {
  return (
    <button className={`btn btn-${color} btn-${size}`}>
      {text}
    </button>
  );
}

// Sử dụng
<Button text="Click me" />                        // color=blue, size=medium
<Button text="Submit" color="green" size="large" />
```

> 💡 **Giống Java**: Default props như **default parameter values** trong Java 8+

---

## ✅ Props Validation với PropTypes

PropTypes giúp validate type của props (giống type checking trong Java).

### Cài đặt PropTypes

```bash
npm install prop-types
```

### Sử dụng PropTypes

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email, isActive, role }) {
  return (
    <div className="user-card">
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      <p>Role: {role}</p>
    </div>
  );
}

// Định nghĩa PropTypes
UserCard.propTypes = {
  name: PropTypes.string.isRequired,      // Bắt buộc, phải là string
  age: PropTypes.number.isRequired,       // Bắt buộc, phải là number
  email: PropTypes.string,                // Optional, phải là string
  isActive: PropTypes.bool,               // Optional, phải là boolean
  role: PropTypes.oneOf(['admin', 'user', 'guest']) // Chỉ nhận 3 giá trị
};

// Default Props
UserCard.defaultProps = {
  isActive: false,
  role: 'user'
};
```

### PropTypes phổ biến

```jsx
Component.propTypes = {
  // Primitive types
  name: PropTypes.string,
  age: PropTypes.number,
  isActive: PropTypes.bool,
  callback: PropTypes.func,
  data: PropTypes.object,
  items: PropTypes.array,

  // Specific types
  element: PropTypes.element,              // React element
  node: PropTypes.node,                    // Anything renderable

  // Enum
  status: PropTypes.oneOf(['pending', 'approved', 'rejected']),

  // Array of specific type
  numbers: PropTypes.arrayOf(PropTypes.number),

  // Object with specific shape
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string
  }),

  // Required
  requiredFunc: PropTypes.func.isRequired,
};
```

> 💡 **So sánh với Java**: PropTypes giống như **method signature** với type declarations trong Java.

---

## 👶 Children Props

`children` là một prop đặc biệt, chứa nội dung bên trong component.

### Cú pháp

```jsx
// Parent
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

// Sử dụng
function App() {
  return (
    <Card>
      <h2>Card Title</h2>
      <p>This is card content</p>
      <button>Action</button>
    </Card>
  );
}
```

### Ví dụ thực tế: Layout Component

```jsx
function Layout({ children }) {
  return (
    <div className="layout">
      <header>
        <h1>My Website</h1>
      </header>

      <main>
        {children}
      </main>

      <footer>
        <p>&copy; 2024 My Website</p>
      </footer>
    </div>
  );
}

// Sử dụng
function App() {
  return (
    <Layout>
      <h2>Welcome to HomePage</h2>
      <p>This is the main content</p>
    </Layout>
  );
}
```

> 💡 **So sánh với Java**: `children` giống như **Template Method Pattern** - parent định nghĩa structure, child cung cấp implementation.

---

## 🔥 Ví dụ thực tế

### 1. Product Card Component

```jsx
import PropTypes from 'prop-types';

function ProductCard({ product, onAddToCart }) {
  const { id, name, price, image, inStock } = product;

  return (
    <div className="product-card">
      <img src={image} alt={name} />

      <h3>{name}</h3>

      <div className="price">
        ${price.toFixed(2)}
      </div>

      {inStock ? (
        <button
          className="btn-primary"
          onClick={() => onAddToCart(id)}
        >
          Add to Cart
        </button>
      ) : (
        <button className="btn-disabled" disabled>
          Out of Stock
        </button>
      )}
    </div>
  );
}

ProductCard.propTypes = {
  product: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    price: PropTypes.number.isRequired,
    image: PropTypes.string.isRequired,
    inStock: PropTypes.bool
  }).isRequired,
  onAddToCart: PropTypes.func.isRequired
};

ProductCard.defaultProps = {
  product: {
    inStock: true
  }
};

export default ProductCard;
```

### 2. Sử dụng ProductCard

```jsx
function ProductList() {
  const products = [
    { id: 1, name: "Laptop", price: 999.99, image: "/laptop.jpg", inStock: true },
    { id: 2, name: "Phone", price: 599.99, image: "/phone.jpg", inStock: false },
    { id: 3, name: "Tablet", price: 399.99, image: "/tablet.jpg", inStock: true }
  ];

  const handleAddToCart = (productId) => {
    console.log(`Added product ${productId} to cart`);
  };

  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </div>
  );
}
```

### 3. Button Component với Children

```jsx
function Button({ children, variant = "primary", size = "medium", onClick, disabled = false }) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

Button.propTypes = {
  children: PropTypes.node.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  onClick: PropTypes.func,
  disabled: PropTypes.bool
};

// Sử dụng
<Button variant="primary" size="large" onClick={() => alert('Clicked!')}>
  Click Me
</Button>
```

---

## 📝 Bài tập thực hành

### Bài 1: User Profile Card

Tạo component `UserProfile` nhận props:
- `user` (object): { name, age, avatar, bio, isOnline }
- Hiển thị avatar tròn
- Hiển thị badge "Online" nếu `isOnline = true`
- Sử dụng PropTypes

### Bài 2: Product List

Tạo component `ProductList` và `ProductItem`:
- `ProductList`: nhận array products, render list ProductItem
- `ProductItem`: hiển thị tên, giá, hình ảnh
- Thêm button "Add to Cart" (log ra console khi click)

### Bài 3: Alert Component

Tạo component `Alert` nhận props:
- `type`: "success" | "error" | "warning" | "info"
- `message`: string
- `onClose`: function (hiển thị nút X để đóng)
- `children`: nội dung bên trong (optional)

```jsx
// Sử dụng
<Alert type="success" message="Login successful!" onClose={handleClose} />

<Alert type="error">
  <strong>Error!</strong> Something went wrong.
</Alert>
```

### Bài 4: Container Component

Tạo `Container` component:
- Nhận `children` props
- Nhận `maxWidth`: "sm" | "md" | "lg" | "xl"
- Nhận `padding`: boolean (mặc định true)

```jsx
<Container maxWidth="lg" padding={true}>
  <h1>Content here</h1>
</Container>
```

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ JSX là cú pháp để viết UI trong JavaScript
✅ Props là cách truyền dữ liệu giữa components (giống method parameters)
✅ Props là immutable (read-only)
✅ Sử dụng destructuring để code gọn hơn
✅ PropTypes để validate type (giống type checking trong Java)
✅ Children props để tạo reusable layout components

**Bài tiếp theo:** [04 - State và useState](04-state-va-usestate.md)

---

> 💬 **Tips**: Props flow **one-way** (từ trên xuống), giống như method parameters trong Java. Nếu muốn child component thay đổi data của parent, hãy truyền callback function qua props!
