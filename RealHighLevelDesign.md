# CodePop – High-Level Design Document (Sprint 1 – Draft 1)


## 1. Introduction

### Purpose

This High-Level Design document describes the architecture and major system components of the CodePop 2026 platform. CodePop is an AI-driven soda and float ordering system designed to scale across multiple store locations while operating without a centralized global server.

The purpose of this document is to define the overall distributed system structure, architectural decisions, and component responsibilities. It provides a shared technical reference for developers, stakeholders, and instructors so that system components can be implemented independently while remaining compatible within a regionally coordinated architecture.

This document focuses on high-level structure and architectural decisions rather than low-level implementation details.

---

### Scope

This design covers the core CodePop platform, including:

- Distributed client–server architecture  
- Multi-store nationwide support without a centralized server  
- Independent store operation with regional synchronization  
- Role-based dashboards and access control  
- AI-assisted drink recommendations and operational analytics  
- Inventory coordination and machine maintenance tracking  
- Supply hub and regional logistics coordination  
- Payment processing integration  
- Containerized deployment using Docker  
- Regional hosting infrastructure using Google Cloud Platform (GCP)  

Low-level implementation details (specific libraries, internal algorithms, and deployment scripts) are intentionally excluded and will be addressed in the Low-Level Design document.

---

### Audience

This document is intended for:

- Software developers and system architects  
- Project stakeholders and instructors  
- Team members responsible for AI, logistics, or operational modules  

---

## 2. System Overview

### Problem Statement

Dirty soda shops often present customers with overwhelming customization options and long wait times, leading to a stressful ordering experience. As CodePop expands nationwide, managing inventory, machine maintenance, supply coordination, and customer experience across multiple independent store locations introduces significant architectural challenges.

Traditional centralized systems create single points of failure and reduce fault tolerance. Additionally, manual coordination of inventory and maintenance does not scale efficiently across regions.

The system must:

- Support multiple store locations  
- Operate without a centralized global server  
- Allow each store to function independently  
- Support regional synchronization of inventory, supply, and maintenance data  
- Enforce strict role-based access control  

---

### Proposed Solution

CodePop provides a distributed, AI-powered ordering platform built on a regional client–server architecture.

Customers interact with the system through a web-based interface, where they can:

- Customize drinks at their own pace  
- Receive intelligent drink recommendations  
- Schedule pickup times or use geolocation-based preparation  
- Track order status in real time  

Each store operates as an independent system node with its own backend services and database. Stores communicate securely within their assigned region and coordinate with regional supply hubs.

The system supports clearly defined user roles:

- `super_admin` – system-wide oversight  
- `admin` – store-level administration  
- `manager` – store-level operations  
- `logistics_manager` – regional supply coordination  
- `repair_staff` – machine maintenance management  
- `account_user` – registered customers  
- `general_user` – guest users  

Operational automation includes:

- AI-assisted drink recommendations  
- Inventory tracking and restock forecasting  
- Maintenance scheduling optimization  
- Regional supply routing coordination  
- Role-specific dashboards  

By distributing system logic across store nodes and enabling regional synchronization, CodePop ensures scalability, fault tolerance, and operational resilience.

---

### System Constraints

- Must support multiple store locations across the United States  
- Must not rely on a centralized global server  
- Each store must operate independently if disconnected  
- Must enforce strict role-based access control  
- Must support regional synchronization with eventual consistency  
- Must integrate with third-party payment providers  
- Must be deployable using containerized infrastructure  

---

### Hardware Platform

CodePop is designed primarily as a web-based client application supported by distributed store backend systems.

#### Client Layer

Users access CodePop via:

- Mobile web browsers  
- Desktop/laptop browsers  
- Role-based dashboards  

The interface follows responsive web design principles, prioritizing mobile-first layouts while supporting administrative dashboards for operational roles.

Touchscreen interaction is prioritized for customer ordering flows. Larger UI elements and simplified navigation reduce friction during drink customization.

---

#### Store Server Layer

Each store location runs its own backend services responsible for:

- Processing business logic  
- Managing authentication and authorization  
- Handling payments  
- Executing AI models  
- Managing local inventory and maintenance data  
- Synchronizing with regional nodes  

Each store maintains its own local PostgreSQL database.

If a store loses connectivity to other stores or supply hubs, it continues operating independently and synchronizes once connectivity is restored.

---

## 3. Architecture Design

### Architectural Style

CodePop follows a **Distributed Client–Server Architecture** implemented using a **regional multi-node structure**.

The system is divided into three logical layers at each store location:

1. **Presentation Layer (Client)**
2. **Application Layer (Store Backend Services – Django)**
3. **Data Layer (Local PostgreSQL Database)**

