# TVU Books Shop 📚

> **Thiruvalluvar University Academic Textbook & Study Material Marketplace**
> A lightweight, responsive web app for buying and selling university textbooks and study materials, built with **HTML5, Vanilla CSS, Vanilla JavaScript, Firebase (Auth + Firestore), Cloudinary, and EmailJS**.

---

## 📑 Table of Contents

1. [Features](#-features)
2. [Tech Stack](#-tech-stack)
3. [Project Structure](#-project-structure)
4. [Setup Guide](#-setup-guide)
5. [Run Locally](#-run-locally)
6. [Deploy (GitHub + Vercel)](#-deploy-github--vercel)
7. [Feature Checklist](#-feature-checklist)

---

## 🏛️ Features

- **University branding**: Official Thiruvalluvar University emblem and academic colour styling.
- **Google Sign-In**: One-click login; a customer profile is created automatically in `customers/{uid}`.
- **Seller Hub**: Separate seller onboarding (`sellers/{uid}`) without affecting the customer profile.
- **Dynamic book inventory**
  - Cover images uploaded from the browser to **Cloudinary** (URL saved in Firestore).
  - Automatic discount calculation: `((MRP - Discounted) / MRP) * 100`.
  - Edit price, stock, notes, or replace the cover image at any time.
- **Student discovery**
  - Live search by title, author, or department.
  - Filter by department (Engineering, Computer Science, Mathematics, Science, Commerce, Arts, Study Materials, etc.).
  - Sort by price (low → high, high → low) or newest.
- **Wishlist**: Private saved books in `users/{uid}/wishlist/{bookId}`.
- **Cash on Delivery checkout**: Instant order creation in `orders/{orderId}` with campus handover details.
- **Order celebration**: Lightweight canvas confetti and a checkmark pop-up on success.
- **Seller email alerts**: EmailJS sends the seller the buyer's contact and delivery address.
- **Home page butterfly**: A subtle CSS/JS butterfly that flutters and perches on UI elements (respects `prefers-reduced-motion`).
- **Fully responsive**: Mobile, tablet, and desktop layouts.

---

## 🧰 Tech Stack

| Area | Technology |
| :--- | :--- |
| Frontend | HTML5, Vanilla CSS, Vanilla JavaScript |
| Authentication | Firebase Authentication (Google) |
| Database | Cloud Firestore |
| Image storage / CDN | Cloudinary (unsigned uploads) |
| Email notifications | EmailJS |
| Hosting | Vercel |

---

## 📂 Project Structure

```
tvu-books-materials/
├── index.html            # Home page: hero, dynamic catalog, butterfly
├── books.html            # Full catalog with search, filter, sort
├── book-details.html     # Single book view, seller info, buy modal
├── wishlist.html         # Saved items
├── orders.html           # Customer order history
├── seller.html           # Seller registration & onboarding
├── seller-books.html     # Seller inventory, add/edit modal
├── seller-orders.html    # Seller incoming orders & COD collection
│
├── css/
│   ├── style.css         # Design tokens, components, modals
│   └── responsive.css    # Mobile drawer & breakpoints
│
├── js/
│   ├── firebase-config.js      # Firebase initialization & keys
│   ├── cloudinary-service.js   # Cloudinary image upload config
│   ├── auth.js                 # Google auth state & dynamic header
│   ├── email-service.js        # EmailJS order dispatch
│   ├── notifications.js        # Toasts, modals, formatters, confetti
│   ├── butterfly.js            # Animated butterfly (home page)
│   ├── home.js                 # Home catalog controller
│   ├── books.js                # Catalog filter & search
│   ├── book-details.js         # Single book view
│   ├── wishlist.js             # Wishlist controller
│   ├── orders.js               # Customer orders controller
│   ├── seller.js               # Seller onboarding
│   ├── seller-books.js         # Seller inventory & image upload
│   └── seller-orders.js        # Seller incoming orders
│
├── assets/
│   └── tvu-logo.png      # Thiruvalluvar University crest
│
├── firestore.rules       # Cloud Firestore security rules
└── README.md
```

---

## 🚀 Setup Guide

### 1. Create the Firebase project
1. Open the [Firebase Console](https://console.firebase.google.com/).
2. Click **Add project**, name it `tvu-books-materials`, and finish setup.

### 2. Enable Google Authentication
1. Go to **Build → Authentication → Get started → Sign-in method**.
2. Enable **Google**, choose a support email, and **Save**.
3. Under **Settings → Authorized domains**, add `localhost` and your Vercel domain (e.g. `tvu-books-materials.vercel.app`).

### 3. Set up Cloud Firestore
1. Go to **Build → Firestore Database → Create database**.
2. Choose a nearby region (e.g. `asia-south1` / Mumbai) and start in **Production mode**.
3. Open the **Rules** tab, paste the contents of `firestore.rules`, and click **Publish**.

#### Firestore collections

| Collection | Fields |
| :--- | :--- |
| `customers/{uid}` | `uid`, `name`, `email`, `profileImage`, `role: "customer"`, `createdAt`, `updatedAt` |
| `sellers/{uid}` | `uid`, `name`, `email`, `phone`, `university`, `department`, `description`, `profileImage`, `role: "seller"`, `createdAt`, `updatedAt` |
| `books/{bookId}` | `bookId`, `title`, `author`, `category`, `description`, `notes`, `sellerId`, `sellerName`, `sellerEmail`, `originalPrice`, `discountedPrice`, `discountPercentage`, `imageUrl` (Cloudinary HTTPS URL), `stock`, `availability` (`"In Stock"` / `"Out of Stock"`), `createdAt`, `updatedAt` |
| `users/{uid}/wishlist/{bookId}` | `bookId`, `title`, `author`, `category`, `discountedPrice`, `originalPrice`, `imageUrl`, `sellerName`, `addedAt` |
| `orders/{orderId}` | `orderId` (e.g. `TVU-XXXXX`), `customerId`, `customerName`, `customerEmail`, `customerPhone`, `customerAddress`, `sellerId`, `sellerName`, `sellerEmail`, `bookId`, `bookTitle`, `bookImageUrl`, `quantity`, `originalPrice`, `discountedPrice`, `totalAmount`, `paymentMethod: "Cash on Delivery"`, `orderStatus` (`"Confirmed"` / `"Delivered"`), `createdAt`, `updatedAt` |

### 4. Set up Cloudinary (free image storage)
1. Create a free account at [cloudinary.com](https://cloudinary.com/).
2. Copy your **Cloud name** from the dashboard.
3. Go to **Settings → Upload → Upload presets → Add upload preset**.
4. Set **Signing mode** to **Unsigned** (required for browser uploads), optionally set **Folder** to `tvu_books`, then **Save**.
5. Open `js/cloudinary-service.js` and fill in:

```javascript
const CLOUDINARY_CONFIG = {
  cloudName: "your_cloud_name",
  uploadPreset: "your_unsigned_upload_preset",
  folder: "tvu_books"
};
```

> ⚠️ Never put your Cloudinary **API Secret** in frontend code.

### 5. Set up EmailJS (seller order notifications)
1. Sign up at [emailjs.com](https://www.emailjs.com/).
2. Create an **Email Service** (e.g. Gmail) and note the `Service ID`.
3. Create an **Email Template** with the subject `New Book Order - TVU Books & Materials` and these variables:
   `{{to_name}}`, `{{to_email}}`, `{{order_id}}`, `{{customer_name}}`, `{{customer_phone}}`, `{{delivery_address}}`, `{{book_title}}`, `{{quantity}}`, `{{total_amount}}`, `{{payment_method}}`, `{{order_date}}`
4. Copy the `Template ID` and your `Public Key` (Account → API Keys).
5. Open `js/email-service.js` and fill in:

```javascript
const EMAILJS_CONFIG = {
  serviceId: "your_service_id",
  templateId: "your_template_id",
  publicKey: "your_public_key"
};
```

### 6. Connect Firebase in the code
Open `js/firebase-config.js` and paste your web app credentials (Firebase Console → Project settings → Your apps):

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "tvu-books-materials.firebaseapp.com",
  projectId: "tvu-books-materials",
  storageBucket: "tvu-books-materials.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef..."
};
```

---

## 💻 Run Locally

Firebase Auth needs an `http://` or `https://` origin (not `file:///`), so use a local server. Pick one:

**VS Code Live Server**: right-click `index.html` → **Open with Live Server** (`http://127.0.0.1:5500`).

**Python**
```bash
python -m http.server 8000
```
Then open `http://localhost:8000`.

**Node**
```bash
npx serve .
```

---

## 🌐 Deploy (GitHub + Vercel)

### 1. Push to GitHub
```bash
git init
git add .
git commit -m "Initial commit: TVU Books & Materials"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/tvu-books-materials.git
git push -u origin main
```

### 2. Deploy on Vercel
1. Go to [vercel.com](https://vercel.com/) → **Add New → Project**.
2. Import the `tvu-books-materials` repository.
3. Keep the defaults (Framework Preset: **Other**, Root Directory: `./`) and click **Deploy**.
4. Copy the production domain (e.g. `tvu-books-materials.vercel.app`).
5. **Important:** add that domain in Firebase Console → Authentication → Settings → **Authorized domains**, otherwise Google Sign-In will fail on the live site.

New book uploads never need a code change or redeploy; everything is stored in Firestore and Cloudinary.

---

## ✅ Feature Checklist

- [x] Official TVU logo in header
- [x] Google Sign-In with separate `customers/{uid}` profile
- [x] Seller registration with separate `sellers/{uid}` profile
- [x] Book cover upload to Cloudinary, URL saved in `books/{bookId}`
- [x] Automatic discount percentage (e.g. `30% OFF`)
- [x] Live catalog with category chips
- [x] Search by title, author, and category
- [x] Book details page with study notes
- [x] Customer wishlist
- [x] Cash on Delivery only
- [x] Confetti and checkmark on order success
- [x] EmailJS seller notification on new orders
- [x] Customer "My Orders" and seller "Incoming Orders" views
- [x] Seller "My Listed Books" (add, edit, replace image, delete)
- [x] Animated butterfly with reduced-motion support
- [x] Firestore security rules
- [x] Responsive on mobile, tablet, and desktop

---

© 2026 TVU Books Shop • Thiruvalluvar University
