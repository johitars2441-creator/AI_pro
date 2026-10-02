# 🏛 System Architecture & Design Blueprint

This document specifies the technical architecture, component relationships, data flow diagrams, and design patterns for the **Mini E-Commerce Demo Project (MERN Stack)**.

---

## 📑 Table of Contents

- [1. High-Level Architecture](#1-high-level-architecture)
- [2. System Topology](#2-system-topology)
- [3. Frontend Architecture (`/client`)](#3-frontend-architecture-client)
  - [3.1 Technology Stack & Build Tools](#31-technology-stack--build-tools)
  - [3.2 State Management Strategy](#32-state-management-strategy)
  - [3.3 Routing & Route Guards](#33-routing--route-guards)
- [4. Backend Architecture (`/server`)](#4-backend-architecture-server)
  - [4.1 Layered Architecture Pattern](#41-layered-architecture-pattern)
  - [4.2 Middleware Pipeline](#42-middleware-pipeline)
  - [4.3 Database Integration](#43-database-integration)
- [5. System Sequence Diagrams](#5-system-sequence-diagrams)
  - [5.1 User Authentication Flow](#51-user-authentication-flow)
  - [5.2 Product Browsing & Filtering Flow](#52-product-browsing--filtering-flow)
  - [5.3 Order Placement & Atomic Stock Decrement Flow](#53-order-placement--atomic-stock-decrement-flow)
  - [5.4 Admin Order Fulfillment Lifecycle](#54-admin-order-fulfillment-lifecycle)
- [6. Directory Layout & Module Responsibilities](#6-directory-layout--module-responsibilities)

---

## 1. High-Level Architecture

The system operates on a decoupled **Client-Server Single Page Application (SPA)** model communicating over a stateless JSON REST API.

```mermaid
graph TD
    subgraph Browser ["Client Application (React + Vite)"]
        UI["Tailwind CSS UI Components"]
        Router["React Router v6"]
        AuthCtx["Auth Context (JWT + User State)"]
        CartCtx["Cart Context (Local Storage)"]
        Axios["Axios HTTP Client (Bearer Interceptor)"]
        
        UI --> Router
        UI --> AuthCtx
        UI --> CartCtx
        AuthCtx --> Axios
        CartCtx --> Axios
    end

    subgraph Server ["Backend Server (Node.js + Express)"]
        API["Express App (/api)"]
        CorsMW["CORS & Body Parser"]
        AuthMW["Auth & Admin Middleware"]
        Controllers["Controllers (Auth, Category, Product, Order)"]
        Models["Mongoose Models (User, Category, Product, Order)"]

        API --> CorsMW
        CorsMW --> AuthMW
        AuthMW --> Controllers
        Controllers --> Models
    end

    subgraph Database ["Persistence Layer (MongoDB)"]
        Mongo[("MongoDB Database\n(Collections: users, categories, products, orders)")]
        Models --> Mongo
    end

    Axios -->|"HTTPS / JSON REST"| API
```

---

## 2. System Topology

| Component | Host / Port | Responsibilities |
| :--- | :--- | :--- |
| **Frontend Client** | `http://localhost:5173` | Renders HTML/CSS DOM, handles user interactions, manages client-side routing, persists cart in localStorage, stores JWT session. |
| **Backend API** | `http://localhost:5000` | Exposes REST endpoints, validates payloads, enforces role-based authorization, executes business logic, handles database queries. |
| **Database** | `mongodb://localhost:27017` | Persists user credentials, product catalog, categories, and order transaction documents with ACID document guarantees. |

---

## 3. Frontend Architecture (`/client`)

### 3.1 Technology Stack & Build Tools
- **Vite**: Provides instant hot-module replacement (HMR) and optimized ES module bundling.
- **React 18**: Uses functional components and modern React hooks (`useState`, `useEffect`, `useContext`, `useMemo`, `useCallback`).
- **Tailwind CSS 3**: Utility-first CSS framework configured with responsive breakpoints (`sm`, `md`, `lg`, `xl`) and standard design tokens.
- **Axios Instance**: Configured with a central baseURL (`VITE_API_BASE_URL`) and an automated request interceptor that injects `Authorization: Bearer <token>`.

### 3.2 State Management Strategy
To adhere to the requirement of keeping this a lean mini-project, external complex state libraries (Redux, Zustand) are avoided. Instead, native **React Context API** is utilized:

1. **`AuthContext`**:
   - Stores current user info `{ _id, name, email, role }` and the raw JWT token.
   - Initialized by checking `localStorage.getItem('token')`.
   - Exposes methods: `login(email, password)`, `register(formData)`, and `logout()`.
   - Synchronizes token persistence in `localStorage`.

2. **`CartContext`**:
   - Stores an array of cart items: `[{ product, quantity }]`.
   - Hydrated from `localStorage.getItem('cart')` on initial load.
   - Enforces stock boundary condition: prevents `quantity > product.stock`.
   - Exposes methods: `addToCart(product, qty)`, `updateQuantity(productId, qty)`, `removeFromCart(productId)`, `clearCart()`, and derived state `cartTotal`, `cartCount`.

### 3.3 Routing & Route Guards
React Router v6 structures the application into three access zones:

```mermaid
graph TD
    RouteRoot["App Router"]
    
    subgraph PublicRoutes ["Public Zone"]
        Home["/ (Home)"]
        Products["/products (Catalog)"]
        ProdDetail["/products/:id (Details)"]
        Cart["/cart (Shopping Cart)"]
        Login["/login (Login)"]
        Register["/register (Register)"]
    end

    subgraph CustomerRoutes ["Protected Customer Zone (Requires Auth)"]
        Checkout["/checkout (Place Order)"]
        MyOrders["/my-orders (Order History)"]
    end

    subgraph AdminRoutes ["Protected Admin Zone (Requires role == 'admin')"]
        AdminDash["/admin (Overview)"]
        AdminCats["/admin/categories (Category CRUD)"]
        AdminProds["/admin/products (Product CRUD)"]
        AdminOrders["/admin/orders (Order Status Fulfillment)"]
    end

    RouteRoot --> PublicRoutes
    RouteRoot -->|"ProtectedRoute"| CustomerRoutes
    RouteRoot -->|"AdminRoute"| AdminRoutes
```

---

## 4. Backend Architecture (`/server`)

### 4.1 Layered Architecture Pattern
The backend enforces a clean separation of concerns using the classic **Controller-Service-Model** pattern:

1. **Route Layer (`/routes`)**: Defines endpoint URLs, HTTP verbs, and mounts appropriate middlewares.
2. **Middleware Layer (`/middleware`)**: Handles cross-cutting concerns (token verification, role validation, input sanitization, global error catching).
3. **Controller Layer (`/controllers`)**: Extracts parameters from `req.body`, `req.query`, and `req.params`, performs input checks, coordinates model operations, and returns standardized JSON responses.
4. **Data Model Layer (`/models`)**: Defines Mongoose schemas, validators, pre-save hooks (bcrypt hashing), and indexes.

### 4.2 Middleware Pipeline

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Express as Express Routing
    participant AuthMW as Auth Middleware (protect)
    participant AdminMW as Admin Middleware (adminOnly)
    participant Controller as Route Controller
    participant DB as MongoDB

    Client->>Express: HTTP Request + Bearer Token
    Express->>AuthMW: Inspect Authorization Header
    alt Token Missing or Invalid
        AuthMW-->>Client: 401 Unauthorized { message: "Not authorized, token failed" }
    else Valid Token
        AuthMW->>DB: User.findById(decoded.id).select('-password')
        DB-->>AuthMW: User document
        AuthMW->>Express: req.user = user; next()
        alt Admin Route Requested
            Express->>AdminMW: Check req.user.role === 'admin'
            alt Not Admin
                AdminMW-->>Client: 403 Forbidden { message: "Admin access required" }
            else Is Admin
                AdminMW->>Controller: next()
            end
        else Standard Protected Route
            Express->>Controller: Invoke controller handler
        end
        Controller->>DB: Execute business logic
        DB-->>Controller: Result
        Controller-->>Client: 200/201 JSON Response
    end
```

---

## 5. System Sequence Diagrams

### 5.1 User Authentication Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as Customer / Admin
    participant Frontend as React Client
    participant AuthAPI as /api/auth/login
    participant UserDB as Users Collection

    User->>Frontend: Enter Email & Password
    Frontend->>Frontend: Validate client-side (non-empty, email format)
    Frontend->>AuthAPI: POST { email, password }
    AuthAPI->>UserDB: User.findOne({ email })
    alt User Not Found
        UserDB-->>AuthAPI: null
        AuthAPI-->>Frontend: 401 Unauthorized ("Invalid email or password")
        Frontend-->>User: Display error toast
    else User Found
        AuthAPI->>AuthAPI: bcrypt.compare(password, user.password)
        alt Password Mismatch
            AuthAPI-->>Frontend: 401 Unauthorized ("Invalid email or password")
            Frontend-->>User: Display error toast
        else Password Valid
            AuthAPI->>AuthAPI: jwt.sign({ id: user._id, role: user.role }, JWT_SECRET, { expiresIn: '7d' })
            AuthAPI-->>Frontend: 200 OK { token, user: { _id, name, email, role } }
            Frontend->>Frontend: Save token to localStorage & update AuthContext
            alt user.role === 'admin'
                Frontend-->>User: Redirect to /admin
            else user.role === 'customer'
                Frontend-->>User: Redirect to / (or previous page)
            end
        end
    end
```

### 5.2 Product Browsing & Filtering Flow

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant UI as Products Page
    participant ProdAPI as /api/products
    participant DB as MongoDB Products

    Customer->>UI: Select Category "Electronics" & type "phone"
    UI->>ProdAPI: GET /api/products?category=electronics&search=phone
    ProdAPI->>DB: Product.find({ category: catId, name: { $regex: 'phone', $options: 'i' } })
    DB-->>ProdAPI: Matched Product documents
    ProdAPI-->>UI: 200 OK [ { _id, name, price, stock, image, category }, ... ]
    UI-->>Customer: Render responsive product card grid
```

### 5.3 Order Placement & Atomic Stock Decrement Flow

> [!IMPORTANT]
> **Zero-Trust Pricing Rule**: The frontend submits only product IDs and requested quantities. The server performs an authoritative price lookup directly from MongoDB to compute the true total.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant CheckoutUI as Checkout Page
    participant OrderAPI as POST /api/orders
    participant ProdDB as Products Collection
    participant OrderDB as Orders Collection

    Customer->>CheckoutUI: Submit shipping details + Cash on Delivery
    CheckoutUI->>OrderAPI: POST { items: [{ productId, quantity }], shippingAddress }
    OrderAPI->>OrderAPI: Verify req.user is authenticated
    loop For each item in items
        OrderAPI->>ProdDB: Product.findById(item.productId)
        ProdDB-->>OrderAPI: Product record
        alt Product not found OR Product.stock < item.quantity
            OrderAPI-->>CheckoutUI: 400 Bad Request ("Insufficient stock for item: " + name)
            CheckoutUI-->>Customer: Display error toast & prevent order
        else Stock is sufficient
            OrderAPI->>OrderAPI: Calculate itemSubtotal = Product.price * item.quantity
            OrderAPI->>OrderAPI: Add to totalAmount & prepare snapshot item
        end
    end
    OrderAPI->>OrderDB: Order.create({ user: req.user._id, products, totalAmount, shippingAddress, status: "Pending" })
    OrderDB-->>OrderAPI: Created Order Document
    loop For each item in items
        OrderAPI->>ProdDB: Product.findByIdAndUpdate(productId, { $inc: { stock: -quantity } })
    end
    OrderAPI-->>CheckoutUI: 201 Created { message: "Order placed successfully", order }
    CheckoutUI->>CheckoutUI: cartContext.clearCart()
    CheckoutUI-->>Customer: Redirect to /my-orders with success toast
```

### 5.4 Admin Order Fulfillment Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Pending : Customer places order
    Pending --> Confirmed : Admin verifies order details
    Confirmed --> Shipped : Admin dispatches package
    Shipped --> Delivered : Customer receives package
    
    Pending --> Cancelled : Admin cancels (out of stock/invalid address)
    Confirmed --> Cancelled : Admin cancels prior to shipment
    
    Delivered --> [*]
    Cancelled --> [*]
```

---

## 6. Directory Layout & Module Responsibilities

### Client Modules
- `src/api/client.js`: Configured Axios instance with request and response interceptors.
- `src/context/AuthContext.jsx`: User state, login, register, logout, token persistence.
- `src/context/CartContext.jsx`: Cart items, quantity controls, stock bounds checking.
- `src/components/common/Navbar.jsx`: Storefront header with category shortcuts, search trigger, cart counter, user menu.
- `src/components/admin/AdminSidebar.jsx`: Collapsible navigation sidebar for Categories, Products, Orders, and Storefront switch.
- `src/pages/admin/AdminOrders.jsx`: Table view of customer orders with live status mutation selector.

### Server Modules
- `src/config/db.js`: Initializes Mongoose connection with error handling and reconnection logic.
- `src/middleware/authMiddleware.js`: Extracts Bearer token, verifies JWT, attaches `req.user`.
- `src/middleware/adminMiddleware.js`: Checks `req.user && req.user.role === 'admin'`.
- `src/controllers/orderController.js`: Encapsulates zero-trust pricing, stock validation, decrement, and status transitions.
- `src/seeder/seed.js`: Standalone script to populate a default admin (`admin@example.com` / `admin123`), sample categories, and demo products.