Each store operates as an independent node with its own backend services and database instance. There is no centralized global database.

Stores communicate securely within their region and with assigned supply hubs using authenticated REST APIs over encrypted HTTPS connections.

This design ensures:

- No single point of failure  
- Independent store operation  
- Regional coordination  
- Eventual consistency of shared operational data  

---

### Deployment Architecture (Docker + Google Cloud)

All backend services are containerized using Docker.

Each Store Node and Supply Hub Node includes:

- Dockerized Django backend service  
- Dockerized AI modules  
- Background job workers  
- Local PostgreSQL database  

Deployment options include:

- On-premise hardware at store locations  
- Regional infrastructure hosted on Google Cloud  
- Hybrid deployment models  

Google Cloud Platform (GCP) may be used for:

- Hosting regional supply hub nodes  
- Providing secure networking between regions  
- Monitoring and logging infrastructure  
- Managed compute resources  

However, GCP does not function as a centralized global server. It supports distributed regional infrastructure.

Docker provides:

- Environment consistency across development and production  
- Reproducible builds  
- Service isolation  
- Simplified scaling and deployment  

---

### Major System Components

- Web-Based Client Interfaces  
- Store Backend API Service (Django)  
- Regional Supply Hub Services  
- AI Services Module  
- Inventory & Supply Coordination Module  
- Machine Maintenance Module  
- Order Management Module  
- Regional Synchronization Module  
- Data Import Module  
- Local PostgreSQL Databases  

All communication occurs through authenticated REST APIs over encrypted HTTPS channels.

Stores synchronize regionally rather than globally.

---

### Design Decisions

**Distributed Client–Server Architecture**  
Chosen to satisfy the requirement that the system must not rely on a centralized server and that each store must operate independently.

**Containerization (Docker)**  
Selected for portability, consistency, and simplified deployment across distributed nodes.

**Regional Infrastructure (Google Cloud)**  
Chosen to provide scalable compute resources, secure networking, and monitoring while preserving distributed store autonomy.

**Relational Database (PostgreSQL)**  
Selected to support transactional integrity for orders, payments, inventory, and machine tracking at the store level.

---

### Alternatives Considered

- **Fully Distributed Store Servers:** Rejected due to synchronization complexity, increased security surface area, and operational overhead.
- **Single On-Premise Monolithic Server:** Rejected due to limited scalability and lack of fault tolerance compared to cloud-based deployment.


## Section 4: Modules and Components (Internal Interfaces)

* **User Management Module:** Manages all user types across the expanded role hierarchy.
  * Responsibilities
    * User registration, login, and profile management for all user types
    * Role assignment and multi-location access management
    * Session management and role-based token generation with JWT
    * Enforce role-based access control following hierarchy: `super_admin → admin → manager, logistics_manager, repair_staff → account_user → general_user`
    * Manage drink preferences, likes/dislikes for account_users
    * Handle customer complaint submission and tracking
    * Manage user's saved favorite drinks
  * Components
    * User Service: CRUD operations for all user types, role assignment
    * Authentication Service: Login/logout, session management, token generation
    * Permission Service: Enforce role-based access control
    * Preference Service: Manage drink preferences and ingredient likes/dislikes
    * Complaint Service: Handle customer complaint submission and tracking
    * Favorite Service: Manage user's saved favorite drinks

* **Store Management Module (NEW):** Manages store locations and regional organization.
  * Responsibilities
    * Store metadata management including location coordinates
    * Region assignment and store-to-hub relationships
    * Track operational status, connectivity state, sync timestamps
    * Map 7 supply hub regions
  * Components
    * Store Registry Service: Store metadata, location coordinates, region assignment
    * Region Service: Map 7 supply hub regions, define store-to-hub relationships
    * Store Status Service: Track operational status, connectivity, sync state

* **Supply Hub & Inventory Module (EXPANDED):** Manages regional supply coordination, inventory tracking, and supply transfers.
  * Responsibilities
    * Track stock at store level (syrups, sodas, add-ins, physical items)
    * Manage 7 regional hubs with cross-region delivery (1000-mile rule)
    * Optimize delivery routes between hubs and stores
    * AI-assisted restock recommendations and low-stock alerts
    * Store-to-store supply transfers for nearby locations
    * Manage available drinks, ingredients, and customization options
  * Components
    * Inventory Service: Track stock at store level
    * Supply Hub Service: Manage 7 regional hubs, cross-region delivery
    * Supply Routing Service: Optimize delivery routes between hubs → stores and store → store
    * Reorder Service: AI-assisted restock recommendations, low-stock alerts
    * Transfer Service: Store-to-store supply transfers for nearby locations
    * Menu Service: Manage available drinks and customization options
    * Ingredient Service: Track ingredient types (syrups, sodas, add-ins) and availability

