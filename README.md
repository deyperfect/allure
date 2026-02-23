# Allure

A modern e-commerce shopping app built with Vue 3. Supports separate user and admin experiences, full cart management, and order checkout.

🛍️ **Live demo:** [allure-neon.vercel.app](https://allure-neon.vercel.app)

---

## Features

**Customers**
- Register and log in with JWT-based authentication
- Browse the active product catalog
- View individual product details
- Add items to cart, update quantities, and remove items
- See per-item subtotals and cart total
- Checkout and place orders
- View order confirmation and order history
- View and manage profile details

**Admins**
- Access a dedicated admin dashboard
- Create, update, and deactivate/reactivate products
- View all orders across all users

**General**
- Dynamic navbar (shows Register/Login for guests, Logout for authenticated users)
- Light / Dark mode toggle
- Toast notifications via Notyf
- Role-based route access (admins cannot add to cart; guests cannot access protected pages)
- Single-page app with client-side routing via Vue Router

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Vue 3 |
| Build Tool | Vite |
| State Management | Pinia |
| HTTP Client | Axios |
| Styling | Bootstrap 5 + Bootstrap Icons |
| Notifications | Notyf |
| Deployment | Vercel |

---

## Getting Started

### Prerequisites

- Node.js v18 or higher
- npm
- A running backend API (configure the URL in your `.env` file)

### Installation

```bash
# Clone the repository
git clone https://github.com/deyperfect/allure.git
cd allure

# Install dependencies
npm install
```

### Environment Variables

Create a `.env.local` file in the project root for local development:

```env
VITE_JOB_TRACKER_API=http://localhost:4000
```

For production, update `.env.production` with your hosted API URL.

### Running the App

```bash
# Start the development server
npm run dev
```

The app will be available at `http://localhost:5173`.

### Build for Production

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

---

## Project Structure

```
src/
├── api.js                  # Axios instance with auth interceptor
├── main.js                 # App entry point
├── App.vue                 # Root component
├── stores/
│   └── global.js           # Pinia store (user state, cart count)
├── components/
│   ├── NavbarComponent.vue
│   ├── ThemeToggle.vue
│   ├── EditProductButton.vue
│   └── ToggleProductStatusButton.vue
└── pages/
    ├── RegisterPage.vue
    ├── LoginPage.vue
    ├── ProductsPage.vue
    ├── ProductDetailsPage.vue
    ├── CartViewPage.vue
    ├── CheckoutPage.vue
    ├── OrderConfirmationPage.vue
    ├── AdminDashboardPage.vue
    ├── AddProductPage.vue
    ├── EditProductPage.vue
    └── ProfilePage.vue
```

---

## Deployment

This project is deployed via [Vercel](https://vercel.com/). Connect the repository to Vercel and it will automatically build and deploy on every push to `main`. Make sure to set the `VITE_JOB_TRACKER_API` environment variable in your Vercel project settings.

---

## Authors

- **Edryl Palinis**
- **Norman Chingcuanco**