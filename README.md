# Lusso Diseño

A production-grade luxury home décor storefront. React + Vite + Tailwind CSS + Firebase (Firestore + Auth).

## 1. Install & run

```bash
npm install
npm run dev
```

Opens at `http://localhost:5173`.

## 2. Firebase setup

Firebase is already configured in `src/firebase/firebase.js` with the project credentials you provided
(`lusso-diseno`). Two things to do in the Firebase Console before the site is fully live:

### a) Firestore rules

Go to **Firestore Database → Rules** and paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /products/{productId} {
      allow read: if true;
      allow write: if request.auth != null;
    }

    match /orders/{orderId} {
      allow create: if true;
      allow read, update, delete: if request.auth != null;
    }
  }
}
```

Full explanation and a dev-only "wide open" fallback rule set is documented directly in
`src/firebase/firebase.js`.

### b) Create your admin login

Go to **Authentication → Users → Add user**, create an email/password login. That's what you'll use to
sign into the hidden admin panel.

## 3. Accessing the admin panel

There is **no visible admin button anywhere in the UI**. To open it:

> Click the "LUSSO" wordmark in the header **10 times quickly** (within ~1.5 seconds total).

This navigates to `/control-9f2x`, a route that is never linked from any page and renders its own
full-screen login gate (Firebase Auth). From there you get:

- **Dashboard** — live order count, revenue, product count, pending orders
- **Products** — create / edit / delete products (writes to the `products` collection)
- **Orders** — real-time list of every order submitted through checkout

If you want a different secret path or click count, edit `src/hooks/useSecretTrigger.js` (click count/
window) and the route string in `src/components/Header.jsx` + `src/App.jsx` (`/control-9f2x`).

## 4. Firestore data model

**`products`**
```
{
  title: string,
  price: number,
  category: string,
  finish: string,
  description: string,
  image: string (URL),
  createdAt: serverTimestamp
}
```

**`orders`**
```
{
  name, email, phone, address, notes: string,
  items: [{ productId, title, price, quantity, finish }],
  total: number,
  status: 'pending' | 'confirmed' | 'shipped' | ...,
  createdAt: serverTimestamp
}
```

## 5. Project structure

```
src/
  admin/           Admin panel (login, dashboard, products CMS, orders)
  components/      Shared UI: Header, Footer, ProductCard, UI atoms
  context/         CartContext (sessionStorage-persisted cart)
  firebase/        firebase.js (init), products.js, orders.js (Firestore helpers)
  hooks/           useProducts, useAdminAuth, useSecretTrigger
  pages/           Home, Shop, ProductDetail, Cart, Checkout, Journal, NotFound
  utils/           formatINR
```

## 6. Adding your first products

Once signed into the admin panel, use **Products → Add Product** to create real catalog entries —
the Home and Shop pages are wired to Firestore live via `onSnapshot`, so new products appear instantly
without a redeploy.
