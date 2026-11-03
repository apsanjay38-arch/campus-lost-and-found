# 🏛️ Campus Lost & Found Management System (Assignment 2)

An integrated, enterprise-grade full-stack **MERN** (React.js + Node.js + Express.js + MongoDB) Web Application designed for university campuses. The system enables students, faculty, and administrative staff to report lost items, log found items, submit ownership claims with verification evidence, track claim approval workflows, and moderate campus posts.

---

## 🌟 Key Highlights & Upgrades (Assignment 1 → Assignment 2)

| Feature | Assignment 1 (Frontend Only) | Assignment 2 (Full-Stack MERN) |
| :--- | :--- | :--- |
| **Architecture** | Static HTML, CSS, Vanilla JS | **Component-Driven React.js (Vite) + Node.js/Express REST API + MongoDB** |
| **Data Persistence** | Browser `localStorage` / Mock JS objects | **MongoDB Database via Mongoose with Schema Validation & ObjectId Refs** |
| **Authentication** | Simulated localStorage flag | **Stateless JWT Authentication with Bcrypt Password Hashing** |
| **Authorization** | UI-only dropdown role toggle | **Role-Based Access Control (RBAC)**: `student`, `staff`, `admin` |
| **CRUD Operations** | Local array mutations | **Real HTTP REST APIs (GET, POST, PUT, DELETE) modifying MongoDB** |
| **Claim Workflow** | Static prompt / mock list | **Multi-step Claim Verification** with Admin review modal & automated item status sync |
| **Dashboard** | Hardcoded sample numbers | **Real-Time Aggregation** calculated dynamically from MongoDB collections |
| **Error Handling** | Basic alert boxes | **Centralized Express Error Middleware** + Custom Toast Notifications & Loading Spinners |
| **Search & Filters** | Client-side array filter | **Server-side Multi-Field Regex Querying** across name, category, location, and status |
| **UI / UX Design** | Custom CSS System | **Preserved Premium SaaS Aesthetics** with Plus Jakarta Sans typography and responsive drawer |

---

## 🚀 Quick Start & Installation Guide

### Prerequisites
- **Node.js** (v18 or higher recommended)
- **npm** (v9 or higher)
- **MongoDB** (Local instance `mongodb://127.0.0.1:27017` OR MongoDB Atlas URI). *Note: An automated fallback ensures zero-configuration startup.*

### 1. Clone & Navigate to Workspace
```bash
cd campus
```

### 2. Install All Dependencies
Install root, backend, and frontend dependencies with a single command:
```bash
npm run install:all
```
*Or install separately:*
```bash
# Backend
cd backend && npm install

# Frontend
cd ../frontend && npm install
```

### 3. Configure Environment Variables
A `.env` file is pre-configured in `backend/.env`:
```env
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb://127.0.0.1:27017/campus_lost_found
JWT_SECRET=campus_lost_found_jwt_secret_key_2026_super_secure
JWT_EXPIRE=30d
```

### 4. Seed Realistic Campus Demo Data
Populate MongoDB with realistic campus users, lost/found items, claims, and notifications:
```bash
npm run seed
```

### 5. Run the Full Stack Application
Start both the Express Backend (`http://localhost:5000`) and the React Frontend (`http://localhost:5173`) concurrently:
```bash
npm run dev
```

Open your browser at **`http://localhost:5173`**.

---

## 🔑 Demo Login Credentials

Quick 1-Click Demo Login buttons are provided on the Login page:

| Role | Name | Email | Password | Access Capabilities |
| :--- | :--- | :--- | :--- | :--- |
| **Student** | Aarav Sharma | `aarav.sharma@campus.edu.in` | `password123` | Report items, submit claims, track claims, edit/delete own posts |
| **Admin** | Prof. K. Verma | `proctor.office@campus.edu.in` | `admin123` | Full Proctor Control Center, approve/reject claims, delete items, manage users |
| **Staff** | Vikram Singh | `vikram.lab@campus.edu.in` | `password123` | Report lab belongings, view claims, manage posts |

---

## 🏗️ System Architecture & Data Flow

### Architectural Layering

