# Simple Point of Sale (POS) Frontend

A modern Point of Sale (POS) frontend application built with Vue 3, TypeScript, Vite, Pinia, and TailwindCSS. This project is developed following the course:  
[Full-Stack Laravel 12, Vue 3, TypeScript: Bikin Aplikasi POS Sederhana](https://buildwithangga.com/kelas/laravel-12-vue-3-typescript-bikin-aplikasi-pos-sederhana)

## Features

- User Authentication (Login)
- Admin Dashboard
- Product Management (Create, Edit, Delete, View)
- Product Categories Management
- Customer Management
- Point of Sale (POS) Interface
- Transaction History
- Responsive UI with TailwindCSS & PrimeVue

## Project Structure

```
src/
  api/             # API services (Axios instances, endpoints)
  assets/          # Static assets (images, fonts, etc.)
  components/      # Reusable UI components
  layout/          # Main layouts (App, Auth, Dashboard)
  pages/           # Main application pages (Views)
  router/          # Routing configuration
  stores/          # State management (Pinia)
  types/           # TypeScript interfaces and types
```

## Getting Started

1. Clone this repository
2. Install dependencies:

    ```sh
    npm install
    ```

3. Start the development server:

    ```sh
    npm run dev
    ```

4. Build for production:

    ```sh
    npm run build
    ```

## Tech Stack

- [Vue 3](https://vuejs.org/)
- [TypeScript](https://www.typescriptlang.org/)
- [Vite](https://vitejs.dev/)
- [Pinia](https://pinia.vuejs.org/)
- [Vue Router](https://router.vuejs.org/)
- [TailwindCSS](https://tailwindcss.com/)
- [PrimeVue](https://primevue.org/)
- [Axios](https://axios-http.com/)
