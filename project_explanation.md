# ShopHub E-Commerce Project Structure Explanation

The project is built around a modern **Microservices Architecture** where different components of the system are decoupled into standalone containers. This design makes the platform highly scalable, robust, and perfect for DevOps practices.

Here is a detailed breakdown of the complete project structure:

## 1. 🗂️ The `services/` Directory (Backend)
This is the core logic of the application, split into five independent microservices based on their domain. Each service contains its own `Dockerfile` and operates somewhat autonomously.
* **`user-service/`** *(Node.js & Express)*: Manages user authentication (Registration, Login), JWT token generation, and user profiles. Connects to the `users_db` PostgreSQL database.
* **`product-service/`** *(Python & FastAPI)*: Manages the product catalog, categories, product searches, and stock. Connects to the `products_db` PostgreSQL database.
* **`cart-service/`** *(Node.js & Express)*: A high-performance service that manages user shopping carts. To make cart additions/updates blazing fast, it connects to an in-memory **Redis** cache instead of a traditional relational database.
* **`order-service/`** *(Node.js & Express)*: Handles the checkout process, tracking order histories, and updating order status. Connects to the `orders_db` PostgreSQL database.
* **`payment-service/`** *(Python & FastAPI)*: A mock payment gateway that simulates verifying credit cards (approving valid cards and failing cards ending in `0000`).

## 2. 🖥️ The `frontend/` Directory (Web UI)
* A modern React application built with **Next.js 14**.
* It includes pages like home, product details, auth, cart, and checkout (`app/` directory).
* It communicates directly with the backend microservices by making HTTP requests (defined in `lib/api.ts`).

## 3. 🚪 The `nginx/` Directory (API Gateway)
* Acts as the main entryway (Reverse Proxy) for the entire platform, listening on port `80`.
* The `nginx.conf` acts essentially as a traffic cop. When a user requests `/api/products`, Nginx forwards that traffic to the hidden `product-service` port. If a user requests a web page, it forwards it to the Next.js `frontend` container.
* This is a crucial DevOps component because it means you only need to expose a single port (`80`) to the public internet while keeping all your sensitive backend microservices completely isolated in a private docker network.

## 4. 🗄️ The `db/` Directory (Infrastructure)
* Contains the database initialization scripts like `init.sql`.
* Since microservices require their own isolated databases, this script automatically provisions the `users_db`, `products_db`, and `orders_db` inside the PostgreSQL container on startup. It also seeds example products and an admin user.

## 5. 🐳 Orchestration Files (`docker-compose.yml`)
* **`docker-compose.yml`**: Defines how all these puzzle pieces fit together. It maps out the containers, networks (`ecommerce-network`), persistent volumes for the databases (to prevent data loss when containers stop), environment variables mapping, and dependency start orders (e.g., waiting for Postgres to be healthy before starting the services).

---
**Why this matters for DevOps:**
This is a highly scalable structure. Because every service is containerized separately, a DevOps engineer can easily deploy this architecture to Kubernetes, scale up just the `product-service` if traffic surges, or isolate failures without bringing down the whole platform!