* **Machine Maintenance Module (NEW):** Tracks and schedules machine maintenance across all locations.
  * Responsibilities
    * Track machines per store (type, operational dates, status)
    * Monitor 7 statuses: `normal`, `repair-start`, `repair-end`, `warning`, `error`, `out-of-order`, `schedule-service`
    * AI-optimized scheduling with time constraints
    * Minimize technician travel while respecting max service intervals
    * Notify repair_staff of warning/error conditions
  * Components
    * Machine Registry Service: Track machines per store (type, operational dates, status)
    * Status Tracking Service: Monitor 7 machine statuses
    * Repair Schedule Service: AI-optimized scheduling with time constraints
    * Route Optimization Service: Minimize technician travel while respecting max service intervals
    * Alert Service: Notify repair_staff of warning/error conditions

* **Order Management Module (UPDATED):** Handles orders across multiple store locations.
  * Responsibilities
    * Create, update, cancel orders; assign to specific store
    * Process payments via Stripe
    * Process refunds within 24-hour policy window
    * Geolocation-based or scheduled preparation timing
    * Track drink pickup status, auto-expire after 30 minutes
    * Suggest optimal store based on user location and wait time
    * Send push notifications for order status updates
  * Components
    * Order Service: Manages order creation, updates, cancellation, and status
    * Payment Service: Process payments via Stripe, handle payment methods
    * Refund Service: Process refunds within 24-hour policy window
    * Pickup Coordination Service: Geolocation-based or scheduled preparation timing
    * Cooler Management Service: Track drink pickup status, auto-expire after 30 minutes
    * Store Recommendation Service: Suggest optimal store based on user location and wait time
    * Notification Service: Send push notifications for order status updates

* **Communication/Sync Module (NEW):** Enables decentralized store-to-store communication.
  * Responsibilities
    * Identify nearby stores within region
    * Synchronize inventory, supply needs, maintenance status
    * Handle sync conflicts with eventual consistency
    * Buffer operations when disconnected, sync on reconnect
  * Components
    * Peer Discovery Service: Identify nearby stores within region
    * Data Sync Service: Synchronize inventory, supply needs, maintenance status
    * Conflict Resolution Service: Handle sync conflicts with eventual consistency
    * Offline Queue Service: Buffer operations when disconnected, sync on reconnect

* **Data Import Module (NEW):** Processes CSV imports for bulk data operations.
  * Responsibilities
    * Validate and parse supply usage and repair schedule files
    * Reject malformed data, report errors with details
    * Apply validated data to appropriate entities
    * Track import logs for auditing
  * Components
    * CSV Parser Service: Validate and parse supply usage and repair schedule files
    * Validation Service: Reject malformed data, report errors with details
    * Import Processor Service: Apply validated data to appropriate entities
    * Import History Service: Track import logs for auditing

* **AI Services Module (EXPANDED):** Provides AI-assisted decision making across multiple domains.
  * Responsibilities
    * Personalized drink suggestions based on preferences/history
    * Analyze usage patterns, predict demand
    * Optimize maintenance routes and priorities
    * Provide justification for AI recommendations
  * Components
    * Drink Recommendation Service: Personalized suggestions based on preferences/history
    * Supply Forecasting Service: Analyze usage patterns, predict demand
    * Repair Scheduling Service: Optimize maintenance routes and priorities
    * Explanation Service: Provide justification for AI recommendations

---

## Diagrams

### 1. Component Interaction Diagram

**Description**: Shows the 8 modules and their interactions

