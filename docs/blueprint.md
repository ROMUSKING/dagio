# Expanded Action Plan: Building a Locally Hosted TigerBeetle/Rust/HTMX ERP

This document outlines a concrete, phased action plan to develop a modular, open-source Enterprise Resource Planning (ERP) system. The architecture is centered around **TigerBeetle** as the financial ledger, **Rust** for high-performance microservices, and **HTMX** for a dynamic, server-rendered user interface.

---

## Core Principles

- **Transactional Integrity:** All financial and inventory transactions will be processed by TigerBeetle, leveraging its double-entry accounting primitives for verifiable accuracy.
- **Performance & Safety:** Microservices will be built in Rust to ensure memory safety and high performance for all business logic.
- **Event-Driven Architecture:** Apache Kafka will serve as the central event bus, decoupling services and ensuring resilience.
- **Modern & Simple Frontend:** The user interface will be built with HTMX, allowing for dynamic, interactive experiences driven directly from the backend services, which will form a "universal API" by serving both data and HTML fragments.
- **Open & Collaborative:** The entire project will be developed as an open-source initiative to foster community involvement and innovation.

---

## Phase 1: Foundation Platform MVP (Months 1-6)

**Goal:** Establish the core infrastructure and build a functional proof-of-concept demonstrating the primary architectural pattern: a Rust service using TigerBeetle that serves an HTMX frontend.

**Team:** Single Developer

### Step 1: Local Infrastructure Setup (Weeks 1-3)

1. **Install Local Kubernetes:**
   - Set up a local Kubernetes cluster.
   - **Action:** `k3d cluster create erp-dev` to initialize a lightweight cluster.
2. **Install Tooling:**
   - Ensure `kubectl`, `helm`, and `docker` are installed and configured.
3. **Deploy Core Components via Helm:**
   - **PostgreSQL:** Deploy a standard PostgreSQL 15 instance. This will act as the "control plane" database for non-transactional metadata (e.g., product descriptions, user info).
     - **Action:** Use the Bitnami Helm chart: `helm install postgres bitnami/postgresql`.
   - **TigerBeetle:** Deploy a single-replica TigerBeetle 0.14 instance within a Kubernetes pod. For this phase, a simple Deployment manifest is sufficient.
     - **Action:** Create `infra/tigerbeetle/deployment.yaml` and apply it with `kubectl apply -f`.
4. **Initialize Git Repository:**
   - **Action:** Create a new public repository on GitHub.
   - **Structure:**
     ```
     /
     ├── .github/      # CI/CD workflows
     ├── docs/         # Architectural diagrams, user guides
     ├── infra/        # Kubernetes manifests, Helm charts
     └── services/     # Rust microservices
         └── inventory-service/
     ├── README.md
     └── LICENSE
     ```
   - **Action:** Create an initial `README.md` outlining the project vision, setup instructions, and a link to the `CONTRIBUTING.md`.

### Step 2: First Microservice & HTMX Integration (Weeks 4-12)

1. **Scaffold inventory-service in Rust:**
   - **Action:** `cargo new inventory-service` within the `/services` directory.
   - **Dependencies:** Add to `Cargo.toml`: `tokio`, `axum`, `tigerbeetle-rust`, `sqlx`, `maud` (for templating), `serde`, `dotenv`.
2. **Implement Control Plane & Data Plane Models:**
   - **PostgreSQL (Control Plane):** Define the schema for metadata.
     - **Action:** Create a SQL migration file for a `products` table (`id`, `sku`, `name`, `description`) and a `locations` table (`id`, `name`).
   - **TigerBeetle (Data Plane):** Implement the logic to create and manage TigerBeetle records.
     - **Accounts:** On startup, the service will read from PostgreSQL and create corresponding TigerBeetle Accounts for each SKU@Location combination.
       - `account.id`: A unique ULID.
       - `account.user_data_128`: Store the `product.id` (as bytes).
       - `account.user_data_64`: Store the `location.id` (as bytes).
       - `account.code`: 200 (Inventory Asset).
       - `account.ledger`: 10 (Operational Ledger).
     - **Transfers:** Create a function `move_stock(from_account_id, to_account_id, quantity, order_id)` that constructs and sends a Transfer to TigerBeetle.
3. **Develop the Universal API & HTMX Frontend:**
   - **API Endpoints (Axum Handlers):**
     - `GET /`: Renders a full HTML page with the main layout and an initial view of the inventory. This page must include `<script src="https://unpkg.com/htmx.org@1.9.10"></script>`.
     - `POST /inventory/move`: Accepts form data (`from_location`, `to_location`, `sku`, `quantity`). It calls the `move_stock` function. On success, it returns an HTML fragment representing the updated inventory list.
     - `GET /inventory/search?q=...`: Accepts a search query and returns an HTML fragment of the filtered inventory table body.
   - **Create HTMX UI (Maud templates):**
     - `inventory.html`: A table displaying inventory levels.
     - **Stock Move Form:** A simple form with inputs for SKU, quantity, and locations. The form will use `hx-post="/inventory/move"` and `hx-target="#inventory-table"` to update the table on submission.
     - **Search Box:** An input with `hx-get="/inventory/search"`, `hx-trigger="keyup changed delay:500ms"`, and `hx-target="#inventory-table"` for live search functionality.