```
+-------------------------------------------------------------------------+
|                              USER BROWSER                               |
+-------------------------------------------------------------------------+
                                    │
                                    ▼
+-------------------------------------------------------------------------+
|                  REACT.JS FRONTEND (VITE + ROUTER)                      |
|  • Pages: Home, Dashboard, Lost, Found, Report, ItemDetails, Admin...   |
|  • Components: Navbar, Sidebar, ItemCard, FilterBar, ClaimModal, Toast  |
|  • Context & State: AuthContext (JWT, User Session, Alerts)             |
|  • Central API Client: Axios with Bearer Interceptors (api.js)          |
+-------------------------------------------------------------------------+
                                    │  HTTP Requests (JSON)
                                    ▼  /api/auth, /api/items, /api/claims...
+-------------------------------------------------------------------------+
|                 EXPRESS.JS REST API BACKEND (NODE.JS)                   |
|  • Middlewares: CORS, Express JSON Parser, Auth (JWT), Admin, Error404  |
|  • Controllers: authController, itemController, claimController, etc.   |
|  • Validation & Business Logic: Duplicate Checks, Status Sync, Security |
+-------------------------------------------------------------------------+
                                    │  Mongoose ODM
                                    ▼
+-------------------------------------------------------------------------+
|                           MONGODB DATABASE                              |
|  • Collections: users, items, claims, notifications                     |
|  • Indexes: Text Search Index, Foreign Key References, Compound Keys    |
+-------------------------------------------------------------------------+
```

### Data Flow Example: Claim Verification & Item Status Sync

```
Student Submits Claim in React Modal
               │
               ▼  POST /api/claims { itemId, reason, proof } [Bearer Token]
Express Auth Middleware (validates JWT & attaches req.user)
               │
               ▼
Claim Controller validates Rule 4 (No duplicates) & Rule 5 (Not already returned)
               │
               ▼
Mongoose creates Claim in MongoDB with status: 'pending'
               │
               ▼
Admin logs into Proctor Dashboard & clicks "Approve Claim"
               │
               ▼  PUT /api/claims/:id { status: 'approved', adminNotes: '...' }
Claim Controller sets Claim status to 'approved'
               │
               ▼
Item status automatically updated to 'claimed' in MongoDB (Rule 6)
               │
               ▼
Claimant receives in-app Notification with Proctor verification notes
               │
               ▼
React UI updates claim badge to "Claim Approved" and item to "Claimed"
```

---

## 🗄️ MongoDB Collections & Schema Design

### 1. `users` Collection
| Field | Type | Validation / Rules | Description |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Primary Key | Auto-generated MongoDB ID |
| `name` | `String` | Required, Max 100 | Student/Staff Full Name |
| `email` | `String` | Required, Unique, Lowercase | Institutional Campus Email |
| `password` | `String` | Required, Min 6, Hashed | Bcrypt salted hash (hidden by default) |
| `studentId` | `String` | Optional, Trimmed | Roll number or Employee ID |
| `department`| `String` | Default: `'General'` | College Academic Department |
| `phone` | `String` | Optional | Direct Contact Number |
| `role` | `String` | Enum: `['student', 'staff', 'admin']` | Authorization access level |
| `avatar` | `String` | Derived | Initials badge (e.g. `'AS'`) |
| `createdAt` | `Date` | Auto Timestamp | Registration timestamp |

### 2. `items` Collection
| Field | Type | Validation / Rules | Description |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Primary Key | Item Unique Identifier |
| `itemName` | `String` | Required, Max 150 | Title of the lost/found object |
| `type` | `String` | Enum: `['lost', 'found']` | Report Classification |
| `category` | `String` | Required | Electronics, ID Cards, Bags, Wallets, etc. |
| `description`| `String`| Required | Detailed description of object |
| `location` | `String` | Required | Campus venue (Library, Lab, Canteen, etc.) |
| `date` | `String` | Required | Date lost or recovered |
| `time` | `String` | Optional | Approximate time |
| `color` | `String` | Optional | Color / Finish |
| `brand` | `String` | Optional | Manufacturer / Brand |
| `image` | `String` | URL String | Item Photo URL |
| `status` | `String` | Enum: `['lost', 'found', 'claimed', 'returned']` | Current Lifecycle Status |
| `reportedBy`| `ObjectId` | Ref: `User`, Required | Creator user reference |
| `contactName`| `String`| Optional | Display contact person |
| `contactEmail`|`String`| Optional | Display email address |
| `contactPhone`|`String`| Optional | Direct contact phone |
| `additionalDetails`|`String`| Optional | Handover & recovery notes |
| `createdAt` | `Date` | Auto Timestamp | Date reported |

