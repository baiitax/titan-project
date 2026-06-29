# Project Titan: Integrated Garage & Car Stand Management System
An enterprise-grade, decoupled, modular software platform engineered to seamlessly manage two independent yet interconnected business units: a **Car Maintenance Garage** and a **Car Stand (Consignment Lot)**.
Project Titan optimizes the hybrid **"Refurbish-and-Sell"** business pipeline by automating cross-department debt calculations and enforcing strict, API-level Role-Based Access Control (RBAC) with an unalterable financial audit trail.
## 🚀 Tech Stack
 * **Framework:** Next.js (App Router)
 * **Language:** TypeScript
 * **Styling:** Tailwind CSS
 * **Backend & Database:** Supabase (PostgreSQL, Auth, Edge Functions)
 * **Security:** Row-Level Security (RLS) & Database Triggers
## 💎 Core Features
### 1. Centralized Vehicle Registry
 * **Single Intake:** Vehicles are registered once via immutable structural parameters (VIN, Engine Number, Make, Model).
 * **Global State Machine:** Real-time system flags monitor and enforce sequential progression through operational phases.
### 2. Maintenance Garage Module
 * **Digital Job Cards:** Live tracking of parts costs and labor allocation.
 * **Automated Handoffs:** Supports immediate retail client billing or seamless transfer to consignment inventory with automatically frozen and deferred balances.
### 3. Car Stand Module
 * **Asset Tracking:** Lightweight consignment lot engine mapping fixed lot fees and target retail list pricing.
 * **One-Click Settlements:** Instantaneous generation of itemized payouts upon entry of the final sale price.
### 4. Financial Bridge & Governance
 * **Automated Reconciliation:** Programmatic calculation of client payouts:
   Final Payout = Final Sale Price - Fixed Stand Fee - Deferred Garage Bill
 * **Immutable Audit Trail:** Append-only database ledger logging all data mutations. Deletions and historical modifications are blocked at the database level.
## 📂 Database Architecture
The system relies on a PostgreSQL schema managed via Supabase. Below is the structural layout of the main tables:
```
    [public.profiles] (RBAC Roles: manager / management)
            │
            ▼
    [public.vehicles] <───────────────┐ (Central Hub)
            │                         │
            ├─────────────────────────┼────────────────────────┐
            ▼                         │                        ▼
   [public.garage_jobs]               │            [public.car_stand_inventory]
   (Parts + Labor Billing)            │            (Consignment Fees & Sales)
            │                         │                        │
            └───────── [Financial Bridge Sync] ────────────────┘

```
### Core Schema Definition
```sql
-- Enums
CREATE TYPE user_role AS ENUM ('manager', 'management');
CREATE TYPE vehicle_state AS ENUM ('Under Maintenance', 'Pending Transfer', 'Active on Stand', 'Settled & Exited', 'Archived (Paid)', 'Archived (Closed)');

-- Central Vehicle Table
CREATE TABLE public.vehicles (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    vin TEXT UNIQUE NOT NULL,
    make TEXT NOT NULL,
    model TEXT NOT NULL,
    year INT NOT NULL,
    status vehicle_state NOT NULL DEFAULT 'Under Maintenance',
    owner_name TEXT NOT NULL,
    owner_email TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc'::text, NOW()) NOT NULL
);

```
## 🔐 Security & Role-Based Access Control (RBAC)
The application enforces a strict split-ring security environment to decouple daily operations from corporate governance.
| User Segment | Target Roles | Access Scope |
|---|---|---|
| **Manager Segment** | Workshop Managers, Sales Attendants | Read/Write access to active records. No deletion or historical modification rights. |
| **Management Segment** | Business Owners, C-Suite, Auditors | Full CRUD capabilities, immutable audit logs, financial ledger configuration. |
 * **API Enforcement:** Implemented directly via Supabase Row-Level Security (RLS) policies. Unauthorized requests across security segments instantly drop connection with a 403 Forbidden error.
 * **Data Protection:** Client contact profiles and physical settlement documentation are fully encrypted at rest.
## 🛠️ Getting Started & Installation
### Prerequisites
 * Node.js (v18.0.0 or higher)
 * A Supabase Project Instance
### 1. Clone the Repository
```bash
git clone https://github.com/your-organization/project-titan.git
cd project-titan

```
### 2. Install Dependencies
```bash
npm install

```
### 3. Environment Configuration
Create a .env.local file in the root directory and populate it with your Supabase credentials:
```env
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-supabase-service-role-key

```
### 4. Database Setup
 1. Copy the contents of /supabase/schema.sql into the **Supabase SQL Editor**.
 2. Run the script to generate tables, custom enums, and database policies.
### 5. Run the Local Development Server
```bash
npm run dev

```
Open http://localhost:3000 in your browser to view the application gate.
## 📈 System Workflow Lifecycle
Vehicles must progress strictly through the following state configurations:
```
[Intake] ──> Under Maintenance ──> Pending Transfer ──> Active on Stand ──> Settled & Exited ──> Archived

```
 1. **Garage Phase:** Vehicle checked in \rightarrow Under Maintenance.
 2. **The Bridge:** Job card closed as consignment \rightarrow Pending Transfer (Garage invoice balance freezes).
 3. **Stand Phase:** Vehicle accepted by sales lot \rightarrow Active on Stand.
 4. **Settlement:** Sale price entered \rightarrow Settled & Exited (Cross-department calculation triggers automatically).
 5. **Audit Close:** Executive runs system validation \rightarrow Archived (Closed).
## 📄 License
This project is proprietary and confidential. Unauthorized copying, modification, or distribution of this software via any medium is strictly prohibited. Generated and maintained by **DECIDIDO LTD**.
