# VORTEX LOUNGE — Esports Lounge & Gaming Café Platform

A complete, production-ready full-stack web application and relational database management system for **VORTEX LOUNGE**, a high-end Esports Gaming Café and Sim Racing Lounge.

Designed for consumer players, floor shift supervisors, and lounge general managers.

---

## 1. System Features by Role

### 🎮 Players / Customers
- **Explore Gaming Fleet:** Real-time floor availability across 16 rigs (Esports Pro Arena RTX 4080 / 240Hz OLED, VIP Elite 360Hz Liquid Pods, Console PS5 Pro Bay, and Fanatec Direct Drive Sim Racing Cockpits).
- **Active Session HUD:** Real-time ticking stopwatch timer, running charges prorated by minute, active station details, and instant 1-tap checkout.
- **Advance Reservations:** Book stations ahead with deposit credit hold.
- **Digital Lounge Wallet:** Instant credit top-up ($15, $30, $50, $100), automated session checkout deduction, and transaction ledger.
- **Vortex Café & Bar Delivery:** Full digital menu with categorized ordering (Craft energy cans, nitro cold brew, hot loaded snacks, desserts) delivered directly to player gaming stations.
- **Passes & History:** Review active membership tier (Standard, Silver Gamer, Gold Pro, Vortex VIP), loyalty points, receipts, and gameplay statistics.

### 🎧 Staff / Shift Operators
- **Live Floor Control:** Instant overview of occupied, available, reserved, and maintenance stations.
- **Fast Player Check-In:** Seat walk-in guests or online reservations onto any rig.
- **Live Session Telemetry:** Monitor elapsed minutes, billing rates, and execute checkout & wallet settlements.
- **Kitchen & Bar Fulfillment Queue:** Real-time order pipeline (`PLACED` → `PREPARING` → `SERVED`) with delivery destination.
- **Shift Timeclock & Attendance:** Clock-in and clock-out with automated late-arrival tracking.
- **Time-Off & Leave Management:** Submit vacation, sick, or personal leave requests directly to the general manager.

### 📊 Lounge Manager / General Manager
- **Executive Dashboard:** Live floor utilization percentage, gross daily revenue, active rigs count, on-duty personnel, and pending café orders.
- **Business Intelligence Analytics Hub:** Interactive reports backed by complex SQL views:
  - Station utilization, total runtime hours, and gross gaming revenue.
  - Daily revenue breakdown across gaming sessions, café orders, and wallet top-ups.
  - Peak gaming hours analysis (24-hour cycle demand distribution).
  - Café profitability, bestsellers, COGS, and profit margins.
  - Player loyalty leaderboard and lifetime lounge spend rankings.
  - Staff attendance, shift reliability, late arrival logs, and earned wages.
- **Automated Payroll Engine:** Run payroll calculations (`sp_generate_staff_payroll`) partitioned into regular and 1.5x overtime hours, review generated pay slips, and disburse payments.
- **Station Fleet Management:** Add new stations, adjust base hourly rates, edit hardware specifications, and toggle maintenance mode.

---

## 2. Relational Database Architecture

The database suite is located under `/database` and runs in **MySQL 8.0+** and **MySQL Workbench**:

