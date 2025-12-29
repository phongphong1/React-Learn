# 02 - Setup Môi Trường với Vite

## 📚 Mục Lục

1. [Giới thiệu về Vite](#giới-thiệu-về-vite)
2. [Cài đặt Node.js và npm](#cài-đặt-nodejs-và-npm)
3. [Tạo project React với Vite](#tạo-project-react-với-vite)
4. [Cấu trúc thư mục chuẩn](#cấu-trúc-thư-mục-chuẩn)
5. [Cấu hình project](#cấu-hình-project)
6. [Chạy và build project](#chạy-và-build-project)
7. [Bài tập thực hành](#bài-tập-thực-hành)

---

## ⚡ Giới thiệu về Vite

**Vite** (phát âm: /vit/ - tiếng Pháp nghĩa là "nhanh") là một build tool hiện đại cho frontend, được tạo bởi Evan You (tác giả của Vue.js).

### Tại sao dùng Vite thay vì Create React App?

| **Create React App (CRA)** | **Vite** |
|----------------------------|----------|
| Khởi động chậm (~30-60s) | Khởi động cực nhanh (~1-2s) |
| Build chậm | Build nhanh hơn 10-100 lần |
| Webpack (cũ) | ESBuild + Rollup (hiện đại) |
| Không còn được maintain | Đang phát triển mạnh |

> 💡 **So sánh với Java**: Vite như **Spring Boot với embedded Tomcat**, trong khi CRA như **deploy WAR file lên Tomcat riêng** - cồng kềnh và chậm hơn.

---

## 📦 Cài đặt Node.js và npm

### Kiểm tra phiên bản hiện tại

```bash
# Kiểm tra Node.js
node --version
# Cần: v18.0.0 trở lên (khuyến nghị v20+)

# Kiểm tra npm
npm --version
# Cần: v9.0.0 trở lên
```

### Cài đặt Node.js (nếu chưa có)

**Linux/Ubuntu:**
```bash
# Sử dụng nvm (Node Version Manager) - khuyến nghị
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Cài Node.js phiên bản LTS
nvm install --lts
nvm use --lts
```

**macOS:**
```bash
# Sử dụng Homebrew
brew install node

# Hoặc dùng nvm
brew install nvm
nvm install --lts
```

**Windows:**
- Download từ [nodejs.org](https://nodejs.org/)
- Chọn phiên bản LTS
- Chạy installer

---

## 🚀 Tạo project React với Vite

### Bước 1: Tạo project mới

```bash
# Cú pháp: npm create vite@latest <tên-project> -- --template react
npm create vite@latest my-react-app -- --template react

# Hoặc với TypeScript (khuyến nghị cho dự án lớn)
npm create vite@latest my-react-app -- --template react-ts
```

### Bước 2: Di chuyển vào thư mục project

```bash
cd my-react-app
```

### Bước 3: Cài đặt dependencies

```bash
# Giống như chạy mvn install trong Maven
npm install

# Hoặc dùng pnpm (nhanh hơn npm)
# npm install -g pnpm
# pnpm install
```

### Bước 4: Chạy development server

```bash
npm run dev
```

Bạn sẽ thấy output:

```
  VITE v5.0.0  ready in 500 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

> 💡 **So sánh với Spring Boot**: `npm run dev` giống như chạy `mvn spring-boot:run` - khởi động dev server để test ứng dụng.

---

## 📁 Cấu trúc thư mục chuẩn

### Cấu trúc mặc định của Vite

```
my-react-app/
├── node_modules/          # Dependencies (giống thư mục .m2/repository trong Maven)
├── public/                # Static assets (không qua bundler)
│   └── vite.svg
├── src/                   # Source code chính
│   ├── assets/           # Images, fonts, styles (qua bundler)
│   │   └── react.svg
│   ├── App.css           # Styles cho App component
│   ├── App.jsx           # Component chính
│   ├── index.css         # Global styles
│   └── main.jsx          # Entry point (giống main() trong Java)
├── .gitignore
├── index.html            # HTML template
├── package.json          # Giống pom.xml trong Maven
├── vite.config.js        # Cấu hình Vite
└── README.md
```

### Cấu trúc thư mục chuẩn cho dự án thực tế

Hãy tổ chức lại như sau:

```
my-react-app/
├── public/
│   ├── images/           # Hình ảnh static
│   └── favicon.ico
├── src/
│   ├── assets/           # Assets cần xử lý (images, fonts)
│   │   ├── images/
│   │   ├── fonts/
│   │   └── styles/
│   │       └── global.css
│   ├── components/       # Reusable components
│   │   ├── common/      # Button, Input, Card...
│   │   ├── layout/      # Header, Footer, Sidebar...
│   │   └── features/    # Feature-specific components
│   ├── pages/            # Page components (giống Controller trong Spring)
│   │   ├── Home.jsx
│   │   ├── About.jsx
│   │   └── Contact.jsx
│   ├── hooks/            # Custom React Hooks
│   │   └── useAuth.js
│   ├── services/         # API services (giống Service layer trong Spring)
│   │   ├── api.js       # Axios config
│   │   └── userService.js
│   ├── store/            # State management (Context/Zustand)
│   │   └── authStore.js
│   ├── utils/            # Utility functions
│   │   ├── constants.js
│   │   └── helpers.js
│   ├── App.jsx           # Root component
│   └── main.jsx          # Entry point
├── .env                  # Environment variables
├── .env.example
├── package.json
└── vite.config.js
```

### So sánh với cấu trúc Spring Boot

| **Spring Boot** | **React (Vite)** |
|----------------|------------------|
| `src/main/java/` | `src/` |
| `controller/` | `pages/` |
| `service/` | `services/` |
| `model/entity/` | `types/` hoặc `models/` |
| `config/` | `config/` hoặc trong `vite.config.js` |
| `utils/` | `utils/` |
| `resources/static/` | `public/` |
| `application.properties` | `.env` |
| `pom.xml` | `package.json` |

---

## ⚙️ Cấu hình project

### 1. File `package.json`

```json
{
  "name": "my-react-app",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",                    // Chạy dev server
    "build": "vite build",            // Build production
    "preview": "vite preview",        // Preview production build
    "lint": "eslint ."                // Kiểm tra code
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0",
    "eslint": "^8.55.0"
  }
}
```

> 💡 **So sánh với Maven**: `dependencies` giống `<dependencies>`, `scripts` giống `<build><plugins>`

### 2. File `vite.config.js`

```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],

  // Cấu hình alias (giống import trong Java)
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@pages': path.resolve(__dirname, './src/pages'),
      '@services': path.resolve(__dirname, './src/services'),
      '@utils': path.resolve(__dirname, './src/utils'),
    }
  },

  // Cấu hình server dev
  server: {
    port: 3000,              // Đổi port (mặc định 5173)
    open: true,              // Tự động mở browser
    proxy: {
      '/api': {
        target: 'http://localhost:8080',  // Backend Spring Boot
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },

  // Cấu hình build
  build: {
    outDir: 'dist',          // Thư mục output
    sourcemap: true,         // Tạo source map cho debug
    minify: 'esbuild',       // Minify code
  }
});
```

### 3. File `.env` - Environment Variables

```bash
# .env.development
VITE_API_URL=http://localhost:8080
VITE_APP_NAME=My React App
VITE_ENABLE_MOCK=true

# .env.production
VITE_API_URL=https://api.production.com
VITE_APP_NAME=My React App
VITE_ENABLE_MOCK=false
```

**Sử dụng trong code:**

```jsx
const apiUrl = import.meta.env.VITE_API_URL;
const appName = import.meta.env.VITE_APP_NAME;

console.log('API URL:', apiUrl);
```

> ⚠️ **Lưu ý**: Biến môi trường trong Vite phải bắt đầu với `VITE_` mới được expose ra client.

---

## 🏃 Chạy và build project

### Development Mode

```bash
# Chạy dev server (hot reload)
npm run dev

# Hoặc với custom port
npm run dev -- --port 3000

# Mở trên network (để test trên mobile)
npm run dev -- --host
```

### Production Build

```bash
# Build project
npm run build

# Build sẽ tạo thư mục dist/
# ├── assets/
# │   ├── index-abc123.js   # Đã minify
# │   └── index-def456.css
# └── index.html
```

### Preview Production Build

```bash
# Xem trước bản build
npm run preview
```

### So sánh với Spring Boot

| **Spring Boot** | **React (Vite)** |
|----------------|------------------|
| `mvn spring-boot:run` | `npm run dev` |
| `mvn clean package` | `npm run build` |
| `java -jar app.jar` | `npm run preview` |

---

## 📚 Cài đặt thư viện phổ biến

### 1. React Router (Routing)

```bash
npm install react-router-dom
```

### 2. Axios (HTTP Client)

```bash
npm install axios
```

### 3. Zustand (State Management)

```bash
npm install zustand
```

### 4. React Hook Form + Zod (Form Validation)

```bash
npm install react-hook-form zod @hookform/resolvers
```

### 5. TailwindCSS (CSS Framework)

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 6. React Query (Data Fetching)

```bash
npm install @tanstack/react-query
```

---

## 🔥 Ví dụ thực tế: Setup project hoàn chỉnh

```bash
# 1. Tạo project
npm create vite@latest fithub-frontend -- --template react
cd fithub-frontend

# 2. Cài dependencies
npm install

# 3. Cài thêm thư viện cần thiết
npm install react-router-dom axios zustand
npm install react-hook-form zod @hookform/resolvers

# 4. Tạo cấu trúc thư mục
mkdir -p src/{components/{common,layout,features},pages,hooks,services,store,utils}

# 5. Tạo các file cấu hình
touch .env.development .env.production

# 6. Chạy project
npm run dev
```

### File `src/main.jsx` - Entry Point

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './index.css';

// Render app vào DOM (giống main() trong Java)
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### File `src/App.jsx` - Root Component

```jsx
import { useState } from 'react';
import './App.css';

function App() {
  const [count, setCount] = useState(0);

  return (
    <div className="App">
      <h1>Welcome to FitHub</h1>
      <div className="card">
        <button onClick={() => setCount(count + 1)}>
          Count is {count}
        </button>
      </div>
      <p className="info">
        Edit src/App.jsx and save to test HMR (Hot Module Replacement)
      </p>
    </div>
  );
}

export default App;
```

---

## 📝 Bài tập thực hành

### Bài 1: Tạo project đầu tiên

1. Tạo một React project với Vite tên `my-first-app`
2. Cài đặt `react-router-dom` và `axios`
3. Tạo cấu trúc thư mục theo chuẩn đã học
4. Chạy project và truy cập `http://localhost:5173`

### Bài 2: Cấu hình Vite

Thêm vào `vite.config.js`:
- Alias `@` trỏ đến `src/`
- Port 3000
- Proxy `/api` về `http://localhost:8080`

### Bài 3: Environment Variables

1. Tạo file `.env.development`:
```bash
VITE_API_URL=http://localhost:8080
VITE_APP_NAME=My First App
```

2. Tạo component hiển thị `VITE_APP_NAME` trong header

### Bài 4: Build và Deploy

1. Chạy `npm run build`
2. Kiểm tra thư mục `dist/`
3. Chạy `npm run preview` để xem kết quả

---

## 🎓 Tổng kết

Sau bài học này, bạn đã biết:

✅ Cài đặt và cấu hình Vite cho React
✅ Tạo cấu trúc thư mục chuẩn cho dự án thực tế
✅ Cấu hình proxy để kết nối với backend Spring Boot
✅ Sử dụng environment variables
✅ Build và deploy ứng dụng React

**Bài tiếp theo:** [03 - JSX và Props](03-jsx-va-props.md)

---

> 💬 **Tips**: Luôn tạo cấu trúc thư mục rõ ràng ngay từ đầu, giống như bạn tổ chức package trong Java. Điều này sẽ giúp project dễ maintain khi scale lên!
