# 05 - useEffect và Lifecycle

## 📚 Mục Lục

1. [Side Effects là gì?](#side-effects-là-gì)
2. [useEffect Hook](#useeffect-hook)
3. [Component Lifecycle](#component-lifecycle)
4. [Dependency Array](#dependency-array)
5. [Cleanup Function](#cleanup-function)
6. [useEffect vs @PostConstruct (Spring)](#useeffect-vs-postconstruct)
7. [Call API với useEffect](#call-api-với-useeffect)
8. [Common Patterns](#common-patterns)
9. [Ví dụ thực tế](#ví-dụ-thực-tế)
10. [Bài tập thực hành](#bài-tập-thực-hành)

---

## 🎯 Side Effects là gì?

**Side Effect** là các thao tác **bên ngoài scope của component**, không liên quan trực tiếp đến việc render UI.

### Các loại Side Effects phổ biến

| **Side Effect** | **Ví dụ** |
|----------------|-----------|
| **Data Fetching** | Call API từ backend |
| **Subscriptions** | WebSocket, EventListener |
| **DOM Manipulation** | Thay đổi title, focus input |
| **Timers** | setTimeout, setInterval |
| **Logging** | Console.log, analytics |
| **Local Storage** | Lưu/đọc data từ localStorage |

> 💡 **So sánh với Java**: Side effects giống như các thao tác I/O, database query, external API call - không phải business logic thuần túy.

```java
// Java Service Layer
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @PostConstruct  // Chạy sau khi bean được khởi tạo
    public void init() {
        // Side effect: Load config từ database
        loadConfiguration();
        // Side effect: Connect to external service
        connectToExternalAPI();
    }

    public User getUser(Long id) {
        // Side effect: Database query
        return userRepository.findById(id);
    }
}
```

---

## 🪝 useEffect Hook

`useEffect` cho phép thực hiện side effects trong functional component.

### Cú pháp cơ bản

```jsx
import { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // Code chạy sau mỗi lần render
    console.log("Component rendered");
  });

  return <div>Hello</div>;
}
```

### Cú pháp đầy đủ

```jsx
useEffect(() => {
  // 1. Setup logic (side effect)
  console.log("Effect running");

  // 2. Cleanup function (optional)
  return () => {
    console.log("Cleanup");
  };
}, [dependencies]); // 3. Dependency array (optional)
```

### Ba dạng useEffect

```jsx
function ExampleComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");

  // 1️⃣ Không có dependency array → Chạy sau MỖI render
  useEffect(() => {
    console.log("Chạy sau mỗi render");
  });

  // 2️⃣ Empty dependency array [] → Chỉ chạy 1 lần (mount)
  useEffect(() => {
    console.log("Chạy 1 lần khi component mount");
  }, []);

  // 3️⃣ Có dependencies → Chạy khi dependencies thay đổi
  useEffect(() => {
    console.log("Chạy khi count thay đổi");
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <input value={name} onChange={(e) => setName(e.target.value)} />
    </div>
  );
}
```

---

## 🔄 Component Lifecycle

### Class Component Lifecycle (Cũ)

```jsx
// Class Component (cách cũ)
class MyComponent extends React.Component {
  componentDidMount() {
    // Chạy sau khi component mount
  }

  componentDidUpdate(prevProps, prevState) {
    // Chạy sau mỗi lần update
  }

  componentWillUnmount() {
    // Chạy trước khi component unmount
  }

  render() {
    return <div>Hello</div>;
  }
}
```

### Functional Component với useEffect (Hiện đại)

```jsx
function MyComponent() {
  // ✅ componentDidMount
  useEffect(() => {
    console.log("Component mounted");
  }, []);

  // ✅ componentDidUpdate (khi count thay đổi)
  useEffect(() => {
    console.log("Count updated");
  }, [count]);

  // ✅ componentWillUnmount
  useEffect(() => {
    return () => {
      console.log("Component will unmount");
    };
  }, []);

  return <div>Hello</div>;
}
```

### So sánh với Spring Bean Lifecycle

| **Spring Boot** | **React** |
|----------------|-----------|
| `@PostConstruct` | `useEffect(() => {}, [])` |
| Bean initialization | Component mount |
| `@PreDestroy` | cleanup function `return () => {}` |
| Bean destruction | Component unmount |

---

## 📦 Dependency Array

Dependency array quyết định **khi nào effect chạy lại**.

### 1. Không có dependency array

```jsx
useEffect(() => {
  console.log("Chạy sau MỖI lần render");
  // ⚠️ Hiếm khi dùng, có thể gây performance issue
});
```

### 2. Empty dependency array `[]`

```jsx
useEffect(() => {
  console.log("Chỉ chạy 1 lần khi mount");

  // Use cases:
  // - Fetch data lần đầu
  // - Setup event listeners
  // - Connect to external service
}, []);
```

### 3. Với dependencies

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    console.log("Chạy khi userId thay đổi");

    // Fetch user data khi userId thay đổi
    fetchUser(userId).then(data => setUser(data));
  }, [userId]); // Chỉ chạy khi userId thay đổi

  return <div>{user?.name}</div>;
}
```

### ⚠️ Lỗi thường gặp: Missing dependencies

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  // ❌ SAI: count trong dependencies bị thiếu
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // Luôn log 0, không update
    }, 1000);

    return () => clearInterval(timer);
  }, []); // ESLint warning!

  // ✅ ĐÚNG: Thêm count vào dependencies
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // Log giá trị mới nhất
    }, 1000);

    return () => clearInterval(timer);
  }, [count]);

  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

---

## 🧹 Cleanup Function

Cleanup function dùng để **dọn dẹp resources** trước khi:
1. Effect chạy lại
2. Component unmount

### Khi nào cần Cleanup?

| **Side Effect** | **Cần Cleanup?** | **Lý do** |
|----------------|-----------------|----------|
| setTimeout | ✅ Có | Tránh memory leak |
| setInterval | ✅ Có | Tránh chạy khi component unmount |
| Event Listener | ✅ Có | Tránh multiple listeners |
| WebSocket | ✅ Có | Đóng connection |
| Subscription | ✅ Có | Unsubscribe |
| Fetch API | ⚠️ Không bắt buộc | Nhưng nên cancel request |

### Ví dụ Cleanup

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    console.log("Setup timer");

    // Setup: Tạo interval
    const intervalId = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // Cleanup: Xóa interval
    return () => {
      console.log("Cleanup timer");
      clearInterval(intervalId);
    };
  }, []); // Chỉ chạy 1 lần

  return <div>Seconds: {seconds}</div>;
}
```

### Event Listener với Cleanup

```jsx
function WindowSize() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    // Setup: Add event listener
    const handleResize = () => {
      setWidth(window.innerWidth);
    };

    window.addEventListener('resize', handleResize);

    // Cleanup: Remove event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <div>Window width: {width}px</div>;
}
```

---

## 🔄 useEffect vs @PostConstruct (Spring)

### Spring Boot Example

```java
@Component
public class DataService {

    private List<User> users;

    @PostConstruct  // Chạy sau khi bean được inject
    public void init() {
        // Load data từ database
        users = userRepository.findAll();

        // Connect to external service
        externalAPI.connect();
    }

    @PreDestroy  // Chạy trước khi bean bị destroy
    public void cleanup() {
        // Close connections
        externalAPI.disconnect();
    }
}
```

### React Equivalent

```jsx
function DataComponent() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    // Tương đương @PostConstruct
    console.log("Component mounted - Load data");

    // Load data
    fetchUsers().then(data => setUsers(data));

    // Connect to service
    const connection = connectToService();

    // Tương đương @PreDestroy
    return () => {
      console.log("Component unmounting - Cleanup");
      connection.disconnect();
    };
  }, []); // Empty array = chỉ chạy khi mount/unmount

  return <div>{users.length} users loaded</div>;
}
```

---

## 🌐 Call API với useEffect

### Pattern cơ bản

```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Async function bên trong useEffect
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await axios.get('/api/users');
        setUsers(response.data);
        setError(null);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []); // Chỉ call 1 lần khi mount

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Fetch khi props/state thay đổi

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let isCancelled = false; // Flag để tránh update sau khi unmount

    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await axios.get(`/api/users/${userId}`);

        // Chỉ update state nếu component còn mounted
        if (!isCancelled) {
          setUser(response.data);
        }
      } catch (error) {
        if (!isCancelled) {
          console.error(error);
        }
      } finally {
        if (!isCancelled) {
          setLoading(false);
        }
      }
    };

    fetchUser();

    // Cleanup: Đánh dấu request đã cancelled
    return () => {
      isCancelled = true;
    };
  }, [userId]); // Re-fetch khi userId thay đổi

  if (loading) return <div>Loading user...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### AbortController (Modern way)

```jsx
function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    const fetchUsers = async () => {
      try {
        const response = await axios.get('/api/users', {
          signal: controller.signal  // Pass abort signal
        });
        setUsers(response.data);
      } catch (error) {
        if (error.name === 'CanceledError') {
          console.log('Request cancelled');
        } else {
          console.error(error);
        }
      }
    };

    fetchUsers();

    // Cleanup: Cancel request khi unmount
    return () => {
      controller.abort();
    };
  }, []);

  return <div>...</div>;
}
```

---

## 🎨 Common Patterns

### 1. Document Title

```jsx
function PageTitle({ title }) {
  useEffect(() => {
    document.title = title;
  }, [title]);

  return <h1>{title}</h1>;
}

// Sử dụng
<PageTitle title="Home Page" />
```

### 2. LocalStorage Sync

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// Sử dụng
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current theme: {theme}
    </button>
  );
}
```

### 3. Debounced Search

```jsx
function SearchBox() {
  const [searchTerm, setSearchTerm] = useState("");
  const [results, setResults] = useState([]);

  useEffect(() => {
    // Debounce: Chỉ search sau 500ms user ngừng gõ
    const timeoutId = setTimeout(() => {
      if (searchTerm) {
        searchAPI(searchTerm).then(data => setResults(data));
      }
    }, 500);

    // Cleanup: Clear timeout nếu user tiếp tục gõ
    return () => clearTimeout(timeoutId);
  }, [searchTerm]);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

---

## 🔥 Ví dụ thực tế

### 1. Real-time Clock

```jsx
function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => clearInterval(intervalId);
  }, []);

  return (
    <div>
      <h1>{time.toLocaleTimeString()}</h1>
    </div>
  );
}
```

### 2. Fetch Products with Pagination

```jsx
function ProductList() {
  const [products, setProducts] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  useEffect(() => {
    const fetchProducts = async () => {
      setLoading(true);
      try {
        const response = await axios.get(`/api/products?page=${page}&limit=10`);

        if (response.data.length === 0) {
          setHasMore(false);
        } else {
          setProducts(prev => [...prev, ...response.data]);
        }
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };

    fetchProducts();
  }, [page]);

  return (
    <div>
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>

      {loading && <div>Loading...</div>}

      {hasMore && !loading && (
        <button onClick={() => setPage(prev => prev + 1)}>
          Load More
        </button>
      )}
    </div>
  );
}
```

### 3. Auto-save Form

```jsx
function AutoSaveForm() {
  const [formData, setFormData] = useState({ title: "", content: "" });
  const [saveStatus, setSaveStatus] = useState("saved");

  useEffect(() => {
    setSaveStatus("saving...");

    const timeoutId = setTimeout(() => {
      // Save to backend
      axios.post('/api/drafts', formData)
        .then(() => setSaveStatus("saved"))
        .catch(() => setSaveStatus("error"));
    }, 2000); // Auto-save sau 2s không có thay đổi

    return () => clearTimeout(timeoutId);
  }, [formData]);

  return (
    <div>
      <p>Status: {saveStatus}</p>

      <input
        value={formData.title}
        onChange={(e) => setFormData({ ...formData, title: e.target.value })}
        placeholder="Title"
      />

      <textarea
        value={formData.content}
        onChange={(e) => setFormData({ ...formData, content: e.target.value })}
        placeholder="Content"
      />
    </div>
  );
}
```

---

## 📝 Bài tập thực hành

### Bài 1: Countdown Timer

Tạo component CountdownTimer:
- Props: `seconds` (số giây đếm ngược)
- State: `timeLeft`
- useEffect: Giảm `timeLeft` mỗi giây
- Dừng khi `timeLeft = 0`
- Cleanup khi component unmount

### Bài 2: Weather App

Tạo Weather component:
- Fetch data từ API: `https://api.openweathermap.org/`
- State: `weather`, `loading`, `error`
- Input city name, fetch khi user nhập xong (debounce 500ms)
- Hiển thị: nhiệt độ, mô tả thời tiết

### Bài 3: Infinite Scroll

Tạo InfiniteScroll component:
- Fetch posts từ API (pagination)
- useEffect: Detect khi user scroll đến cuối trang
- Tự động load thêm posts
- Cleanup event listener

### Bài 4: Chat Application

Tạo ChatRoom component:
- useEffect: Connect to WebSocket khi mount
- Listen messages từ server
- Cleanup: Disconnect khi unmount
- State: `messages`, `isConnected`

---

## 🎓 Tổng kết

Sau bài học này, bạn đã hiểu:

✅ Side effects và tại sao cần useEffect
✅ useEffect Hook với dependency array
✅ Component lifecycle: mount, update, unmount
✅ Cleanup function để dọn dẹp resources
✅ So sánh với @PostConstruct/@PreDestroy trong Spring
✅ Call API với useEffect
✅ Common patterns: timer, event listener, auto-save

**Bài tiếp theo:** [06 - Form và Validation](06-form-va-validation.md)

---

> 💬 **Tips**: Luôn thêm **cleanup function** khi dùng timer, event listener, hoặc subscription để tránh memory leak!
