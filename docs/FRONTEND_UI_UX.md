# 🎨 Frontend UI/UX Design System & Wireframe Specifications

This document outlines the design tokens, component hierarchy, responsive layouts, page-by-page wireframe blueprints, and interactive UI states for the **Mini E-Commerce Demo Project**.

---

## 📑 Table of Contents

- [1. Design System & Tailwind CSS Foundations](#1-design-system--tailwind-css-foundations)
  - [1.1 Color Palette](#11-color-palette)
  - [1.2 Typography & Elevation](#12-typography--elevation)
  - [1.3 Responsive Breakpoints](#13-responsive-breakpoints)
- [2. Component Architecture](#2-component-architecture)
- [3. Public Storefront Wireframes](#3-public-storefront-wireframes)
  - [3.1 Navigation Header (`Navbar.jsx`)](#31-navigation-header-navbarjsx)
  - [3.2 Home Page (`Home.jsx`)](#32-home-page-homejsx)
  - [3.3 Product Catalog (`Products.jsx`)](#33-product-catalog-productsjsx)
  - [3.4 Product Details (`ProductDetails.jsx`)](#34-product-details-productdetailsjsx)
  - [3.5 Shopping Cart (`Cart.jsx`)](#35-shopping-cart-cartjsx)
  - [3.6 Checkout & COD Order Placement (`Checkout.jsx`)](#36-checkout--cod-order-placement-checkoutjsx)
  - [3.7 Customer Order History (`MyOrders.jsx`)](#37-customer-order-history-myordersjsx)
  - [3.8 Login & Register Cards (`Login.jsx`, `Register.jsx`)](#38-login--register-cards-loginjsx-registerjsx)
- [4. Admin Panel Wireframes](#4-admin-panel-wireframes)
  - [4.1 Admin Shell Layout & Sidebar (`AdminLayout.jsx`)](#41-admin-shell-layout--sidebar-adminlayoutjsx)
  - [4.2 Category Management (`AdminCategories.jsx`)](#42-category-management-admincategoriesjsx)
  - [4.3 Product Management (`AdminProducts.jsx`)](#43-product-management-adminproductsjsx)
  - [4.4 Order Management & Status Transition (`AdminOrders.jsx`)](#44-order-management--status-transition-adminordersjsx)
- [5. Micro-Interactions & Feedback States](#5-micro-interactions--feedback-states)

---

## 1. Design System & Tailwind CSS Foundations

The user interface follows a modern, crisp, and distraction-free design aesthetic built with **Tailwind CSS**.

### 1.1 Color Palette

```text
Neutral Surfaces:
  - App Background:       bg-slate-50 (#F8FAFC)
  - Card & Container:     bg-white (#FFFFFF)
  - Border Lines:         border-slate-200 (#E2E8F0)
  - Primary Text:         text-slate-900 (#0F172A)
  - Secondary Text:       text-slate-600 (#475569)
  - Subdued / Placeholders: text-slate-400 (#94A3B8)

Brand Accents:
  - Primary Accent:       bg-indigo-600 hover:bg-indigo-700 text-white
  - Primary Subtle:       bg-indigo-50 text-indigo-700 border-indigo-200

Status & Feedback:
  - Success / In Stock:   bg-emerald-50 text-emerald-700 border-emerald-200
  - Warning / Low Stock:  bg-amber-50 text-amber-700 border-amber-200
  - Danger / Out of Stock:bg-rose-50 text-rose-700 border-rose-200
  - Shipped / Info:       bg-blue-50 text-blue-700 border-blue-200
```

### 1.2 Typography & Elevation
- **Font Family**: System Sans-Serif (`Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`).
- **Cards**: `rounded-xl border border-slate-200 bg-white shadow-sm hover:shadow-md transition-shadow duration-200`.
- **Buttons**: `rounded-lg font-medium transition-all duration-150 active:scale-[0.98]`.
- **Inputs**: `rounded-lg border border-slate-300 px-3.5 py-2.5 text-sm focus:border-indigo-600 focus:ring-2 focus:ring-indigo-100 outline-none`.

### 1.3 Responsive Breakpoints
- Mobile: `< 640px` (Single column cards, slide-out hamburger navigation).
- Tablet: `640px - 1024px` (Two column catalog grid, collapsible admin sidebar).
- Desktop: `> 1024px` (Four column catalog grid, fixed admin sidebar).

---

## 2. Component Architecture

```text
src/
├── components/
│   ├── common/
│   │   ├── Navbar.jsx          # Public storefront navigation header
│   │   ├── Footer.jsx          # Simple clean footer with copyright & links
│   │   ├── Modal.jsx           # Reusable dialog modal wrapper
│   │   ├── ConfirmDialog.jsx   # Delete confirmation modal
│   │   ├── LoadingSpinner.jsx  # SVG loading spinner
│   │   └── EmptyState.jsx      # Icon + message + action button
│   ├── product/
│   │   ├── ProductCard.jsx     # Card component with image, tags, price, add-to-cart
│   │   ├── ProductGrid.jsx     # Responsive grid container
│   │   ├── CategoryPills.jsx   # Horizontal category selection pills
│   │   └── SearchBar.jsx       # Search input with debounce & clear button
│   ├── cart/
│   │   ├── CartItemRow.jsx     # Individual item row with quantity adjusters
│   │   └── OrderSummary.jsx    # Sticky price calculation card
│   └── admin/
│       ├── AdminSidebar.jsx    # Persistent navigation bar for admin views
│       ├── AdminHeader.jsx     # Breadcrumb title & admin profile pill
│       └── StatusBadge.jsx     # Color-coded badge for order status
```

---

## 3. Public Storefront Wireframes

### 3.1 Navigation Header (`Navbar.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  [🛒 E-Shop Demo]    [Search products...] [🔍]    Catalog   Cart (3)   [Jane Doe ▼]|
+-----------------------------------------------------------------------------------+
|  All Categories:  [All]  [Electronics]  [Fashion]  [Shoes]  [Accessories]         |
+-----------------------------------------------------------------------------------+
```
- Brand logo links to `/`.
- Search input executes live query filtering on `/products`.
- Cart badge updates dynamically based on items stored in `CartContext`.
- Dropdown menu shows "My Orders", "Admin Panel" (if admin), and "Logout".

---

### 3.2 Home Page (`Home.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  HERO BANNER                                                                      |
|  Discover Quality Products Delivered Directly to Your Door                        |
|  Experience seamless shopping with instant Cash-on-Delivery fulfillment.          |
|  [ Browse Catalog ➔ ]                                                             |
+-----------------------------------------------------------------------------------+
|  FEATURED CATEGORIES                                                              |
|  +----------------+  +----------------+  +----------------+  +----------------+   |
|  |  Electronics   |  |    Fashion     |  |     Shoes      |  |  Accessories   |   |
|  |   12 items     |  |    8 items     |  |    6 items     |  |    4 items     |   |
|  +----------------+  +----------------+  +----------------+  +----------------+   |
+-----------------------------------------------------------------------------------+
|  LATEST ARRIVALS                                                                  |
|  [Product Grid: Top 4 items with quick Add to Cart]                               |
+-----------------------------------------------------------------------------------+
```

---

### 3.3 Product Catalog (`Products.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  Products                                                                         |
|  Search: [ headphones_______ ]   Category: [All] [Electronics*] [Fashion] [Shoes] |
+-----------------------------------------------------------------------------------+
|  +-------------------+  +-------------------+  +-------------------+              |
|  | [ Image ]         |  | [ Image ]         |  | [ Image ]         |              |
|  | Wireless Headset  |  | Studio Earbuds    |  | Gaming Mic        |              |
|  | Electronics       |  | Electronics       |  | Electronics       |              |
|  | $99.99            |  | $49.99            |  | $79.99            |              |
|  | In Stock (15)     |  | In Stock (8)      |  | Out of Stock (0)  |              |
|  | [ Add to Cart ]   |  | [ Add to Cart ]   |  | [ Disabled ]      |              |
|  +-------------------+  +-------------------+  +-------------------+              |
+-----------------------------------------------------------------------------------+
```

---

### 3.4 Product Details (`ProductDetails.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  Home > Electronics > Wireless Over-Ear Headphones                                |
+-----------------------------------------------------------------------------------+
|  +------------------------+   Wireless Over-Ear Headphones                        |
|  |                        |   Category: Electronics                               |
|  |                        |   -------------------------------------------------   |
|  |      [ Large Image ]   |   $99.99                                              |
|  |                        |   Availability: [ In Stock (15 available) ]           |
|  |                        |   -------------------------------------------------   |
|  +------------------------+   Description:                                        |
|                               Engineered with premium active noise cancellation,  |
|                               40mm neodymium drivers, and up to 30 hours playback.|
|                                                                                   |
|                               Quantity:  [ - ]  [ 2 ]  [ + ]                      |
|                               [ 🛒 Add 2 to Cart ]                                |
+-----------------------------------------------------------------------------------+
```

---

### 3.5 Shopping Cart (`Cart.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  Shopping Cart (2 items)                                                          |
+---------------------------------------------------+-------------------------------+
|  Item                               Qty    Total  |  Order Summary                |
|  +----------------------------------------------+ |  Subtotal (3 items):  $249.97 |
|  | [img] Wireless Headset    $99.99 [- 2 +] $199.98| |  Shipping:                  FREE |
|  |       [Remove]                               | |  Payment:                  COD    |
|  +----------------------------------------------+ |  ---------------------------  |
|  | [img] Studio Earbuds      $49.99 [- 1 +]  $49.99| |  Total:              $249.97 |
|  |       [Remove]                               | |                               |
|  +----------------------------------------------+ |  [ Proceed to Checkout ➔ ]    |
|  [ ⬅ Continue Shopping ]                          |  Guaranteed stock reservation |
+---------------------------------------------------+-------------------------------+
```

---

### 3.6 Checkout & COD Order Placement (`Checkout.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  Checkout                                                                         |
+---------------------------------------------------+-------------------------------+
|  1. Shipping Address                              |  Order Summary (2 items)      |
|                                                   |  - Wireless Headset x2: $199.98|
|  Full Name:      [ Jane Doe___________________ ]  |  - Studio Earbuds x1:   $49.99|
|  Phone Number:   [ +1 555-0123________________ ]  |  ---------------------------  |
|  Street Address: [ 123 Market Street, Apt 4B___ ]  |  Total to Pay (COD):  $249.97 |
|  City:           [ Springfield________________ ]  |                               |
|  Pincode:        [ 97477______________________ ]  |  Payment Method:              |
|                                                   |  (•) Cash on Delivery (COD)   |
|                                                   |                               |
|                                                   |  [ Place Order Now (COD) ]    |
+---------------------------------------------------+-------------------------------+
```

---

### 3.7 Customer Order History (`MyOrders.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  My Orders                                                                        |
+-----------------------------------------------------------------------------------+
|  Order #ORD-660C1A    Placed on: Oct 2, 2026    Status: [ 🟡 Pending ]  Total: $249.97
|  ---------------------------------------------------------------------------------
|  • Wireless Headset  x 2  - $99.99 each
|  • Studio Earbuds    x 1  - $49.99 each
|  Delivering to: Jane Doe, 123 Market Street, Springfield (97477)
+-----------------------------------------------------------------------------------+
```

---

### 3.8 Login & Register Cards (`Login.jsx`, `Register.jsx`)

```text
+---------------------------------------+
|  Sign In to Your Account              |
|  -----------------------------------  |
|  Email:                               |
|  [ admin@example.com________________] |
|                                       |
|  Password:                            |
|  [ ••••••••••_______________________] |
|                                       |
|  [ Sign In ➔ ]                        |
|                                       |
|  Demo Credentials:                    |
|  Admin: admin@example.com / admin123  |
|  Customer: john@example.com / cust123 |
|                                       |
|  Don't have an account? [ Register ]  |
+---------------------------------------+
```

---

## 4. Admin Panel Wireframes

### 4.1 Admin Shell Layout & Sidebar (`AdminLayout.jsx`)

```text
+--------------------+--------------------------------------------------------------+
| [ ⚙ Admin Center ] |  Admin Dashboard > Products                       [ Admin ▼] |
+--------------------+--------------------------------------------------------------+
| 📊 Dashboard       |  Catalog Products                            [ + Add Product]|
| 🏷 Categories      |  ----------------------------------------------------------- |
| 📦 Products        |  [Search...]   Filter: [All]                                 |
| 📑 Orders (12)     |  +-----+-------------------+------------+--------+-----+----+ |
| ------------------ |  | Img | Name              | Category   | Price  |Stock|Act | |
| 🌐 Storefront View |  | [x] | Wireless Headset  | Electronics| $99.99 | 15  |⚙ 🗑 | |
| 🚪 Sign Out        |  | [x] | Mechanical Keybd  | Electronics| $79.99 |  0  |⚙ 🗑 | |
+--------------------+--+-----+-------------------+------------+--------+-----+----+ |
```

---

### 4.2 Category Management (`AdminCategories.jsx`)
- **Table Columns**: Category Name, Description, Associated Products Count, Actions (Edit, Delete).
- **Add/Edit Modal**: Simple pop-up containing Name and Description inputs.
- **Delete Confirmation**: Warning dialog noting that deleting a category with attached products is blocked.

---

### 4.3 Product Management (`AdminProducts.jsx`)
- **Table Columns**: Image Preview, Product Name, Category Badge, Unit Price, Stock Count (with color coding: `Stock > 5` = green, `1 <= Stock <= 5` = amber, `Stock == 0` = red badge), Action buttons.
- **Product Modal**:
  - Name (text)
  - Description (textarea)
  - Price (number with decimals)
  - Stock (integer)
  - Category (dropdown populated from `/api/categories`)
  - Image URL (text input with live thumbnail preview)

---

### 4.4 Order Management & Status Transition (`AdminOrders.jsx`)

```text
+-----------------------------------------------------------------------------------+
|  Customer Orders                                                                  |
+----------+---------------+---------------------------+---------+-----------+------+
| Order ID | Customer      | Products Ordered          | Total   | Date      |Status|
+----------+---------------+---------------------------+---------+-----------+------+
| #660C1A  | Jane Doe      | Wireless Headset (x2)     | $249.97 | Oct 2, 26 |[v]   |
|          | jane@test.com | Studio Earbuds (x1)       |         |           |      |
+----------+---------------+---------------------------+---------+-----------+------+
```
- Status selector dropdown updates the order state on change (`Pending` ➔ `Confirmed` ➔ `Shipped` ➔ `Delivered` ➔ `Cancelled`) and triggers a notification toast.

---

## 5. Micro-Interactions & Feedback States

1. **Toast Notifications (`react-hot-toast`)**:
   - `toast.success("Added to cart!")`
   - `toast.error("Requested quantity exceeds available stock")`
   - `toast.success("Order placed successfully! Order ID: #1234")`
2. **Loading States**:
   - Buttons display an inline SVG spinner and enter `disabled` state during active HTTP submissions.
   - Catalog grid displays pulsed skeleton placeholder cards while fetching data.
3. **Empty States**:
   - Products page: "No products found matching your search. [Clear Filters]".
   - Cart page: "Your shopping cart is empty. [Explore Catalog]".
   - My Orders page: "You haven't placed any orders yet. [Start Shopping]".
4. **Delete Dialog Modal**:
   - Two-step confirmation: "Are you sure you want to delete this product? This action cannot be undone." with `Cancel` and `Delete Product` buttons.