### Step 3: Documentation & Open Source Prep (Weeks 13-24)

1. **Document Setup:** Create a detailed `docs/setup.md` with step-by-step instructions.
2. **Architectural Diagrams:** Use Mermaid in `docs/architecture.md` to show the flow: `User -> HTMX -> Axum Service -> TigerBeetle/Postgres`.
3. **Prepare for Community:**
   - Finalize `CONTRIBUTING.md` (with coding standards and PR process) and `CODE_OF_CONDUCT.md`.
   - Create GitHub issue templates for bug reports and feature requests.
   - Populate the backlog with well-defined "good first issues" (e.g., "Add flash messages for successful stock moves," "Improve error handling for invalid SKU").

---

## Phase 2: Ecosystem Expansion (Months 7-12)

**Goal:** Expand platform capabilities by adding more services and integrating Kafka, while formally opening the project to community contributors.

**Team:** Solo Developer + First Contributors

### Step 1: Go Public & Integrate Kafka (Months 7-9)

1. **Publish & Announce:** Announce the public repository on relevant forums (e.g., Rust subreddit, Hacker News).
2. **Establish Communication:** Create and publicize a Discord server for community discussion.
3. **Deploy Kafka:** Deploy Apache Kafka on Kubernetes using the Strimzi operator Helm chart.
4. **Integrate Kafka into inventory-service:**
   - Add the `rdkafka` dependency.
   - Define an `InventoryMoved` event schema (e.g., as a Rust struct serialized to JSON).
   - After a successful `create_transfers` call to TigerBeetle, the service will publish the `InventoryMoved` event to an `inventory.events` Kafka topic. This ensures that the event is only broadcast after the state change is committed to the ledger.

### Step 2: Develop New Services & Universal API (Months 10-12)

1. **Develop order-service:**
   - Create a new Rust service to manage sales orders.
   - **Data Model:**
     - **PostgreSQL:** `orders` and `order_items` tables.
     - **TigerBeetle:** Create accounts for `AccountsReceivable` (per customer), `SalesRevenue`, and `CostOfGoodsSold`.
   - **Logic:**
     - It will consume `inventory.events` to maintain a local, eventually consistent cache of stock levels.
     - When an order is fulfilled, it will create a linked set of TigerBeetle transfers (debiting COGS, crediting Inventory; debiting AccountsReceivable, crediting SalesRevenue).
     - It will publish an `OrderFulfilled` event to a `sales.events` topic.
   - **UI:** Expose its own HTMX-driven UI components for creating and viewing orders.
2. **Refine the Universal API:**
   - **Content Negotiation:** Implement proper content negotiation in all services. Based on the `Accept` header (`application/json` vs. `text/html`), endpoints will return either JSON data or an HTML fragment.
   - **API Documentation:** Use a tool like OpenAPI to document the JSON responses for programmatic users.

---

## Phase 3: Full ERP Build-out (Months 13-24)

**Goal:** Mature the platform into a comprehensive ERP system with a self-sustaining open-source community.

**Team:** Core Maintainers + Active Community

### Step 1: Scale Development & Features (Months 13-18)

1. **Complete ERP Modules:** Coordinate community efforts to build out:
   - **Purchasing:** Manage purchase orders, suppliers, and goods receipts.
   - **Manufacturing:** Handle bills of materials (BOMs), work orders, and production reporting.
   - **Finance:** General ledger, accounts payable, financial reporting.
2. **Implement Wasm Plugin System:**
   - Integrate Extism into the order-service.
   - **Use Case:** Allow users to upload a Wasm plugin that implements a custom pricing strategy. The order-service will execute the plugin to calculate the final price when an order is created.
3. **Develop a Unified Frontend:**
   - Create a minimal "shell" application (e.g., using Axum with a base template).
   - This shell will provide the main navigation and dashboard structure. The content for each section will be loaded dynamically from the respective microservices using HTMX's `hx-boost` for navigation and composing fragments.

### Step 2: Harden the Platform & Formalize Governance (Months 19-24)

1. **Security & Performance:**
   - Implement centralized authentication using Keycloak. Services will validate JWTs passed from the frontend shell.
   - Use `cargo-audit` in CI to scan for vulnerabilities.
   - Define Service Level Objectives (SLOs) for API latency and uptime, and set up monitoring with Prometheus and Grafana.
2. **Establish Governance:**
   - Create a formal `GOVERNANCE.md` defining a Technical Steering Committee (TSC) and the process for becoming a maintainer.
   - Adopt an Architecture Decision Record (ADR) process for proposing and documenting significant technical changes.
3. **Automate Releases:**
   - Set up a full CI/CD pipeline in GitHub Actions that, on a git tag, will:
     - Run `cargo test` and `cargo clippy -- -D warnings`.
     - Build optimized, multi-stage Docker images for each service.
     - Push images to a container registry (e.g., GitHub Container Registry).
     - Package and publish Helm charts for easy deployment.