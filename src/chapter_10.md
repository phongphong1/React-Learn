# 10 - Project Thực Hành: FitHub Mini

## 📚 Mục Lục

1. [Giới thiệu Project](#giới-thiệu-project)
2. [Yêu cầu chức năng](#yêu-cầu-chức-năng)
3. [Tech Stack](#tech-stack)
4. [Thiết kế Database](#thiết-kế-database)
5. [Backend API (Spring Boot)](#backend-api-spring-boot)
6. [Frontend Structure](#frontend-structure)
7. [Implementation Guide](#implementation-guide)
8. [Các tính năng nâng cao](#các-tính-năng-nâng-cao)
9. [Deployment](#deployment)
10. [Next Steps](#next-steps)

---

## 🎯 Giới thiệu Project

**FitHub Mini** là một ứng dụng quản lý khóa học gym đơn giản, kết hợp tất cả kiến thức đã học từ Bài 1 đến Bài 9.

### Mục tiêu

✅ Áp dụng kiến thức React từ cơ bản đến nâng cao
✅ Kết nối với Backend Spring Boot
✅ Xây dựng ứng dụng fullstack hoàn chỉnh
✅ Thực hành best practices

### Demo Features

```
┌─────────────────────────────────────┐
│           FitHub Mini               │
├─────────────────────────────────────┤
│  👤 User Authentication             │
│  📚 Browse Gym Courses              │
│  🛒 Enroll in Courses               │
│  👨‍💼 Instructor Dashboard           │
│  📊 Course Management (CRUD)        │
│  💬 Reviews & Ratings               │
└─────────────────────────────────────┘
```

---

## 📋 Yêu cầu chức năng

### 1. User Features (Member)

- **Authentication**
  - Đăng ký tài khoản
  - Đăng nhập/Đăng xuất
  - JWT token authentication

- **Course Browsing**
  - Xem danh sách khóa học
  - Filter theo category (Yoga, CrossFit, Boxing, etc.)
  - Search theo tên
  - View chi tiết khóa học

- **Enrollment**
  - Đăng ký khóa học
  - Xem khóa học đã đăng ký
  - Hủy đăng ký

- **Reviews**
  - Đánh giá khóa học (1-5 sao)
  - Viết review
  - Xem reviews của người khác

### 2. Instructor Features

- **Course Management**
  - Tạo khóa học mới
  - Sửa khóa học
  - Xóa khóa học
  - Upload ảnh khóa học

- **Dashboard**
  - Xem số lượng học viên
  - Thống kê khóa học
  - Xem reviews

### 3. Admin Features (Optional)

- Quản lý users
- Quản lý tất cả courses
- Approve/Reject courses

---

## 🛠️ Tech Stack

### Frontend
- ⚛️ React 18
- ⚡ Vite
- 🎨 TailwindCSS / CSS Modules
- 🔀 React Router DOM v6
- 📡 Axios
- 🐻 Zustand (State Management)
- 📝 React Hook Form + Zod
- 🎭 React Icons

### Backend
- ☕ Spring Boot 3.x
- 🔐 Spring Security + JWT
- 🗄️ MySQL
- 📦 JPA/Hibernate
- ✅ Bean Validation

### Tools
- Git & GitHub
- Postman (API Testing)
- VS Code

---

## 💾 Thiết kế Database

### Entity Relationship Diagram

```
┌─────────────┐       ┌──────────────┐       ┌─────────────┐
│    User     │       │   Course     │       │ Enrollment  │
├─────────────┤       ├──────────────┤       ├─────────────┤
│ id (PK)     │       │ id (PK)      │       │ id (PK)     │
│ username    │       │ name         │       │ userId (FK) │
│ email       │       │ description  │       │ courseId FK │
│ password    │───┐   │ category     │   ┌───│ enrollDate  │
│ role        │   │   │ price        │   │   │ status      │
│ createdAt   │   │   │ duration     │   │   └─────────────┘
└─────────────┘   │   │ instructorId │   │
                  │   │ imageUrl     │   │   ┌─────────────┐
                  │   │ rating       │   │   │   Review    │
                  └──►│ createdAt    │◄──┤   ├─────────────┤
                      └──────────────┘   │   │ id (PK)     │
                                         └───│ userId (FK) │
                                             │ courseId FK │
                                             │ rating      │
                                             │ comment     │
                                             │ createdAt   │
                                             └─────────────┘
```

### SQL Schema

```sql
-- Users Table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('MEMBER', 'INSTRUCTOR', 'ADMIN') DEFAULT 'MEMBER',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Courses Table
CREATE TABLE courses (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(50) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    duration INT NOT NULL, -- in minutes
    instructor_id BIGINT NOT NULL,
    image_url VARCHAR(255),
    rating DECIMAL(2,1) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (instructor_id) REFERENCES users(id)
);

-- Enrollments Table
CREATE TABLE enrollments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    enroll_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('ACTIVE', 'COMPLETED', 'CANCELLED') DEFAULT 'ACTIVE',
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (course_id) REFERENCES courses(id),
    UNIQUE KEY (user_id, course_id)
);

-- Reviews Table
CREATE TABLE reviews (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (course_id) REFERENCES courses(id),
    UNIQUE KEY (user_id, course_id)
);
```

---

## 🔙 Backend API (Spring Boot)

### Project Structure

```
fithub-backend/
├── src/main/java/com/fithub/
│   ├── config/
│   │   ├── SecurityConfig.java
│   │   ├── CorsConfig.java
│   │   └── JwtConfig.java
│   ├── controller/
│   │   ├── AuthController.java
│   │   ├── CourseController.java
│   │   ├── EnrollmentController.java
│   │   └── ReviewController.java
│   ├── dto/
│   │   ├── LoginRequest.java
│   │   ├── RegisterRequest.java
│   │   ├── CourseDTO.java
│   │   └── ReviewDTO.java
│   ├── entity/
│   │   ├── User.java
│   │   ├── Course.java
│   │   ├── Enrollment.java
│   │   └── Review.java
│   ├── repository/
│   │   ├── UserRepository.java
│   │   ├── CourseRepository.java
│   │   ├── EnrollmentRepository.java
│   │   └── ReviewRepository.java
│   ├── service/
│   │   ├── AuthService.java
│   │   ├── CourseService.java
│   │   ├── EnrollmentService.java
│   │   └── ReviewService.java
│   └── security/
│       ├── JwtTokenProvider.java
│       └── JwtAuthenticationFilter.java
└── src/main/resources/
    └── application.properties
```

### Key API Endpoints

```java
// Auth
POST   /api/auth/register
POST   /api/auth/login
GET    /api/auth/me

// Courses
GET    /api/courses                    // Get all courses (public)
GET    /api/courses/{id}               // Get course by ID
POST   /api/courses                    // Create course (instructor only)
PUT    /api/courses/{id}               // Update course (owner only)
DELETE /api/courses/{id}               // Delete course (owner only)
GET    /api/courses/category/{category} // Filter by category
GET    /api/courses/search?q=query     // Search courses

// Enrollments
POST   /api/enrollments                // Enroll in course
GET    /api/enrollments/my-courses     // Get user's enrolled courses
DELETE /api/enrollments/{id}           // Cancel enrollment

// Reviews
POST   /api/reviews                    // Create review
GET    /api/reviews/course/{courseId}  // Get reviews for course
PUT    /api/reviews/{id}               // Update review
DELETE /api/reviews/{id}               // Delete review
```

### Example: Course Controller

```java
@RestController
@RequestMapping("/api/courses")
public class CourseController {

    @Autowired
    private CourseService courseService;

    @GetMapping
    public ResponseEntity<List<CourseDTO>> getAllCourses(
        @RequestParam(required = false) String category
    ) {
        List<CourseDTO> courses = category != null
            ? courseService.getCoursesByCategory(category)
            : courseService.getAllCourses();
        return ResponseEntity.ok(courses);
    }

    @GetMapping("/{id}")
    public ResponseEntity<CourseDTO> getCourseById(@PathVariable Long id) {
        CourseDTO course = courseService.getCourseById(id);
        return ResponseEntity.ok(course);
    }

    @PostMapping
    @PreAuthorize("hasRole('INSTRUCTOR')")
    public ResponseEntity<CourseDTO> createCourse(@Valid @RequestBody CourseDTO courseDTO) {
        CourseDTO created = courseService.createCourse(courseDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('INSTRUCTOR')")
    public ResponseEntity<CourseDTO> updateCourse(
        @PathVariable Long id,
        @Valid @RequestBody CourseDTO courseDTO
    ) {
        CourseDTO updated = courseService.updateCourse(id, courseDTO);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('INSTRUCTOR')")
    public ResponseEntity<Void> deleteCourse(@PathVariable Long id) {
        courseService.deleteCourse(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## ⚛️ Frontend Structure

### Project Structure

```
fithub-frontend/
├── public/
│   └── images/
├── src/
│   ├── assets/
│   │   ├── images/
│   │   └── styles/
│   │       └── global.css
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.jsx
│   │   │   ├── Input.jsx
│   │   │   ├── Card.jsx
│   │   │   ├── Loading.jsx
│   │   │   └── ErrorMessage.jsx
│   │   ├── layout/
│   │   │   ├── Header.jsx
│   │   │   ├── Footer.jsx
│   │   │   └── Sidebar.jsx
│   │   └── course/
│   │       ├── CourseCard.jsx
│   │       ├── CourseDetail.jsx
│   │       ├── CourseForm.jsx
│   │       └── CourseFilter.jsx
│   ├── contexts/
│   │   └── AuthContext.jsx
│   ├── hooks/
│   │   ├── useAuth.js
│   │   └── useCourses.js
│   ├── pages/
│   │   ├── HomePage.jsx
│   │   ├── CoursesPage.jsx
│   │   ├── CourseDetailPage.jsx
│   │   ├── MyCoursesPage.jsx
│   │   ├── InstructorDashboard.jsx
│   │   ├── LoginPage.jsx
│   │   └── RegisterPage.jsx
│   ├── services/
│   │   ├── api.js
│   │   ├── authService.js
│   │   ├── courseService.js
│   │   ├── enrollmentService.js
│   │   └── reviewService.js
│   ├── store/
│   │   └── useAuthStore.js
│   ├── utils/
│   │   ├── constants.js
│   │   └── helpers.js
│   ├── App.jsx
│   └── main.jsx
├── .env.development
├── .env.production
├── package.json
└── vite.config.js
```

---

## 🚀 Implementation Guide

### Phase 1: Setup & Authentication (Week 1)

**Backend:**
1. Create Spring Boot project
2. Setup MySQL database
3. Implement User entity & repository
4. JWT authentication
5. AuthController (register, login)

**Frontend:**
1. Create Vite React project
2. Setup Axios instance
3. AuthContext implementation
4. Login & Register pages
5. Protected routes

**Checklist:**
- [ ] User có thể đăng ký
- [ ] User có thể đăng nhập
- [ ] JWT token được lưu trong localStorage
- [ ] Protected routes hoạt động
- [ ] Logout thành công

---

### Phase 2: Course Management (Week 2)

**Backend:**
1. Course entity & repository
2. CourseController CRUD operations
3. Category filter
4. Search functionality

**Frontend:**
1. CourseCard component
2. CoursesPage với filter
3. CourseDetailPage
4. InstructorDashboard
5. CourseForm (create/edit)

**Checklist:**
- [ ] Hiển thị danh sách courses
- [ ] Filter theo category
- [ ] Search courses
- [ ] Xem chi tiết course
- [ ] Instructor tạo course
- [ ] Instructor sửa/xóa course của mình

---

### Phase 3: Enrollment & Reviews (Week 3)

**Backend:**
1. Enrollment entity & repository
2. EnrollmentController
3. Review entity & repository
4. ReviewController
5. Update course rating

**Frontend:**
1. Enroll button trên CourseDetail
2. MyCoursesPage
3. Review form
4. Display reviews
5. Rating component

**Checklist:**
- [ ] User enroll vào course
- [ ] Xem danh sách courses đã enroll
- [ ] Cancel enrollment
- [ ] Viết review
- [ ] Xem reviews
- [ ] Rating hiển thị đúng

---

### Phase 4: UI/UX Enhancement (Week 4)

**Frontend:**
1. Loading states
2. Error handling
3. Toast notifications
4. Responsive design
5. Dark mode (optional)

**Checklist:**
- [ ] Loading spinner
- [ ] Error messages
- [ ] Success notifications
- [ ] Mobile responsive
- [ ] Smooth transitions

---

## 🎨 Example Components

### 1. CourseCard Component

```jsx
// components/course/CourseCard.jsx
import { Link } from 'react-router-dom';

function CourseCard({ course }) {
  return (
    <Link to={`/courses/${course.id}`} className="course-card">
      <img src={course.imageUrl} alt={course.name} />

      <div className="course-info">
        <span className="category">{course.category}</span>

        <h3>{course.name}</h3>

        <div className="rating">
          {'⭐'.repeat(Math.round(course.rating))}
          <span>({course.rating})</span>
        </div>

        <div className="footer">
          <span className="price">${course.price}</span>
          <span className="duration">{course.duration} min</span>
        </div>
      </div>
    </Link>
  );
}

export default CourseCard;
```

### 2. CoursesPage with Filter

```jsx
// pages/CoursesPage.jsx
import { useState, useEffect } from 'react';
import { courseService } from '../services/courseService';
import CourseCard from '../components/course/CourseCard';

function CoursesPage() {
  const [courses, setCourses] = useState([]);
  const [category, setCategory] = useState('all');
  const [searchQuery, setSearchQuery] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchCourses();
  }, [category]);

  const fetchCourses = async () => {
    try {
      setLoading(true);
      const data = category === 'all'
        ? await courseService.getAllCourses()
        : await courseService.getCoursesByCategory(category);
      setCourses(data);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };

  const filteredCourses = courses.filter(course =>
    course.name.toLowerCase().includes(searchQuery.toLowerCase())
  );

  return (
    <div className="courses-page">
      <h1>Gym Courses</h1>

      <div className="filters">
        <input
          type="search"
          placeholder="Search courses..."
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
        />

        <select value={category} onChange={(e) => setCategory(e.target.value)}>
          <option value="all">All Categories</option>
          <option value="Yoga">Yoga</option>
          <option value="CrossFit">CrossFit</option>
          <option value="Boxing">Boxing</option>
          <option value="Pilates">Pilates</option>
        </select>
      </div>

      {loading ? (
        <div>Loading...</div>
      ) : (
        <div className="course-grid">
          {filteredCourses.map(course => (
            <CourseCard key={course.id} course={course} />
          ))}
        </div>
      )}
    </div>
  );
}

export default CoursesPage;
```

### 3. Enroll Button

```jsx
// components/course/EnrollButton.jsx
import { useState } from 'react';
import { enrollmentService } from '../../services/enrollmentService';
import { useAuth } from '../../contexts/AuthContext';

function EnrollButton({ courseId, onEnrolled }) {
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);

  const handleEnroll = async () => {
    if (!user) {
      alert('Please login to enroll');
      return;
    }

    try {
      setLoading(true);
      await enrollmentService.enrollCourse(courseId);
      alert('Enrolled successfully!');
      onEnrolled?.();
    } catch (error) {
      alert('Enrollment failed: ' + error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <button
      onClick={handleEnroll}
      disabled={loading}
      className="btn-primary"
    >
      {loading ? 'Enrolling...' : 'Enroll Now'}
    </button>
  );
}

export default EnrollButton;
```

---

## 🚀 Các tính năng nâng cao

### 1. Image Upload

```jsx
// Frontend
const handleImageUpload = async (file) => {
  const formData = new FormData();
  formData.append('image', file);

  const response = await axios.post('/api/courses/upload', formData, {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  });

  return response.data.imageUrl;
};
```

```java
// Backend
@PostMapping("/upload")
public ResponseEntity<Map<String, String>> uploadImage(
    @RequestParam("image") MultipartFile file
) {
    String imageUrl = fileStorageService.store(file);
    return ResponseEntity.ok(Map.of("imageUrl", imageUrl));
}
```

### 2. Real-time Search

```jsx
import { useState, useEffect } from 'react';
import { useDebouncedValue } from '../hooks/useDebounce';

function SearchBar() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebouncedValue(query, 500);

  useEffect(() => {
    if (debouncedQuery) {
      searchCourses(debouncedQuery);
    }
  }, [debouncedQuery]);

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### 3. Pagination

```jsx
function CoursesPage() {
  const [page, setPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);

  const fetchCourses = async () => {
    const response = await courseService.getCourses(page, 12);
    setCourses(response.data);
    setTotalPages(response.totalPages);
  };

  return (
    <>
      {/* Course list */}

      <div className="pagination">
        <button
          disabled={page === 1}
          onClick={() => setPage(page - 1)}
        >
          Previous
        </button>

        <span>Page {page} of {totalPages}</span>

        <button
          disabled={page === totalPages}
          onClick={() => setPage(page + 1)}
        >
          Next
        </button>
      </div>
    </>
  );
}
```

---

## 🌐 Deployment

### Frontend (Vercel/Netlify)

```bash
# Build
npm run build

# Deploy to Vercel
npm i -g vercel
vercel --prod

# Or deploy to Netlify
npm i -g netlify-cli
netlify deploy --prod --dir=dist
```

### Backend (Railway/Render)

```bash
# Build JAR
mvn clean package

# Deploy to Railway
railway up

# Or deploy to Render
# Connect GitHub repo
```

---

## 🎓 Next Steps

### Tính năng mở rộng

1. **Payment Integration**
   - Stripe/PayPal payment gateway
   - Purchase courses
   - Payment history

2. **Video Lessons**
   - Upload video lessons
   - Video player
   - Track progress

3. **Live Chat**
   - WebSocket real-time chat
   - Chat with instructor
   - Group chat

4. **Analytics Dashboard**
   - Charts (Chart.js/Recharts)
   - Revenue statistics
   - User engagement

5. **Notifications**
   - Email notifications
   - Push notifications
   - In-app notifications

### Học thêm

- **React Query**: Data fetching & caching
- **TypeScript**: Type-safe development
- **Next.js**: SSR & SSG
- **Testing**: Jest, React Testing Library
- **CI/CD**: GitHub Actions, Jenkins

---

## 🎊 Tổng kết

Chúc mừng bạn đã hoàn thành khóa học **React Mastery**!

### Bạn đã học được:

✅ React fundamentals (Components, Props, State)
✅ Hooks (useState, useEffect, useContext, custom hooks)
✅ Form handling & validation
✅ State management (Context API, Zustand)
✅ Routing với React Router
✅ API integration với Backend Spring Boot
✅ JWT Authentication
✅ Build fullstack application

### Tiếp tục phát triển:

📚 Đọc React documentation
🛠️ Build thêm nhiều projects
👥 Tham gia community (Reddit, Discord)
📝 Viết blog chia sẻ kiến thức
💼 Tìm kiếm cơ hội thực tập/việc làm

---

## 📞 Liên hệ & Hỗ trợ

Nếu có bất kỳ thắc mắc nào trong quá trình học, hãy:
- Tìm kiếm trên Stack Overflow
- Đọc documentation chính thức
- Tham gia React community

**Good luck với career path của bạn!** 🚀

---

> 💬 **Final Tips**: Thực hành là chìa khóa thành công! Hãy code mỗi ngày, build nhiều projects, và đừng ngại mắc lỗi. Mỗi lỗi là một bài học quý giá!