### 3. `claims` Collection
| Field | Type | Validation / Rules | Description |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Primary Key | Claim Unique ID |
| `item` | `ObjectId` | Ref: `Item`, Required | Associated Item reference |
| `claimant` | `ObjectId` | Ref: `User`, Required | Claimant User reference |
| `claimantName`|`String`| Required | Claimant Name |
| `claimantEmail`|`String`| Required | Claimant Email |
| `claimantPhone`|`String`| Optional | Phone for Proctor handover |
| `studentId` | `String` | Optional | Student Roll Number |
| `reason` | `String` | Required | Distinctive proof of ownership |
| `verificationDetails`|`String`| Optional | Receipts / ID proof notes |
| `status` | `String` | Enum: `['pending', 'approved', 'rejected', 'returned']` | Proctor Review Status |
| `adminNotes`| `String` | Optional | Official Proctor notes & decision log |
| `createdAt` | `Date` | Auto Timestamp | Submission timestamp |

### 4. `notifications` Collection
| Field | Type | Description |
| :--- | :--- | :--- |
| `recipient` | `ObjectId (Ref: User)` | Target user for notification |
| `title` | `String` | Notification Title |
| `message` | `String` | Detailed notification message |
| `type` | `String (Enum: 'info', 'success', 'warning', 'danger')` | Visual icon style |
| `isRead` | `Boolean (Default: false)` | Read status |
| `link` | `String` | Navigation link |

---

## 📡 REST API Reference

### 🔐 Authentication APIs (`/api/auth`)
| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/register` | Public | Register new user with student/staff info & return JWT |
| `POST` | `/api/auth/login` | Public | Validate credentials & return JWT token |
| `GET` | `/api/auth/me` | Private | Get profile of logged-in user |
| `PUT` | `/api/auth/profile` | Private | Update name, department, phone, roll number |
| `PUT` | `/api/auth/password` | Private | Change password with current password verification |

### 📦 Item APIs (`/api/items`)
| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/items` | Public | Query items with `?search=`, `?type=`, `?category=`, `?location=`, `?status=` |
| `GET` | `/api/items/:id` | Public | Get full item specifications, reporter info, and claim history |
| `POST` | `/api/items` | Private | Create a new lost or found item report |
| `PUT` | `/api/items/:id` | Private (Owner/Admin) | Update item details or lifecycle status |
| `DELETE` | `/api/items/:id` | Private (Owner/Admin) | Delete item and cascade remove associated claims |

### 📝 Claim APIs (`/api/claims`)
| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/claims` | Private | Fetch claims (Students see own; Admins see all) |
| `GET` | `/api/claims/:id` | Private | View specific claim details |
| `POST` | `/api/claims` | Private | Submit ownership claim with proof & duplicate check |
| `PUT` | `/api/claims/:id` | Private (Admin Only) | Approve or reject claim; automatically syncs item status |
| `DELETE` | `/api/claims/:id` | Private (Claimant/Admin) | Withdraw / delete claim record |

### 🛡️ Admin & User APIs (`/api/admin`, `/api/users`)
| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/admin/dashboard`| Public/Private | Real-time aggregated statistics from MongoDB |
| `POST` | `/api/seed` | Public/Admin | Reset database collections to initial clean state |
| `GET` | `/api/users` | Private (Admin Only) | List all registered users with post & claim counts |
| `PUT` | `/api/users/:id` | Private (Admin/Self) | Update user details or assign role (`student`/`staff`/`admin`) |
| `DELETE` | `/api/users/:id` | Private (Admin Only) | Delete user and cascade delete their posts & claims |

