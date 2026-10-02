# 🧪 End-to-End Demo Flow & Verification Guide

This document provides a step-by-step testing script and verification manual for executing the complete 11-step live demonstration of the **Mini E-Commerce Demo Project**, along with edge-case test matrices.

---

## 📑 Table of Contents

- [1. Test Credentials & Pre-requisites](#1-test-credentials--pre-requisites)
- [2. The 11-Step Master Demo Workflow](#2-the-11-step-master-demo-workflow)
  - [Step 1: Admin Authentication](#step-1-admin-authentication)
  - [Step 2: Admin Adds Category](#step-2-admin-adds-category)
  - [Step 3: Admin Adds New Product](#step-3-admin-adds-new-product)
  - [Step 4: Real-time Storefront Verification](#step-4-real-time-storefront-verification)
  - [Step 5: Customer Registration & Login](#step-5-customer-registration--login)
  - [Step 6: Catalog Browsing & Search](#step-6-catalog-browsing--search)
  - [Step 7: Category Filtering](#step-7-category-filtering)
  - [Step 8: Cart Management & Stock Limit](#step-8-cart-management--stock-limit)
  - [Step 9: Checkout with Cash on Delivery](#step-9-checkout-with-cash-on-delivery)
  - [Step 10: Order Placement & Automatic Stock Decrement](#step-10-order-placement--automatic-stock-decrement)
  - [Step 11: Admin Order Review & Status Update](#step-11-admin-order-review--status-update)
- [3. Edge Cases & Security Test Matrix](#3-edge-cases--security-test-matrix)

---

## 1. Test Credentials & Pre-requisites

The database seeder (`npm run seed` in `/server`) provisions the following testing accounts:

| Role | Email | Password | Initial State |
| :--- | :--- | :--- | :--- |
| **System Admin** | `admin@example.com` | `admin123` | Pre-created with `role: 'admin'` |
| **Test Customer** | `john@example.com` | `customer123` | Pre-created with `role: 'customer'` |

---

## 2. The 11-Step Master Demo Workflow

```text
Admin Login ➔ Add Category ➔ Add Product ➔ Visible on Website ➔ Customer Register/Login
     ➔ Browse Products ➔ Filter by Category ➔ Add to Cart ➔ Checkout (COD)
     ➔ Place Order (Stock -1) ➔ Admin Sees Order & Updates Status
```

### Step 1: Admin Authentication
1. Navigate to `http://localhost:5173/login`.
2. Enter email `admin@example.com` and password `admin123`.
3. Click **Sign In**.
4. **Expected Result**: Success toast displays, user is redirected to `/admin` dashboard showing administrative metrics and sidebar.

---

### Step 2: Admin Adds Category
1. In the Admin Sidebar, click **Categories** (`/admin/categories`).
2. Click **+ Add Category**.
3. In the modal, enter:
   - **Name**: `Smart Audio`
   - **Description**: `Wireless earbuds, studio monitors, and soundbars`
4. Click **Save Category**.
5. **Expected Result**: Modal closes, success toast appears, and `Smart Audio` displays in the category table with 0 products.

---

### Step 3: Admin Adds New Product
1. In the Admin Sidebar, click **Products** (`/admin/products`).
2. Click **+ Add Product**.
3. In the modal, fill in:
   - **Name**: `Studio Pro Wireless Headphones`
   - **Category**: Select `Smart Audio`
   - **Price ($)**: `149.99`
   - **Stock**: `5` (set to 5 to observe stock decrement)
   - **Image URL**: `https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=600`
   - **Description**: `Studio-grade high fidelity noise cancelling headphones.`
4. Click **Create Product**.
5. **Expected Result**: Product is saved, table updates immediately showing stock count `5` in green/amber badge.

---

### Step 4: Real-time Storefront Verification
1. Click **Storefront View** in the sidebar or open a new browser tab at `http://localhost:5173/products`.
2. Notice the new category pill `Smart Audio` is available.
3. Observe `Studio Pro Wireless Headphones` is visible in the product grid with price `$149.99` and badge `In Stock (5)`.

---

### Step 5: Customer Registration & Login
1. Click **Sign Out** or use an incognito window.
2. Navigate to `http://localhost:5173/register`.
3. Register a new customer:
   - **Name**: `Alice Walker`
   - **Email**: `alice@example.com`
   - **Password**: `password123`
   - **Confirm Password**: `password123`
4. Click **Create Account**.
5. **Expected Result**: Account is created, user is automatically logged in as customer, navbar displays `Alice Walker`.

---

### Step 6: Catalog Browsing & Search
1. On the Products page (`/products`), locate the search bar.
2. Type `Studio` in the search input.
3. **Expected Result**: Catalog filters in real time to show only `Studio Pro Wireless Headphones`.

---

### Step 7: Category Filtering
1. Clear the search input.
2. Click the `Smart Audio` category pill.
3. **Expected Result**: URL updates to `?category=Smart+Audio`, and only items matching the `Smart Audio` category are shown.

---

### Step 8: Cart Management & Stock Limit
1. Click on the product card to navigate to `/products/:id`.
2. On the details page, verify the stock badge displays `5 available`.
3. Try clicking `+` past 5 units.
4. **Expected Result**: Quantity selector caps at `5` and prevents exceeding available stock.
5. Set quantity to `2` and click **Add to Cart**.
6. **Expected Result**: Success toast: `"Added 2 Studio Pro Wireless Headphones to cart"`, navbar cart badge updates to `(2)`.

---

### Step 9: Checkout with Cash on Delivery
1. Click the Cart icon or navigate to `/cart`.
2. Verify:
   - Item: `Studio Pro Wireless Headphones`
   - Unit Price: `$149.99`
   - Quantity: `2`
   - Total: `$299.98`
3. Click **Proceed to Checkout**.
4. On `/checkout`, fill in the shipping details:
   - **Full Name**: `Alice Walker`
   - **Phone**: `+1 555-987-6543`
   - **Address**: `742 Evergreen Terrace`
   - **City**: `Springfield`
   - **Pincode**: `97477`
5. Note payment method is fixed to **Cash on Delivery (COD)**.

---

### Step 10: Order Placement & Automatic Stock Decrement
1. Click **Place Order Now (COD)**.
2. **Backend Execution**:
   - Backend looks up `Studio Pro Wireless Headphones` in MongoDB.
   - Verifies server price `$149.99` * 2 = `$299.98`.
   - Checks stock: `5 >= 2` (Valid).
   - Decrements stock from `5` to `3`.
   - Creates Order document.
3. **Frontend Response**:
   - Cart is cleared (`cartCount` becomes `0`).
   - Redirected to `/my-orders`.
   - New order appears with status `Pending` and total `$299.98`.
4. Return to `/products/:id`:
   - Stock counter now displays `In Stock (3 available)`.

---

### Step 11: Admin Order Review & Status Update
1. Sign in as Admin (`admin@example.com` / `admin123`).
2. In the Admin Dashboard, click **Orders** (`/admin/orders`).
3. Locate Alice Walker's new order `#...` with status `Pending`.
4. Click the status dropdown and select **Confirmed**.
   - Status updates; toast confirms change.
5. Next, select **Shipped**.
   - Status updates to `Shipped`.
6. Finally, when package arrives, select **Delivered**.
   - Status badge turns green `Delivered`.
7. Sign back in as Alice Walker (`alice@example.com`) and open `/my-orders`:
   - Verify Alice's order status reflects `Delivered`.

---

## 3. Edge Cases & Security Test Matrix

| Test Scenario | Steps | Expected Result |
| :--- | :--- | :--- |
| **Exceeding Stock in Cart** | Set quantity higher than available stock on product page or in cart | Quantity button disabled / capped at current stock; toast alert shown |
| **Concurrent Stock Depletion** | Customer A and B both have 1 unit in cart. A places order first; B tries to place order | Server returns `400 Bad Request: Insufficient stock`. Customer B's order is blocked |
| **Price Tampering Prevention** | Use Postman/cURL to submit `price: 1.00` in order body | Server ignores submitted price, fetches authoritative price from MongoDB, and bills true amount |
| **Privilege Escalation Guard** | Customer attempts to access `/admin` or issue `POST /api/products` | UI redirects away; backend returns `403 Forbidden` |
| **Duplicate Email Registration** | Register second account with `alice@example.com` | Backend returns `400 Bad Request: Email already registered` |
| **Delete Category with Products** | Attempt to delete category in admin panel while products are attached | System warns that category contains products and safely prevents orphaned products |
| **Negative Stock or Price** | Admin enters `-10` stock or `-5` price in product creation modal | Form validation blocks submission; backend schema `min: 0` rejects |
