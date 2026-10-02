# 📡 REST API Specification & Endpoint Contracts

This document contains the complete specification for the REST API endpoints provided by the **Mini E-Commerce Demo Backend** (`/server`), including request parameters, payload bodies, validation rules, HTTP status codes, and JSON response structures.

---

## 📑 Table of Contents

- [1. General API Conventions](#1-general-api-conventions)
- [2. Authentication Endpoints (`/api/auth`)](#2-authentication-endpoints-apiauth)
  - [2.1 Register Customer (`POST /api/auth/register`)](#21-register-customer-post-apiauthregister)
  - [2.2 Login User (`POST /api/auth/login`)](#22-login-user-post-apiauthlogin)
- [3. Category Endpoints (`/api/categories`)](#3-category-endpoints-apicategories)
  - [3.1 List Categories (`GET /api/categories`)](#31-list-categories-get-apicategories)
  - [3.2 Create Category (`POST /api/categories`)](#32-create-category-post-apicategories)
  - [3.3 Update Category (`PUT /api/categories/:id`)](#33-update-category-put-apicategoriesid)
  - [3.4 Delete Category (`DELETE /api/categories/:id`)](#34-delete-category-delete-apicategoriesid)
- [4. Product Endpoints (`/api/products`)](#4-product-endpoints-apiproducts)
  - [4.1 List Products with Filters (`GET /api/products`)](#41-list-products-with-filters-get-apiproducts)
  - [4.2 Get Product Details (`GET /api/products/:id`)](#42-get-product-details-get-apiproductsid)
  - [4.3 Create Product (`POST /api/products`)](#43-create-product-post-apiproducts)
  - [4.4 Update Product (`PUT /api/products/:id`)](#44-update-product-put-apiproductsid)
  - [4.5 Delete Product (`DELETE /api/products/:id`)](#45-delete-product-delete-apiproductsid)
- [5. Order Endpoints (`/api/orders`)](#5-order-endpoints-apiorders)
  - [5.1 Place Order (`POST /api/orders`)](#51-place-order-post-apiorders)
  - [5.2 List My Orders (`GET /api/orders/my-orders`)](#52-list-my-orders-get-apiordersmy-orders)
  - [5.3 List All Orders - Admin (`GET /api/admin/orders`)](#53-list-all-orders---admin-get-apiadminorders)
  - [5.4 Update Order Status - Admin (`PATCH /api/admin/orders/:id/status`)](#54-update-order-status---admin-patch-apiadminordersidstatus)
- [6. Standard Error Response Format](#6-standard-error-response-format)

---

## 1. General API Conventions

- **Base URL**: `http://localhost:5000/api`
- **Request Format**: `Content-Type: application/json`
- **Authorization**: `Authorization: Bearer <jwt_token>` (for protected endpoints)
- **Status Codes**:
  - `200 OK`: Request succeeded.
  - `201 Created`: Resource successfully created.
  - `400 Bad Request`: Validation failure or business constraint violation.
  - `401 Unauthorized`: Missing, expired, or invalid JWT token.
  - `403 Forbidden`: Authenticated user lacks sufficient permissions (e.g. non-admin).
  - `404 Not Found`: Requested resource does not exist.
  - `500 Internal Server Error`: Unexpected server exception.

---

## 2. Authentication Endpoints (`/api/auth`)

### 2.1 Register Customer (`POST /api/auth/register`)
Creates a new customer account. Returns user profile and initial JWT bearer token.

- **Access**: Public
- **Request Body**:
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "secretpassword",
  "confirmPassword": "secretpassword"
}
```
- **Validation**:
  - `name`: Required, non-empty string.
  - `email`: Required, valid email format, unique in database.
  - `password`: Required, minimum 6 characters.
  - `confirmPassword`: Must match `password` exactly.
- **Success Response (`201 Created`)**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "660c1a8f9b1d8e001f34a101",
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "customer"
  }
}
```
- **Error Response (`400 Bad Request`)**:
```json
{
  "success": false,
  "message": "Passwords do not match"
}
```

---

### 2.2 Login User (`POST /api/auth/login`)
Authenticates email and password for customers and admins. Returns JWT token.

- **Access**: Public
- **Request Body**:
```json
{
  "email": "admin@example.com",
  "password": "admin123"
}
```
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "660c1a8f9b1d8e001f34a999",
    "name": "System Admin",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```
- **Error Response (`401 Unauthorized`)**:
```json
{
  "success": false,
  "message": "Invalid email or password"
}
```

---

## 3. Category Endpoints (`/api/categories`)

### 3.1 List Categories (`GET /api/categories`)
Fetches all product categories.

- **Access**: Public
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "count": 3,
  "categories": [
    {
      "_id": "660c1a8f9b1d8e001f34a201",
      "name": "Electronics",
      "description": "Smartphones, laptops, and gadgets",
      "createdAt": "2026-10-02T10:00:00.000Z"
    },
    {
      "_id": "660c1a8f9b1d8e001f34a202",
      "name": "Fashion",
      "description": "Apparel, clothing, and accessories",
      "createdAt": "2026-10-02T10:01:00.000Z"
    }
  ]
}
```

---

### 3.2 Create Category (`POST /api/categories`)
Creates a new category.

- **Access**: Admin (Requires `protect` + `adminOnly`)
- **Headers**: `Authorization: Bearer <admin_token>`
- **Request Body**:
```json
{
  "name": "Shoes",
  "description": "Athletic, casual, and formal footwear"
}
```
- **Success Response (`201 Created`)**:
```json
{
  "success": true,
  "message": "Category created successfully",
  "category": {
    "_id": "660c1a8f9b1d8e001f34a203",
    "name": "Shoes",
    "description": "Athletic, casual, and formal footwear",
    "createdAt": "2026-10-02T10:15:00.000Z"
  }
}
```

---

### 3.3 Update Category (`PUT /api/categories/:id`)
Updates an existing category.

- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin_token>`
- **Request Body**:
```json
{
  "name": "Footwear & Shoes",
  "description": "All types of shoes, sneakers, and boots"
}
```
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Category updated successfully",
  "category": {
    "_id": "660c1a8f9b1d8e001f34a203",
    "name": "Footwear & Shoes",
    "description": "All types of shoes, sneakers, and boots",
    "updatedAt": "2026-10-02T10:20:00.000Z"
  }
}
```

---

### 3.4 Delete Category (`DELETE /api/categories/:id`)
Deletes a category. Prevents deletion if products are currently linked to this category.

- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin_token>`
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Category removed successfully"
}
```
- **Error Response (`400 Bad Request`)**:
```json
{
  "success": false,
  "message": "Cannot delete category: 4 products are currently assigned to it"
}
```

---

## 4. Product Endpoints (`/api/products`)

### 4.1 List Products with Filters (`GET /api/products`)
Retrieves products with optional query filtering for category and search term.

- **Access**: Public
- **Query Parameters**:
  - `category` (optional): Category ID or Category Name (e.g. `?category=electronics`)
  - `search` (optional): Keyword search string (e.g. `?search=headphones`)
- **Example Request**:
  `GET /api/products?category=electronics&search=phone`
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "count": 1,
  "products": [
    {
      "_id": "660c1a8f9b1d8e001f34a301",
      "name": "Wireless Over-Ear Headphones",
      "description": "High-fidelity Bluetooth headphones with active noise cancellation",
      "price": 99.99,
      "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e",
      "category": {
        "_id": "660c1a8f9b1d8e001f34a201",
        "name": "Electronics"
      },
      "stock": 15,
      "createdAt": "2026-10-02T10:10:00.000Z"
    }
  ]
}
```

---

### 4.2 Get Product Details (`GET /api/products/:id`)
Fetches single product details with populated category information.

- **Access**: Public
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "product": {
    "_id": "660c1a8f9b1d8e001f34a301",
    "name": "Wireless Over-Ear Headphones",
    "description": "High-fidelity Bluetooth headphones with active noise cancellation",
    "price": 99.99,
    "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e",
    "category": {
      "_id": "660c1a8f9b1d8e001f34a201",
      "name": "Electronics",
      "description": "Smartphones, laptops, and gadgets"
    },
    "stock": 15
  }
}
```

---

### 4.3 Create Product (`POST /api/products`)
Creates a new catalog product.

- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin_token>`
- **Request Body**:
```json
{
  "name": "Mechanical Gaming Keyboard",
  "description": "RGB backlit mechanical keyboard with tactile blue switches",
  "price": 79.99,
  "image": "https://images.unsplash.com/photo-1587829741301-dc798b83add3",
  "category": "660c1a8f9b1d8e001f34a201",
  "stock": 20
}
```
- **Success Response (`201 Created`)**:
```json
{
  "success": true,
  "message": "Product created successfully",
  "product": {
    "_id": "660c1a8f9b1d8e001f34a302",
    "name": "Mechanical Gaming Keyboard",
    "price": 79.99,
    "stock": 20,
    "createdAt": "2026-10-02T10:30:00.000Z"
  }
}
```

---

### 4.4 Update Product (`PUT /api/products/:id`)
Updates product specifications, price, or inventory count.

- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin_token>`
- **Request Body**:
```json
{
  "price": 69.99,
  "stock": 25
}
```
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Product updated successfully",
  "product": {
    "_id": "660c1a8f9b1d8e001f34a302",
    "name": "Mechanical Gaming Keyboard",
    "price": 69.99,
    "stock": 25
  }
}
```

---

### 4.5 Delete Product (`DELETE /api/products/:id`)
Deletes a product from the catalog.

- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin_token>`
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

---

## 5. Order Endpoints (`/api/orders`)

### 5.1 Place Order (`POST /api/orders`)
Places a new order using Cash on Delivery. Enforces server-side price lookup and stock decrement.

- **Access**: Customer (Requires `protect`)
- **Headers**: `Authorization: Bearer <customer_token>`
- **Request Body**:
```json
{
  "items": [
    {
      "productId": "660c1a8f9b1d8e001f34a301",
      "quantity": 2
    }
  ],
  "shippingAddress": {
    "name": "Jane Doe",
    "phone": "+1 555-0123",
    "address": "123 Main Street, Apt 4B",
    "city": "Springfield",
    "pincode": "97477"
  }
}
```
- **Execution Logic**:
  1. For each `item`, look up `Product` by `productId`.
  2. Verify `product.stock >= item.quantity`. If false, abort with `400 Bad Request`.
  3. Fetch `product.price` directly from DB. Compute `itemTotal = product.price * item.quantity`.
  4. Sum all item totals to calculate true `totalAmount`.
  5. Save `Order` with `status: 'Pending'` and `paymentMethod: 'COD'`.
  6. Decrement stock for each product: `Product.updateOne({ _id: item.productId }, { $inc: { stock: -item.quantity } })`.
- **Success Response (`201 Created`)**:
```json
{
  "success": true,
  "message": "Order placed successfully",
  "order": {
    "_id": "660c1a8f9b1d8e001f34a501",
    "user": "660c1a8f9b1d8e001f34a101",
    "products": [
      {
        "product": "660c1a8f9b1d8e001f34a301",
        "name": "Wireless Over-Ear Headphones",
        "price": 99.99,
        "quantity": 2
      }
    ],
    "totalAmount": 199.98,
    "shippingAddress": {
      "name": "Jane Doe",
      "phone": "+1 555-0123",
      "address": "123 Main Street, Apt 4B",
      "city": "Springfield",
      "pincode": "97477"
    },
    "paymentMethod": "COD",
    "status": "Pending",
    "createdAt": "2026-10-02T11:00:00.000Z"
  }
}
```

---

### 5.2 List My Orders (`GET /api/orders/my-orders`)
Returns all orders placed by the currently authenticated customer, sorted newest first.

- **Access**: Customer (`protect`)
- **Headers**: `Authorization: Bearer <customer_token>`
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "count": 1,
  "orders": [
    {
      "_id": "660c1a8f9b1d8e001f34a501",
      "totalAmount": 199.98,
      "status": "Pending",
      "paymentMethod": "COD",
      "createdAt": "2026-10-02T11:00:00.000Z",
      "products": [
        {
          "name": "Wireless Over-Ear Headphones",
          "price": 99.99,
          "quantity": 2
        }
      ]
    }
  ]
}
```

---

### 5.3 List All Orders - Admin (`GET /api/admin/orders`)
Returns all customer orders across the entire platform with populated user details.

- **Access**: Admin (`protect` + `adminOnly`)
- **Headers**: `Authorization: Bearer <admin_token>`
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "count": 12,
  "orders": [
    {
      "_id": "660c1a8f9b1d8e001f34a501",
      "user": {
        "_id": "660c1a8f9b1d8e001f34a101",
        "name": "Jane Doe",
        "email": "jane@example.com"
      },
      "totalAmount": 199.98,
      "status": "Pending",
      "shippingAddress": {
        "name": "Jane Doe",
        "phone": "+1 555-0123",
        "city": "Springfield",
        "pincode": "97477"
      },
      "createdAt": "2026-10-02T11:00:00.000Z"
    }
  ]
}
```

---

### 5.4 Update Order Status - Admin (`PATCH /api/admin/orders/:id/status`)
Updates the lifecycle status of an order.

- **Access**: Admin (`protect` + `adminOnly`)
- **Headers**: `Authorization: Bearer <admin_token>`
- **Request Body**:
```json
{
  "status": "Shipped"
}
```
- **Allowed Status Values**:
  - `"Pending"`
  - `"Confirmed"`
  - `"Shipped"`
  - `"Delivered"`
  - `"Cancelled"`
- **Success Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Order status updated to Shipped",
  "order": {
    "_id": "660c1a8f9b1d8e001f34a501",
    "status": "Shipped",
    "updatedAt": "2026-10-02T11:45:00.000Z"
  }
}
```
- **Error Response (`400 Bad Request`)**:
```json
{
  "success": false,
  "message": "Invalid status value: InTransit is not supported"
}
```

---

## 6. Standard Error Response Format

All failed requests return a unified JSON error structure:

```json
{
  "success": false,
  "message": "Descriptive human-readable error explanation",
  "stack": "Error stack trace (only present in development mode)"
}
```