![ComponentInteraction](https://hackmd.io/_uploads/BkOXSEZvZe.png)


**Key Interactions**:
- User Management ↔ Store Management: Role assignments, store access
- Store Management ↔ Supply Hub & Inventory: Inventory at stores
- Machine Maintenance ↔ Store Management: Machines at stores
- Communication/Sync ↔ All modules: Data synchronization
- AI Services ↔ Supply, Maintenance, Order: Recommendations
- Data Import → Supply & Maintenance: CSV processing

---

### 2. Data Flow Diagram (DFD)

**Description**: Shows how data moves through the decentralized system

![DataFlow](https://hackmd.io/_uploads/HybzHNZDWx.png)


---

### 3. Decentralized Communication Diagram

**Description**: Illustrates store-to-store sync architecture

![CommunicationArchitecture](https://hackmd.io/_uploads/SkfbSNbP-x.png)


---

### 4. Role-Permission Matrix Diagram

**Description**: Visual of role hierarchy and data access

![PermissionMatrix](https://hackmd.io/_uploads/S1-eHEZD-l.png)


![RoleHIerarchy](https://hackmd.io/_uploads/HyAjE4-w-l.png)


## 5. Data Design

### 5.1 Data Design Goals

The CodePop system requires a data design that supports a **multi-store, distributed architecture** while maintaining data integrity, security, and scalability. The data model must handle user accounts, orders, inventory, machine maintenance, and operational roles across many locations without relying on a centralized server.

The primary goals of the data design are to:
- Support role-based access across multiple store locations
- Ensure transactional consistency for orders and payments
- Enable inventory and machine tracking per store
- Allow safe synchronization of operational data between nearby stores and regional supply hubs
- Protect sensitive user and business data

To meet these goals, CodePop uses a **relational data model** with clearly defined entities and relationships.

**Design Decision Justification**  
A relational database was selected due to its strong support for structured relationships, constraints, and transactions. These features are essential for maintaining consistency in financial data, inventory updates, and user permissions.

**Alternative Considered**  
A NoSQL or document-based database was considered but rejected because it would complicate transactional workflows and relationship enforcement, especially for orders, payments, and role-based access control.

---

### 5.2 Key Data Entities

#### User
Represents any individual interacting with the system, including customers and operational staff.

- `UserID` (Primary Key)
- `Username`
- `Email` (encrypted)
- `PasswordHash`
- `Role`
- `AccountStatus`

---

#### Order
Represents a single drink order placed at a specific store.

- `OrderID` (Primary Key)
- `UserID` (Foreign Key)
- `StoreID` (Foreign Key)
- `OrderStatus`
- `PaymentStatus`
- `PickupTime`
- `CreatedAt`

---

#### Drink
Represents a finalized drink configuration.

- `DrinkID` (Primary Key)
- `SodaType`
- `Syrups`
- `AddIns`
- `Size`
- `Price`

---

#### InventoryItem
Tracks ingredient quantities at a store location.

- `InventoryID` (Primary Key)
- `StoreID` (Foreign Key)
- `ItemName`
- `Quantity`
- `ThresholdLevel`
- `LastUpdated`

---

#### SupplyHub
Represents a regional supply hub that supports multiple stores.

- `HubID` (Primary Key)
- `Region`
- `Location`
- `AvailableInventory`

---

#### Machine
Represents drink-making equipment at a store.

- `MachineID` (Primary Key)
- `StoreID` (Foreign Key)
- `MachineType`
- `Status`
- `StatusDate`

---

#### Payment
Tracks payment transactions associated with orders.

- `PaymentID` (Primary Key)
- `OrderID` (Foreign Key)
- `UserID` (Foreign Key)
- `Amount`
- `PaymentStatus`
- `Timestamp`

---

#### Notification
Represents system-generated messages sent to users or staff.

- `NotificationID` (Primary Key)
- `UserID` (Foreign Key)
- `Type`
- `Message`
- `Timestamp`

---

### 5.3 Entity Relationships

The following high-level relationships define how data entities interact:

- **User → Order:** One-to-Many  
- **Order → Drink:** Many-to-Many  
- **Store → InventoryItem:** One-to-Many  
- **Store → Machine:** One-to-Many  
- **SupplyHub → Store:** One-to-Many  
- **User → Notification:** One-to-Many  

These relationships support role-based access control, operational reporting, and distributed coordination across stores and regions.

---

### 5.4 Database Design

- **Database Type:** Relational
- **Technology Choice:** PostgreSQL
- **Schema Management:** ORM-managed models
- **Indexes:** Applied to primary keys, foreign keys, and frequently queried fields (e.g., `StoreID`, `UserID`)

This structure ensures data consistency and efficient querying while supporting future growth.

---

### 5.5 Data Access Layer

Data access is handled through a centralized Object-Relational Mapping (ORM) layer. This layer:
- Abstracts raw SQL from application logic
- Enforces validation and access rules
- Supports atomic transactions for multi-step operations such as order placement and payment processing

**Benefits**
- Improves maintainability and readability
- Reduces risk of data corruption
- Centralizes security and permission enforcement

---

### 5.6 Security Considerations

Data security is addressed at multiple levels:
- **Application Level:** Role-based access control limits who can view or modify data
- **Data Level:** Sensitive fields (credentials, payments, location data) are encrypted
- **Transaction Level:** Atomic operations prevent partial or inconsistent updates

This layered approach protects both user privacy and business-critical information.

---

## 6. Integration Points (External Interfaces)  

## External Systems and Services

FloatStack integrates with several external platforms to support:

- Payments  
- Geolocation  
- Notifications  
- AI-driven analytics  
- Bulk data ingestion  

These integrations enable:

- Automation  
- Scalability  
- Reduced human intervention across distributed store locations  

---

## Payment Processing System

**Provider:** Stripe  

### Purpose
- Secure payment processing for drink orders  
- Refund handling for canceled orders  
- Support for credit/debit cards and digital wallets  

### Key Interactions
- Payment authorization at order submission  
- Refund issuance upon order cancellation  
- Transaction status callbacks to update order state  

### Benefits
- PCI-compliant handling of sensitive payment data  
- Built-in fraud protection  
- Scalable transaction processing  

---

## Geolocation Services

**Provider:** Mapbox  

### Purpose
- Detect user location for store recommendations  
- Estimate arrival time for drink preparation  
- Support distance-based preparation triggers  

### Key Interactions
- Location permission requests from users  
- Distance calculation between user and store  
- Travel time estimation for order scheduling  

### Benefits
- High accuracy mapping  
- SDK-based integration for mobile and web  
- Low-latency location queries  

---

## Push Notification System

**Provider:** Firebase Cloud Messaging (FCM)  

### Purpose
- Notify users when drinks are ready  
- Send system alerts for inventory and maintenance  
- Provide promotional and status updates  

### Key Interactions
- Order-ready notifications  
- Low inventory alerts to managers  
- Machine error alerts to repair staff  

### Benefits
- Real-time delivery  
- Cross-platform mobile and web support  
- Scalable event messaging  

---

## AI & Analytics Services

**Internal AI Modules (Python-based)**  

### Used For
- Drink recommendation and randomization  
- Supply usage trend analysis  
- Repair schedule optimization  
- Store recommendation logic  

### Input Sources
- Real-time system data  
- Historical order records  
- CSV uploads for supply and maintenance  

### Outputs
- Predictive supply schedules  
- Optimized maintenance routes  
- Personalized user suggestions  

---

## CSV Data Import Interface

### Purpose
- Bulk upload of supply usage data  
- Bulk upload of machine maintenance schedules  

### Supported Files
- Supply usage by store and date  
- Machine status updates  
- Operational start dates  

### Processing Flow
1. User uploads CSV  
2. System validates schema and values  
3. Data stored locally at relevant store nodes  
4. AI analysis executed  

---

## 7. User Interface (UI) Design Overview  

## Design Principles

The FloatStack interface prioritizes:

- Mobile-first responsiveness  
- Role-based clarity  
- Minimal cognitive load  
- Accessibility compliance (WCAG 2.1)  
- Fast task completion  

Each role interacts with a customized dashboard optimized for their responsibilities.

---

## Customer-Facing Interface

### Core Screens
- Home / Store Selection  
- Drink Builder  
- AI Drink Generator  
- Cart & Checkout  
- Pickup Status  
- Order History (account users)  
- Favorites & Preferences  
- Payment Management  

### Key UI Features
- Ingredient icons instead of long lists  
- Swipeable drink suggestions  
- Large touch-friendly controls  
- Real-time order progress display  
- Store readiness indicators  

---

## Manager Dashboard

### Displays
- Inventory grid sortable by quantity  
- Low-stock alerts  
- Order queue (in-progress & scheduled)  
- Cooler status and wait times  
- Revenue summaries  

### Visual Elements
- Color-coded inventory levels  
- Progress bars for machine uptime  
- Daily sales graphs  

---

## Logistics Manager Dashboard

### Displays
- Regional inventory overview  
- Supply hub stock levels  
- Store demand forecasts  
- Delivery routing suggestions  
- CSV upload panel  

### Features
- Heat maps of high-consumption stores  
- AI-generated reorder schedules  
- Route optimization visualization  

---

## Repair Staff Dashboard

### Displays
- Machine status by store  
- Priority repair queue  
- Route-optimized service plans  
- CSV upload interface  

### Visual Indicators
- **Red:** Critical error  
- **Yellow:** Warning  
- **Green:** Normal  

---

## Admin & Super Admin Dashboards

### Admin
- User account management (single store)  
- Store metrics  
- Permissions control  

### Super Admin
- All store analytics  
- Regional performance comparisons  
- System-wide reporting  

---

## Navigation Flow

- Bottom navigation bar for mobile users  
- Sidebar navigation for dashboards  
- Role-based menu visibility  
- No screen deeper than 2–3 taps  

---

## 8. Input and Output (I/O)  

---

## 8.1 User Inputs

### Order Inputs
- **Drink Customization:** Soda/syrup/add-in selection, size (S/M/L), ice level, quantity (1-10)
- **Timing:** GPS coordinates (geolocation), manual pickup time (15-min intervals), or "I'm here" button
- **Management:** Order confirmation, cancellation (pre-preparation), ratings (1-5 stars + optional text)

### Account Management
- **Registration:** Username, email, password, name (optional)
- **Preferences:** Dietary restrictions, favorites, default customizations, notification settings
- **Payment:** Credit/debit cards, digital wallets (Apple Pay, Google Pay) via Stripe

### CSV Uploads (Admin/Manager)
- **Inventory:** [ItemName, ItemType, Quantity, ThresholdLevel] - 50-500 rows, weekly uploads
- **Machine Maintenance:** [MachineID, Location, Type, Status] - 10-100 rows, daily uploads
- **Supply Logistics:** [LocationID, ItemName, CurrentQuantity, UsageRate] - 100-1000 rows, weekly uploads

### Administrative Inputs
- User account management (creation, permissions, locking)
- System configuration (inventory thresholds, pricing, seasonal menus, store settings)
- Customer service actions (complaint responses, refunds, order remakes)

---

## 8.2 System Outputs

### Notifications
**Push (Firebase):** Order confirmations, preparation status, pickup alerts, expirations, promotions (1-5 per order)  
**Email:** Account confirmations, password resets, receipt summaries, loyalty updates (1-3 per user/week)  
**In-App:** Real-time order status, payment confirmations, AI recommendations

### Dashboards
**Manager:** Real-time inventory grid (color-coded warnings), active orders, cooler status, revenue stats (daily/weekly/monthly)  
**Admin:** User accounts, system health, complaint queue  
**Super Admin:** Multi-location performance, system-wide revenue, inventory trends, regional analytics  
**Logistics Manager:** Regional supply levels, AI forecasting, route optimization, usage trends  
**Repair Staff:** Prioritized maintenance queue, GPS routes, machine status history

### Reports
**Financial:** Daily sales (PDF/CSV), monthly revenue, transaction logs  
**Operational:** Inventory analytics, order trends, prep times, satisfaction scores (weekly/monthly)  
**Maintenance:** Uptime metrics, repair analysis, predictive recommendations (monthly)  
**Supply Chain:** Delivery timeliness, cost comparisons, waste reduction (bi-weekly/monthly)

### AI Outputs
**Drink Recommendations:** Personalized suggestions based on history (JSON format)  
**Customer Service (DialoGPT):** Automated responses, escalation flags, sentiment analysis  
**Inventory Forecasting:** Reorder predictions, low-stock alerts, seasonal adjustments  
**Route Optimization:** Service routes for repair staff, priority sequencing, real-time rerouting

---

## 8.3 Expected Data Volumes

### Users & Orders
- **Per Location:** 500-5,000 users, 100-500 orders/day
- **Nationwide:** 50K-500K users (100-1,000 locations), 10K-500K orders/day
- **Peak Hours:** 50-70% during lunch (11am-2pm) and evening (4pm-7pm)
- **Data Size:** ~2 KB per order → 20 MB to 1 GB/day nationwide

### Inventory
- **Items per Location:** 70-115 items (sodas, syrups, add-ins, physical items)
- **Updates:** 100-500 transactions/day per location
- **Storage:** ~10-20 KB current state, ~50-100 KB daily log, ~1.5-3 MB monthly per location

### Notifications
- **Push:** 30K-2M notifications/day nationwide (~0.5-1 KB each)
- **Email:** 5K-50K emails/week (~2-5 KB each)

### CSV Uploads
- **Inventory:** 10-50 KB, 1-2x/week per location
- **Maintenance:** 5-20 KB, daily per region
- **Logistics:** 50-200 KB, weekly per region

### AI Processing
- **Drink Recommendations:** 10K-100K requests/day (<500ms each)
- **Customer Service:** 500-5K interactions/day (<1 sec/message)
- **Inventory Forecasting:** Once daily per location (1-5 min processing)

### Storage Estimates
**Per Location (Annual):**
- Orders: ~300 MB
- Inventory: ~36 MB
- Users: ~12.5 MB
- Notifications: ~225 MB
- **Total:** ~575 MB/year

**Nationwide (100 locations):**
- Base: ~57.5 GB/year
- With 3x backups/redundancy: ~175 GB/year
- With logs and audit trails: ~250-300 GB/year

---

## 8.4 Scalability Considerations

- **Growth:** Must handle 10x increase within 2 years
- **Peak Load:** Support 3x average during promotions
- **Database:** Location-based partitioning, Redis caching for frequent queries
- **Retention:** Orders (7 years), inventory logs (2 years), notifications (90 days)
- **Archival:** Move 1+ year orders to cold storage, automated cleanup of expired data


---

## 9. Security and Privacy  
**Owner: Curt**

### Requirements Alignment
This section supports the following system requirements:

- Role-based access control for all user types  
- Encryption of sensitive data in transit and at rest  
- Secure inter-store communication  
- Independent store operation without reliance on a centralized server  
- Protection of operational, logistics, and user data  

---

### 9.1 Security Overview
Security is especially important in the CodePop platform due to its handling of payment metadata, user account information, machine maintenance records, and regional logistics data. Because stores operate independently in a decentralized architecture, each store must enforce strong security controls while still enabling trusted communication with other stores and supply hubs.

Security will be implemented across multiple layers including application security, network protection, data encryption, and hardware safeguards to ensure system integrity and availability.

**Design Decision:**  
A security-first architecture was selected to support nationwide scaling while minimizing operational risk.

*A security architecture diagram will illustrate encrypted communication between clients, stores, and regional nodes.*

---

### 9.2 Authentication and Authorization
CodePop will enforce secure authentication mechanisms and strict authorization policies to ensure users only access resources permitted by their role.

**Proposed Approach:**
- Secure authentication using industry-standard protocols  
- Role-Based Access Control (RBAC) supporting:
  - super_admin  
  - admin  
  - manager  
  - logistics_manager  
  - repair_staff  
  - account_user  
  - general_user  

Each role will follow the **principle of least privilege**, ensuring access is limited to only what is necessary for operational responsibilities.

**Design Decision:**  
RBAC was selected due to the expanded number of organizational roles introduced in the 2026 system and its ability to scale cleanly as additional stores and personnel are added.

**Alternative Considered:**  
Attribute-Based Access Control (ABAC) was considered but rejected for this phase due to higher implementation complexity and reduced transparency when auditing permissions.

---

### 9.3 Data Encryption
All sensitive data must be encrypted both **in transit** and **at rest** to protect against unauthorized access.

**Encryption in Transit**
- TLS-secured communication between:
  - Client applications and stores  
  - Store-to-store synchronization  
  - Store-to-supply hub communication  

**Encryption at Rest**
Encryption will protect:
- User credentials  
- Payment-related metadata  
- Location data  
- Operational and maintenance records  

Passwords will be hashed using modern cryptographic algorithms to prevent credential exposure.

**Design Decision:**  
Encrypting both stored and transmitted data ensures consistent protection across distributed nodes where no single system acts as a central security boundary.

---

### 9.4 Secure Inter-Store Communication
Since the platform operates without a centralized server, stores must establish trusted peer-to-peer communication channels.

**Security Measures:**
- Mutual authentication between stores  
- Encrypted communication channels  
- Signed data exchanges to prevent tampering  
- Validation of synchronized data  

**Risk:** Distributed communication increases the system’s attack surface.  
**Mitigation:** Only authenticated nodes may participate in regional synchronization.

---

### 9.5 Network and Infrastructure Security
Infrastructure protections help prevent unauthorized access and service disruption at individual store locations.

**Strategies include:**
- Firewall protections  
- Secure network configuration  
- Traffic monitoring and rate limiting  
- Hardened deployment environments  

These controls help ensure that a compromise at one store does not propagate across the region.

---

### 9.6 Privacy Considerations
CodePop prioritizes user privacy by minimizing unnecessary data collection and giving users control over sensitive features.

**Key Practices:**
- Collect only operationally necessary data  
- Allow users to opt into location-sharing features  
- Encrypt personal data  
- Avoid persistent storage for general-user sessions beyond transaction needs  

These practices align with established privacy standards and promote user trust.

---

### 9.7 Logging and Auditing
Logging supports both operational troubleshooting and security investigations across distributed stores.

The system should log:
- Administrative actions  
- Permission changes  
- Authentication attempts  
- Supply and maintenance updates  

**Design Decision:**  
Audit trails improve incident response capabilities and help detect suspicious behavior across regions.

---

### 9.8 Physical and Hardware Security
Because each store functions as an independent system node, physical safeguards are necessary.

**Considerations:**
- Restricted access to store hardware  
- Secure device configuration  
- Protection against unauthorized peripherals  

Physical protections reduce the likelihood of local tampering compromising regional operations.

---

### 9.9 Security Risks and Mitigation Summary

| Risk | Impact | Mitigation |
|------|--------|------------|
| Unauthorized system access | Data breach | Strong authentication + RBAC |
| Inter-store interception | Data tampering | Encrypted channels |
| Compromised credentials | Account takeover | Secure hashing + monitoring |
| Expanded distributed attack surface | Increased vulnerability | Authenticated nodes + network protections |

---

## 10. Testing Strategy  
**Owner: Curt**

### Requirements Alignment
This section supports the following system requirements:

- System reliability across independent store nodes  
- Validation of CSV-imported operational data  
- Secure handling of sensitive information  
- Fault tolerance and safe synchronization  
- Scalable operation across regions  

---

### 10.1 Testing Philosophy
Testing will focus on validating that the system meets functional requirements while remaining reliable, secure, and scalable within a decentralized environment.

Because this document represents a high-level design, the testing strategy emphasizes overall approach rather than implementation detail. Detailed testing procedures will be expanded in the Low-Level Design document.

---

### 10.2 Testing Levels

#### Unit Testing
Individual modules should be tested in isolation to verify expected behavior.

Examples include:
- Authentication logic  
- Inventory updates  
- CSV validation  
- Role permission enforcement  

**Benefit:** Early defect detection reduces downstream integration risk.

---

#### Integration Testing
Integration testing will verify communication between modules and external interfaces.

Focus areas:
- Store-to-store synchronization  
- Payment workflows  
- AI-driven scheduling inputs  
- CSV import pipelines  

*A system interaction diagram will help identify integration test boundaries.*

---

#### System Testing
System-level tests will validate complete operational workflows such as:

- Placing and canceling orders  
- Updating inventory across stores  
- Generating logistics schedules  
- Tracking machine maintenance  

These tests confirm that distributed components function cohesively.

---

#### Security Testing
Security testing will help identify vulnerabilities prior to deployment.

**Approaches include:**
- Authentication validation  
- Permission boundary testing  
- Encryption verification  
- Controlled penetration-style testing  

**Design Decision:**  
Security testing is prioritized due to the expanded attack surface introduced by inter-store communication.

---

### 10.3 Test Data and Simulation
The system will include seeded datasets to simulate realistic operational environments.

Test datasets should include:
- Multiple store locations  
- Supply hubs  
- Machine inventories  
- User roles  
- Regional inventory levels  

CSV test files will validate bulk import functionality before production use.

---

### 10.4 Distributed System Considerations
Testing must account for partial failures common in decentralized architectures.

**Key Scenarios:**
- Store operation while disconnected  
- Synchronization after reconnection  
- Conflict detection and resolution  
- Regional outages  

**Design Decision:**  
Testing failure scenarios improves resilience and supports the requirement that stores remain operational independently.

---

### 10.5 Automation vs. Manual Testing
Automated testing should be prioritized for repeatable validation, while manual testing supports exploratory scenarios and UI verification.

**Design Decision:**  
Automation enhances long-term maintainability and reduces regression risk as the platform scales.

---

## 11. Risks and Mitigations  
**Owner: Curt**

### Requirements Alignment
This section supports the following system requirements:

- Distributed operation without a centralized server  
- AI-assisted logistics and maintenance  
- Secure handling of operational data  
- Scalable nationwide deployment  
- Reliable store performance  

---

### 11.1 Architectural Risks

**Distributed Synchronization Complexity**  
- **Risk:** Independent stores may experience data inconsistencies.  
- **Impact:** Inventory or maintenance conflicts across regions.  
- **Mitigation:** Conflict detection policies and eventual consistency strategies.

---

**Decentralized Coordination Challenges**  
- **Risk:** Removing a centralized server increases communication complexity.  
- **Mitigation:** Clearly defined synchronization protocols and regional data-sharing practices.

---

### 11.2 Operational Risks

**Supply Shortages**  
- **Impact:** Disrupted store operations and lost revenue.  
- **Mitigation:** AI-assisted forecasting combined with regional supply hubs.

**Machine Downtime**  
- **Impact:** Reduced order fulfillment capability.  
- **Mitigation:** Maintenance tracking and optimized repair scheduling.

---

### 11.3 Security Risks

**Unauthorized Access**  
- Mitigated through RBAC and strong authentication.

**Data Interception**  
- Mitigated through encrypted communication channels.

**Privilege Escalation**  
- Mitigated by enforcing strict permission boundaries.

---

### 11.4 Data Risks

**Synchronization Conflicts**  
- Occur when multiple stores update shared datasets.  
- Resolved through structured conflict-handling policies.

**Invalid CSV Imports**  
- Could corrupt operational records.  
- Prevented through pre-processing validation.

---

### 11.5 Scalability Risks

**Rapid Nationwide Expansion**  
- **Risk:** Infrastructure strain as stores increase.  
- **Mitigation:** Modular architecture and configuration-driven deployment.

---

### 11.6 Summary Table

| Risk Category | Example | Mitigation |
|--------------|---------|------------|
| Architectural | Sync failures | Conflict resolution |
| Operational | Low inventory | AI forecasting |
| Security | Unauthorized access | RBAC + encryption |
| Data | Corrupted imports | File validation |
| Scalability | Rapid growth | Modular design |

---