| Script | Purpose |
|---|---|
| [`01_schema.sql`](file:///c:/Users/hp/OneDrive/Desktop/dbms_capstoneee/database/01_schema.sql) | 3NF normalized schema with PKs, FKs, `CHECK`, `UNIQUE` constraints, indexes, and timestamps across 19 entities. |
| [`02_routines_triggers.sql`](file:///c:/Users/hp/OneDrive/Desktop/dbms_capstoneee/database/02_routines_triggers.sql) | Stored functions, atomic transaction procedures (`START TRANSACTION`, `COMMIT`, `ROLLBACK`), and automated inventory/wallet triggers with `DELIMITER` syntax. |
| [`03_reports_views.sql`](file:///c:/Users/hp/OneDrive/Desktop/dbms_capstoneee/database/03_reports_views.sql) | 6 business intelligence views featuring multi-table `JOIN`s, `GROUP BY`, conditional aggregations, and window-ready metrics. |
| [`04_seed.sql`](file:///c:/Users/hp/OneDrive/Desktop/dbms_capstoneee/database/04_seed.sql) | Rich production seed data (16 stations across 4 zones, 12 esports games, 13 café items, active and historical sessions, staff shifts, leaves, and payroll records). |

### Stored Functions
- `fn_calculate_session_charge(p_minutes, p_category_id, p_user_id) RETURNS DECIMAL(10,2)`:
  Enforces 15-minute minimum billing block, prorates category base hourly rate by minute, queries `player_profiles` & `membership_tiers` to apply tier loyalty discounts, and returns rounded net charge.
- `fn_calculate_net_pay(p_regular_hours, p_overtime_hours, p_hourly_wage, p_deductions, p_bonus) RETURNS DECIMAL(10,2)`:
  Calculates regular pay + (1.5x overtime multiplier) + bonus - deductions.

### Stored Procedures
- `sp_start_gaming_session(p_user_id, p_station_id, p_staff_id, OUT p_session_id)`:
  Atomically checks station availability, ensures user has no active session, inserts `gaming_sessions` record, and updates `gaming_stations.status = 'OCCUPIED'`.
- `sp_checkout_session(p_session_id, p_payment_method, OUT p_total_billed, OUT p_invoice_id)`:
  Atomically computes duration, calculates charge via `fn_calculate_session_charge`, deducts from `player_wallets` if paying by wallet (with balance check), writes `wallet_transactions` ledger entry, creates `invoices`, logs `payments`, frees up the station (`AVAILABLE`), and updates player total play time.
- `sp_topup_wallet(p_user_id, p_amount, p_method, p_ref, OUT p_new_balance)`:
  Adds credit to player wallet, logs ledger record, and awards 1 loyalty point per $1 topped up.
- `sp_generate_staff_payroll(p_start_date, p_end_date, p_staff_id)`:
  Cursor iterates active staff, sums completed shift hours, partitions into regular vs overtime (>40h), invokes `fn_calculate_net_pay`, and upserts `staff_payroll` records.

### Automated Triggers
- `trg_after_cafe_order_item_insert`: Decrements item stock in `cafe_items.stock_quantity`.
- `trg_prevent_negative_wallet`: Raises `SIGNAL SQLSTATE '45000'` if `player_wallets.balance < 0`.
- `trg_on_session_complete`: Synchronizes station status to `AVAILABLE` upon session completion.

---

## 3. Quickstart & Deployment

### Option A: 1-Click Multi-Service Docker Compose
Ensure Docker Desktop is running, then execute from the project root:
```bash
docker-compose up --build
```
- Web Application: `http://localhost:3000`
- API Gateway: `http://localhost:5000`
- MySQL 8.0 Database: `localhost:3306` (preloaded with all scripts)

---

### Option B: Local Setup (MySQL Server / MySQL Workbench)

#### 1. Import Database Scripts in MySQL Workbench
1. Open **MySQL Workbench** and connect to your local MySQL instance (`localhost:3306`).
2. Go to **File → Open SQL Script...** and execute the files in order:
   - `database/01_schema.sql` (Creates `vortex_lounge` database and all 19 tables)
   - `database/02_routines_triggers.sql` (Installs functions, stored procedures & triggers)
   - `database/03_reports_views.sql` (Creates analytical views)
   - `database/04_seed.sql` (Populates seed data)
3. Click the **Lightning Bolt (Execute)** button for each script.

> **Tip:** You can also run the automated import runner from the `server/` directory:
> ```bash
> cd server
> npm run setup:db
> ```

#### 2. Start the Backend API Gateway
```bash
cd server
npm install
npm run dev
```
The API starts on `http://localhost:5000`. Test health: `http://localhost:5000/api/health`.

> **Dual Connection Architecture:** If your MySQL service is temporarily offline, the backend server automatically runs in **Standby Mode** with an integrated in-memory transactional database engine. All endpoints, stored procedures, wallet balances, and views remain 100% interactive! When your MySQL database is online, it connects directly.

#### 3. Start the Frontend Web Application
```bash
cd client
npm install
npm run dev
```
Open `http://localhost:3000` in your browser.

---

## 4. Test Accounts & Instant Switcher

The web app includes an **Instant Account & Role Switcher** in the top navigation bar. You can also log in directly using the default password: `Password123!`

| Role | Name | Email | Username | Description |
|---|---|---|---|---|
| **ADMIN** | Marcus Vance | `admin@vortexlounge.com` | `vortex_admin` | General Lounge Manager, Analytics BI Hub, Payroll Engine |
| **STAFF** | Elena Rostova | `elena@vortexlounge.com` | `elena_floor` | Lounge Shift Lead, Floor Station Check-In/Out, Café Queue |
| **STAFF** | Kai Tanaka | `kai@vortexlounge.com` | `kai_tech` | Systems Engineer, Hardware Maintenance, Network IP |
| **PLAYER** | Alex Rivera | `alex@vortexgamer.com` | `apex_alex` | Vortex VIP Member (25% hourly discount, $142.50 balance) |
| **PLAYER** | Claire DeWitt | `claire@vortexgamer.com` | `neon_claire` | Gold Pro Member (15% hourly discount, $68.00 balance) |
| **PLAYER** | Maya Patel | `maya@vortexgamer.com` | `pixel_maya` | Silver Gamer Member (5% hourly discount, $24.50 balance) |

---

## 5. API Reference Summary

### Authentication & Profiles
- `POST /api/auth/login` — Sign in with email and password
- `POST /api/auth/register` — Create player account, standard tier & wallet
- `GET /api/auth/me` — Fetch authenticated profile, wallet balance & tier perks
- `GET /api/auth/demo-users` — List demo accounts for instant switching

### Gaming Stations & Fleet
- `GET /api/stations` — List all 16 stations with live status and active player info
- `GET /api/stations/categories` — List hardware categories & base hourly rates
- `PATCH /api/stations/:id/status` — Toggle status (`AVAILABLE`, `MAINTENANCE`, `OCCUPIED`)
- `POST /api/stations` — Add new station to fleet (Admin only)

### Gaming Sessions
- `GET /api/sessions/active` — Real-time monitor of all floor sessions (Staff/Admin)
- `GET /api/sessions/my` — Ongoing session & past receipts (Player)
- `POST /api/sessions/start` — Start live session (invokes `sp_start_gaming_session`)
- `POST /api/sessions/checkout` — Checkout & wallet deduction (invokes `sp_checkout_session`)

### Digital Lounge Wallet
- `GET /api/wallet` — Current credit balance & transaction ledger
- `POST /api/wallet/topup` — Top-up credits & award loyalty points (invokes `sp_topup_wallet`)

### Café & Snack Bar
- `GET /api/cafe/menu` — Food, coffee & cold drinks grouped by category
- `POST /api/cafe/order` — Place order with station delivery & wallet checkout
- `GET /api/cafe/orders` — Kitchen/Bar fulfillment queue (`PLACED`, `PREPARING`, `SERVED`)
- `PATCH /api/cafe/orders/:id/status` — Update order status

### Staff Floor Operations & Payroll
- `GET /api/staff/roster` — List employees, hourly wages, and hire dates
- `GET /api/staff/shifts` — Roster shifts and punctuality logs
- `POST /api/staff/timeclock` — Clock-in / clock-out logging
- `POST /api/staff/leaves` — Submit time-off request
- `GET /api/payroll` — Staff pay stubs
- `POST /api/payroll/generate` — Calculate period payroll (invokes `sp_generate_staff_payroll`)
- `PATCH /api/payroll/:id/disburse` — Mark pay slip as disbursed

### Business Intelligence & Analytics
- `GET /api/reports/utilization` — Query `vw_station_utilization_analytics`
- `GET /api/reports/revenue` — Query `vw_daily_revenue_breakdown`
- `GET /api/reports/peak-hours` — Query `vw_peak_gaming_hours`
- `GET /api/reports/cafe` — Query `vw_top_cafe_performers`
- `GET /api/reports/staff` — Query `vw_staff_attendance_summary`
- `GET /api/reports/leaderboard` — Query `vw_player_loyalty_leaderboard`
- `GET /api/reports/summary` — Executive KPI summary cards
#   G a m i n g _ C a f e _ M a n a g e m n t  
 