### 🔔 Notification APIs (`/api/notifications`)
| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/notifications` | Private | Get user notifications with unread counter |
| `PUT` | `/api/notifications/:id/read` | Private | Mark single notification as read |
| `PUT` | `/api/notifications/read-all` | Private | Mark all notifications as read |

---

## ⚡ Implemented Business Rules

- **RULE 1 (Ownership Protection):** Normal users can only edit or delete their own item posts. Admins have global moderation authority.
- **RULE 2 (Authentication Requirement):** Reporting an item or submitting an ownership claim requires a valid JWT session.
- **RULE 3 (Admin Moderation):** Only users with role `admin` can approve or reject submitted claims.
- **RULE 4 (Duplicate Claim Prevention):** A user cannot submit multiple active (`pending`/`approved`) claims for the same item.
- **RULE 5 (Claim Immutability):** Items with status `returned` or `claimed` cannot accept new claims.
- **RULE 6 (Automated Status Sync):** Approving a claim automatically updates the corresponding Item's status to `claimed`, rejects competing pending claims, and dispatches an in-app notification to the claimant.
- **RULE 7 (Input Validation):** All required item fields (title, category, description, location, date) are validated on both frontend and backend before saving.
- **RULE 8 (Credential Security):** Email format and 6+ character passwords are required during registration; passwords are salted and hashed using `bcryptjs`.
- **RULE 9 (Route Protection):** Protected routes (`/dashboard`, `/report-item`, `/my-posts`, `/my-claims`, `/profile`) redirect unauthenticated users to `/login`. Admin routes (`/admin`) restrict access to admin accounts.

---

## 📁 Project Directory Structure

```
campus/
├── package.json                 # Root orchestration scripts (concurrent dev, seed, install)
├── README.md                    # Complete project guide & API docs
├── DOCUMENTATION.md             # Academic submission report
│
├── backend/                     # Node.js + Express.js + MongoDB API
│   ├── config/
│   │   └── db.js                # MongoDB connection + resilient in-memory fallback
│   ├── controllers/
│   │   ├── authController.js    # Register, Login, Me, Profile, Password
│   │   ├── itemController.js    # CRUD, Search, Category/Location Filter
│   │   ├── claimController.js   # Submit, Review, Approve, Reject, Item Sync
│   │   ├── userController.js    # User CRUD, Profile management
│   │   ├── adminController.js   # Dashboard metrics, User/Item moderation
│   │   └── notificationController.js # Notifications & Unread count
│   ├── models/
│   │   ├── User.js              # User Mongoose schema & bcrypt hash
│   │   ├── Item.js              # Item Mongoose schema with text search index
│   │   ├── Claim.js             # Claim Mongoose schema with foreign key refs
│   │   └── Notification.js      # User notification schema
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── itemRoutes.js
│   │   ├── claimRoutes.js
│   │   ├── userRoutes.js
│   │   ├── adminRoutes.js
│   │   └── notificationRoutes.js
│   ├── middleware/
│   │   ├── authMiddleware.js    # JWT token verification
│   │   ├── adminMiddleware.js   # Admin role validation
│   │   └── errorMiddleware.js   # Centralized error handling & 404
│   ├── utils/
│   │   ├── generateToken.js     # JWT utility
│   │   └── seeder.js            # Realistic campus seed data
│   ├── .env                     # Server environment variables
│   ├── .env.example
│   ├── server.js                # Express app entrypoint & API mounting
│   └── package.json
│
├── frontend/                    # React.js Frontend (Vite)
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Navbar.jsx       # Public landing navbar
│   │   │   ├── Sidebar.jsx      # Role-aware dashboard sidebar
│   │   │   ├── Topbar.jsx       # Global search, notifications, profile
│   │   │   ├── ItemCard.jsx     # Reusable item card with status badges
│   │   │   ├── FilterBar.jsx    # Category, Location, Status, Type filters
│   │   │   ├── ClaimModal.jsx   # Interactive claim verification modal
│   │   │   ├── StatusBadge.jsx  # Status badge component
│   │   │   ├── LoadingSpinner.jsx # Loading spinners & skeleton cards
│   │   │   ├── EmptyState.jsx   # Clean empty state UI
│   │   │   ├── ConfirmationModal.jsx # Danger & action confirmation
│   │   │   ├── Toast.jsx        # Floating notifications
│   │   │   └── ProtectedRoute.jsx # Auth & Admin route guards
│   │   ├── pages/
│   │   │   ├── Home.jsx         # Landing page with live campus stats
│   │   │   ├── Login.jsx        # Login with 1-click Demo Fill
│   │   │   ├── Register.jsx     # Registration with student/staff info
│   │   │   ├── Dashboard.jsx    # User dashboard with metrics & recent feed
│   │   │   ├── LostItems.jsx    # Lost items directory with search/filters
│   │   │   ├── FoundItems.jsx   # Found items directory + Claim trigger
│   │   │   ├── ReportItem.jsx   # Create & edit lost/found item form
│   │   │   ├── ItemDetails.jsx  # Item details view & location/contact
│   │   │   ├── MyPosts.jsx      # Manage own items (edit/delete/status)
│   │   │   ├── MyClaims.jsx     # Manage & track own claims
│   │   │   ├── Profile.jsx      # User profile, stats & password update
│   │   │   ├── Notifications.jsx# Notification list & mark as read
│   │   │   └── AdminDashboard.jsx # Full Admin panel (stats, users, items, claims)
│   │   ├── context/
│   │   │   └── AuthContext.jsx  # Auth provider (user state, login, logout)
│   │   ├── hooks/
│   │   │   └── useAuth.js       # Custom auth hook
│   │   ├── services/
│   │   │   └── api.js           # Centralized Axios API client with JWT interceptor
│   │   ├── styles/              # Preserved CSS styling system
│   │   │   ├── style.css
│   │   │   ├── dashboard.css
│   │   │   └── responsive.css
│   │   ├── App.jsx              # React Router routes setup
│   │   └── main.jsx
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
```

---

## 🧪 Comprehensive Verification Checklist

| Test Item | Verification Method | Status |
| :--- | :--- | :--- |
| **Backend Server** | Express starts on port `5000` with `/api/health` returning `online` | ✅ Pass |
| **MongoDB Connection** | Mongoose connects to database & creates collections | ✅ Pass |
| **User Registration** | New user saved to MongoDB with bcrypt hashed password & returns JWT | ✅ Pass |
| **User Login** | Email/password verified, returns valid JWT token and user info | ✅ Pass |
| **1-Click Demo Login** | Student and Admin demo fill buttons authenticate immediately | ✅ Pass |
| **Create Lost Item** | Item saved in MongoDB with `type: 'lost'`, `reportedBy: user._id` | ✅ Pass |
| **Create Found Item** | Item saved in MongoDB with `type: 'found'`, `status: 'found'` | ✅ Pass |
| **Search & Filtering** | Filtering by category, location, and search text performs MongoDB query | ✅ Pass |
| **Claim Submission** | Claimant submits proof; verified against duplicate rule | ✅ Pass |
| **Admin Claim Approval**| Admin approves claim; item status automatically updates to `claimed` | ✅ Pass |
| **Admin Moderation** | Admin can delete inappropriate items and manage user roles | ✅ Pass |
| **Edit/Delete Post** | Student can edit and delete only their own posts | ✅ Pass |
| **Dashboard Metrics** | Metric cards query MongoDB dynamically (no hardcoded numbers) | ✅ Pass |
| **Protected Routes** | Unauthorized navigation to `/dashboard` redirects to `/login` | ✅ Pass |
| **Admin Routes** | Student attempting to access `/admin` redirects to `/dashboard` | ✅ Pass |
| **Responsive Design** | UI renders cleanly on mobile, tablet, and desktop viewports | ✅ Pass |

---

## 🎓 Academic Submission Compliance
This project satisfies all requirements for **Assignment 2**:
1. Full functional integration of **React.js → Node.js → Express.js → MongoDB**.
2. Clean separation of concerns (Models, Controllers, Routes, Middlewares, Services, Components, Pages).
3. Realistic campus seed data and zero-setup database resilience.
4. Comprehensive academic documentation in `DOCUMENTATION.md`.
