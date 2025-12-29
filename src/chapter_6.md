# 06 - Form và Validation

## 📚 Mục Lục

1. [Form trong React](#form-trong-react)
2. [Controlled vs Uncontrolled Components](#controlled-vs-uncontrolled-components)
3. [Xử lý Form cơ bản](#xử-lý-form-cơ-bản)
4. [Validation thủ công](#validation-thủ-công)
5. [React Hook Form](#react-hook-form)
6. [Zod - Schema Validation](#zod-schema-validation)
7. [Kết hợp React Hook Form + Zod](#kết-hợp-react-hook-form-zod)
8. [Ví dụ thực tế](#ví-dụ-thực-tế)
9. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 📝 Form trong React

Form là một phần quan trọng trong mọi ứng dụng web. Trong React, có 2 cách xử lý form:

1. **Controlled Components** (Khuyến nghị)
2. **Uncontrolled Components** (Ít dùng)

> 💡 **So sánh với Spring Boot**: Form trong React giống như **@ModelAttribute** hoặc **@RequestBody** trong Spring Controller - nhận dữ liệu từ client và validate.

---

## 🎮 Controlled vs Uncontrolled Components

### Controlled Components

React **kiểm soát** giá trị của input thông qua **state**.

```jsx
function ControlledForm() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}  // Giá trị từ state
      onChange={(e) => setName(e.target.value)}  // Update state
    />
  );
}
```

**Đặc điểm:**
- ✅ React kiểm soát hoàn toàn giá trị
- ✅ Dễ validate real-time
- ✅ Dễ format input (uppercase, number only, etc.)
- ❌ Cần viết nhiều code hơn

### Uncontrolled Components

React **không** kiểm soát giá trị, sử dụng **ref** để lấy data khi cần.

```jsx
function UncontrolledForm() {
  const nameRef = useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(nameRef.current.value); // Lấy giá trị từ DOM
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} />  {/* Không có value và onChange */}
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Đặc điểm:**
- ✅ Ít code hơn
- ✅ Tương tự HTML form truyền thống
- ❌ Khó validate real-time
- ❌ Khó kiểm soát giá trị

### So sánh

| **Tiêu chí** | **Controlled** | **Uncontrolled** |
|-------------|----------------|------------------|
| **Data source** | React state | DOM |
| **Validation** | Real-time | On submit |
| **Code complexity** | Cao hơn | Thấp hơn |
| **Khuyến nghị** | ✅ Dùng cho hầu hết trường hợp | ⚠️ Chỉ dùng cho form đơn giản |

---

## 📋 Xử lý Form cơ bản

### 1. Single Input

```jsx
function SimpleForm() {
  const [email, setEmail] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault(); // Ngăn page reload
    console.log("Email:", email);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 2. Multiple Inputs

```jsx
function RegistrationForm() {
  const [formData, setFormData] = useState({
    username: "",
    email: "",
    password: ""
  });

  // Generic handler cho tất cả inputs
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Form data:", formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={formData.username}
        onChange={handleChange}
        placeholder="Username"
      />

      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />

      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="Password"
      />

      <button type="submit">Register</button>
    </form>
  );
}
```

### 3. Các loại Input khác

```jsx
function ComplexForm() {
  const [formData, setFormData] = useState({
    text: "",
    textarea: "",
    select: "",
    checkbox: false,
    radio: "",
    multiSelect: []
  });

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;

    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleMultiSelect = (e) => {
    const options = Array.from(e.target.selectedOptions, option => option.value);
    setFormData(prev => ({ ...prev, multiSelect: options }));
  };

  return (
    <form>
      {/* Text Input */}
      <input
        name="text"
        value={formData.text}
        onChange={handleChange}
      />

      {/* Textarea */}
      <textarea
        name="textarea"
        value={formData.textarea}
        onChange={handleChange}
      />

      {/* Select */}
      <select name="select" value={formData.select} onChange={handleChange}>
        <option value="">Choose...</option>
        <option value="option1">Option 1</option>
        <option value="option2">Option 2</option>
      </select>

      {/* Checkbox */}
      <label>
        <input
          name="checkbox"
          type="checkbox"
          checked={formData.checkbox}
          onChange={handleChange}
        />
        Accept terms
      </label>

      {/* Radio */}
      <label>
        <input
          name="radio"
          type="radio"
          value="option1"
          checked={formData.radio === 'option1'}
          onChange={handleChange}
        />
        Option 1
      </label>

      {/* Multi-select */}
      <select
        name="multiSelect"
        multiple
        value={formData.multiSelect}
        onChange={handleMultiSelect}
      >
        <option value="a">A</option>
        <option value="b">B</option>
        <option value="c">C</option>
      </select>
    </form>
  );
}
```

---

## ✅ Validation thủ công

### 1. Validation cơ bản

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState({});

  const validate = () => {
    const newErrors = {};

    // Email validation
    if (!email) {
      newErrors.email = "Email is required";
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      newErrors.email = "Email is invalid";
    }

    // Password validation
    if (!password) {
      newErrors.password = "Password is required";
    } else if (password.length < 6) {
      newErrors.password = "Password must be at least 6 characters";
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();

    if (validate()) {
      console.log("Form is valid!");
      // Submit data to backend
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>

      <button type="submit">Login</button>
    </form>
  );
}
```

### 2. Real-time Validation

```jsx
function RealTimeValidation() {
  const [username, setUsername] = useState("");
  const [usernameError, setUsernameError] = useState("");

  const handleUsernameChange = (e) => {
    const value = e.target.value;
    setUsername(value);

    // Real-time validation
    if (value.length === 0) {
      setUsernameError("");
    } else if (value.length < 3) {
      setUsernameError("Username must be at least 3 characters");
    } else if (!/^[a-zA-Z0-9]+$/.test(value)) {
      setUsernameError("Username can only contain letters and numbers");
    } else {
      setUsernameError("");
    }
  };

  return (
    <div>
      <input
        value={username}
        onChange={handleUsernameChange}
        placeholder="Username"
      />
      {usernameError && <span className="error">{usernameError}</span>}
    </div>
  );
}
```

---

## 🪝 React Hook Form

**React Hook Form** là thư viện giúp quản lý form dễ dàng, hiệu năng cao, ít re-render.

### Cài đặt

```bash
npm install react-hook-form
```

### 1. Basic Usage

```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const {
    register,     // Đăng ký input
    handleSubmit, // Handler cho submit
    formState: { errors } // Errors object
  } = useForm();

  const onSubmit = (data) => {
    console.log("Form data:", data);
    // data = { email: "...", password: "..." }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("email", {
          required: "Email is required",
          pattern: {
            value: /\S+@\S+\.\S+/,
            message: "Email is invalid"
          }
        })}
        placeholder="Email"
      />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        type="password"
        {...register("password", {
          required: "Password is required",
          minLength: {
            value: 6,
            message: "Password must be at least 6 characters"
          }
        })}
        placeholder="Password"
      />
      {errors.password && <span>{errors.password.message}</span>}

      <button type="submit">Login</button>
    </form>
  );
}
```

### 2. Validation Rules

```jsx
function AdvancedForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  return (
    <form onSubmit={handleSubmit(data => console.log(data))}>
      {/* Required */}
      <input {...register("username", { required: true })} />

      {/* Min/Max Length */}
      <input {...register("password", {
        minLength: 6,
        maxLength: 20
      })} />

      {/* Min/Max Value (for numbers) */}
      <input
        type="number"
        {...register("age", {
          min: 18,
          max: 100
        })}
      />

      {/* Pattern (Regex) */}
      <input {...register("phone", {
        pattern: /^[0-9]{10}$/
      })} />

      {/* Custom Validation */}
      <input {...register("confirmPassword", {
        validate: value => value === watch("password") || "Passwords don't match"
      })} />
    </form>
  );
}
```

### 3. Default Values

```jsx
function EditUserForm({ user }) {
  const { register, handleSubmit } = useForm({
    defaultValues: {
      username: user.username,
      email: user.email,
      bio: user.bio
    }
  });

  return <form>...</form>;
}
```

---

## 🔒 Zod - Schema Validation

**Zod** là thư viện schema validation cho TypeScript/JavaScript, tương tự **Bean Validation** trong Java.

### Cài đặt

```bash
npm install zod @hookform/resolvers
```

### 1. Define Schema

```jsx
import { z } from 'zod';

// Định nghĩa schema (giống DTO trong Java với @Valid)
const userSchema = z.object({
  username: z.string()
    .min(3, "Username must be at least 3 characters")
    .max(20, "Username must be at most 20 characters"),

  email: z.string()
    .email("Invalid email format"),

  password: z.string()
    .min(8, "Password must be at least 8 characters")
    .regex(/[A-Z]/, "Password must contain at least one uppercase letter")
    .regex(/[0-9]/, "Password must contain at least one number"),

  age: z.number()
    .int()
    .min(18, "You must be at least 18 years old")
    .max(100),

  website: z.string()
    .url("Must be a valid URL")
    .optional(),

  agreeTerms: z.boolean()
    .refine(val => val === true, "You must agree to terms")
});

// So sánh với Java DTO:
// public class UserDTO {
//     @NotBlank
//     @Size(min = 3, max = 20)
//     private String username;
//
//     @Email
//     private String email;
//
//     @Min(18) @Max(100)
//     private Integer age;
// }
```

### 2. Zod Validation Types

```jsx
import { z } from 'zod';

// String
z.string()
z.string().min(3).max(20)
z.string().email()
z.string().url()
z.string().uuid()
z.string().regex(/pattern/)

// Number
z.number()
z.number().int()
z.number().positive()
z.number().min(0).max(100)

// Boolean
z.boolean()

// Date
z.date()
z.date().min(new Date("2020-01-01"))

// Array
z.array(z.string())
z.array(z.number()).min(1).max(5)

// Object
z.object({
  name: z.string(),
  age: z.number()
})

// Enum
z.enum(["admin", "user", "guest"])

// Optional / Nullable
z.string().optional()
z.string().nullable()
z.string().nullish() // null hoặc undefined

// Union
z.union([z.string(), z.number()])

// Custom Validation
z.string().refine(val => val.length > 0, "Custom error message")
```

---

## 🚀 Kết hợp React Hook Form + Zod

Đây là cách **best practice** để xử lý form trong React!

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 1. Define Schema
const registrationSchema = z.object({
  username: z.string()
    .min(3, "Username must be at least 3 characters")
    .max(20, "Username must be at most 20 characters"),

  email: z.string()
    .email("Invalid email address"),

  password: z.string()
    .min(8, "Password must be at least 8 characters")
    .regex(/[A-Z]/, "Must contain uppercase letter")
    .regex(/[a-z]/, "Must contain lowercase letter")
    .regex(/[0-9]/, "Must contain number"),

  confirmPassword: z.string(),

  age: z.number()
    .int()
    .min(18, "You must be at least 18 years old"),

  agreeTerms: z.boolean()
    .refine(val => val === true, "You must agree to terms")
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ["confirmPassword"] // Error sẽ hiện ở field confirmPassword
});

// 2. Component
function RegistrationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm({
    resolver: zodResolver(registrationSchema) // Tích hợp Zod
  });

  const onSubmit = async (data) => {
    try {
      // Call API
      const response = await axios.post('/api/register', data);
      console.log("Success:", response.data);
    } catch (error) {
      console.error("Error:", error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register("username")} placeholder="Username" />
        {errors.username && <span className="error">{errors.username.message}</span>}
      </div>

      <div>
        <input {...register("email")} type="email" placeholder="Email" />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>

      <div>
        <input {...register("password")} type="password" placeholder="Password" />
        {errors.password && <span className="error">{errors.password.message}</span>}
      </div>

      <div>
        <input {...register("confirmPassword")} type="password" placeholder="Confirm Password" />
        {errors.confirmPassword && <span className="error">{errors.confirmPassword.message}</span>}
      </div>

      <div>
        <input
          {...register("age", { valueAsNumber: true })}
          type="number"
          placeholder="Age"
        />
        {errors.age && <span className="error">{errors.age.message}</span>}
      </div>

      <div>
        <label>
          <input {...register("agreeTerms")} type="checkbox" />
          I agree to terms and conditions
        </label>
        {errors.agreeTerms && <span className="error">{errors.agreeTerms.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Submitting..." : "Register"}
      </button>
    </form>
  );
}
```

---

## 🔥 Ví dụ thực tế

### 1. Login Form với Error Handling

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import axios from 'axios';

const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "Password must be at least 6 characters")
});

function LoginForm() {
  const [serverError, setServerError] = useState("");
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
    resolver: zodResolver(loginSchema)
  });

  const onSubmit = async (data) => {
    try {
      setServerError("");
      const response = await axios.post('/api/auth/login', data);

      // Save token
      localStorage.setItem('token', response.data.token);

      // Redirect
      window.location.href = '/dashboard';
    } catch (error) {
      if (error.response?.status === 401) {
        setServerError("Invalid email or password");
      } else {
        setServerError("An error occurred. Please try again.");
      }
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="login-form">
      <h2>Login</h2>

      {serverError && (
        <div className="alert alert-error">{serverError}</div>
      )}

      <div className="form-group">
        <label>Email</label>
        <input
          {...register("email")}
          type="email"
          className={errors.email ? "error" : ""}
        />
        {errors.email && <span className="error-msg">{errors.email.message}</span>}
      </div>

      <div className="form-group">
        <label>Password</label>
        <input
          {...register("password")}
          type="password"
          className={errors.password ? "error" : ""}
        />
        {errors.password && <span className="error-msg">{errors.password.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Logging in..." : "Login"}
      </button>
    </form>
  );
}
```

### 2. Dynamic Form (Add/Remove Fields)

```jsx
import { useForm, useFieldArray } from 'react-hook-form';

function DynamicForm() {
  const { register, control, handleSubmit } = useForm({
    defaultValues: {
      contacts: [{ name: "", phone: "" }]
    }
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: "contacts"
  });

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <h3>Contacts</h3>

      {fields.map((field, index) => (
        <div key={field.id}>
          <input
            {...register(`contacts.${index}.name`)}
            placeholder="Name"
          />
          <input
            {...register(`contacts.${index}.phone`)}
            placeholder="Phone"
          />
          <button type="button" onClick={() => remove(index)}>
            Remove
          </button>
        </div>
      ))}

      <button
        type="button"
        onClick={() => append({ name: "", phone: "" })}
      >
        Add Contact
      </button>

      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## 📝 Bài tập thực hành

### Bài 1: Contact Form

Tạo contact form với:
- Fields: name, email, subject, message
- Validation: tất cả required, email hợp lệ
- Submit to console
- Loading state khi submit

### Bài 2: Multi-step Registration

Tạo form đăng ký 3 bước:
- Step 1: Personal Info (name, email, age)
- Step 2: Account (username, password, confirm password)
- Step 3: Preferences (newsletter checkbox, interests multi-select)
- Validate mỗi bước trước khi next
- Sử dụng React Hook Form + Zod

### Bài 3: Product Form

Tạo form thêm sản phẩm:
- Fields: name, price, category (select), description (textarea), inStock (checkbox)
- Validation với Zod
- Preview data trước khi submit
- Clear form sau khi submit thành công

### Bài 4: Search Filter Form

Tạo form search/filter products:
- Search input (real-time)
- Category filter (select)
- Price range (min, max)
- Sort by (dropdown)
- Submit để fetch từ API

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ Controlled vs Uncontrolled components
✅ Xử lý form với multiple inputs
✅ Validation thủ công và real-time
✅ React Hook Form để quản lý form hiệu quả
✅ Zod schema validation (giống Bean Validation trong Java)
✅ Kết hợp React Hook Form + Zod (best practice)

**Bài tiếp theo:** [07 - Context API và Zustand](07-context-api-va-zustand.md)

---

> 💬 **Tips**: Sử dụng **React Hook Form + Zod** cho mọi form phức tạp. Điều này giúp code ngắn gọn, hiệu năng cao, và dễ maintain hơn validation thủ công!
