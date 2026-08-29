# Pos-Restaurant
# Restaurant POS — Development Plan (29 Aug → 13 Sep 2026)

**Stack :** Electron.js + React (Vite) + Node.js/Express (local) + Prisma + SQLite + node-thermal-printer + electron-builder.

**Timeline:** 16 days 

---

## Sprint 1 — Fri 29 Aug → Sat 30 Aug: Project Foundation

The base everything else sits on. Do not skip anything here.

- [ ] Scaffold Electron + React (Vite) + Express running together (Electron spawns the local Express server).
- [ ] Set up Prisma + SQLite; write the **full database schema** up front:
  - `users` (roles), `categories`, `menu_items` (price, VAT category), `tables`, `orders`, `order_items`, `bills`, `payments`, `customers`, `loyalty_transactions`, `inventory_items`, `stock_movements`, `recipes`, `suppliers`, `purchase_orders`, `batches`, `wastage`, `expenses`, `staff`, `attendance`, `shifts`, `leaves`, `payroll`, `kot_tickets`, `settings`.
- [ ] App shell: sidebar navigation, routing, base layout, theme.
- [ ] Staff login + role-based access (Admin, Cashier, Waiter, Kitchen) — simple session, all offline.
- [ ] Settings screen skeleton (restaurant name, currency, VAT mode).

**Done when:** App opens as a desktop window, login works, empty pages exist for every module, DB migrates cleanly.

---

## Sprint 2 — Sun 31 Aug → Mon 1 Sep: Menu Management + Tap Ordering (POS Screen)

The POS screen is the heart — build it early so every later module plugs into it.

- [ ] Menu CRUD: categories (Starters, Main Course, Beverages, Desserts…), items with price, image, VAT category, active/inactive.
- [ ] Tap-ordering POS screen:
  - Category tabs → tap item → added to cart.
  - Quantity + / − buttons, remove item.
  - Special instructions per item.
  - Order summary panel with running total.
- [ ] Order engine (backend): create order, add/edit/remove items, order statuses (`draft → confirmed → completed → cancelled`), unique **order/token number** generator.
- [ ] Order type selector: Dine-In / Takeaway (types wired fully in Sprints 3 & 5).

**Done when:** You can build an order by tapping items and confirm it; it saves to DB with a token number.

---

## Sprint 3 — Tue 2 Sep → Wed 3 Sep: Dine-In + Table Management

- [ ] Table layout screen: add/edit tables, visual grid/floor layout.
- [ ] Table status: **Free / Occupied / Reserved** with color coding; status auto-updates from orders.
- [ ] Tap a table → opens POS screen for that table; one running order per table.
- [ ] Edit order before billing (add/remove/change qty — these changes feed KOT reprint in Sprint 4).
- [ ] Table reservations: reserve with customer name/phone/time, mark arrived/cancelled.
- [ ] Transfer order to another table; groundwork for merge (two tables → one bill).

**Done when:** Full dine-in flow works: pick table → order → edit → table shows Occupied → close order frees table.

---

## Sprint 4 — Thu 4 Sep → Fri 5 Sep: Billing + KOT Printing + E-Invoice (VAT)

- [ ] Bill generation from an order: subtotal, discount, VAT, grand total.
- [ ] **Split bill** (by items or equal split) and **merge bill** (multiple tables/orders → one bill).
- [ ] Payment recording: cash / card, change calculation, mark paid → order completed → table freed.
- [ ] **KOT engine:**
  - Auto-generate KOT on order confirm: table no., items, qty, special instructions, token no. — **no prices**.
  - On order change after confirm → auto-print updated KOT (only the changes flagged NEW/CANCELLED).
  - Printing via `node-thermal-printer` (kitchen printer) with a fallback "print to PDF/preview" mode for dev without hardware.
- [ ] Customer receipt printing.
- [ ] **E-Invoice:** invoice number sequence, date/time, customer details, item-level VAT (inclusive & exclusive modes), stored invoice records screen.

**Done when:** Confirm order → KOT prints; bill with correct VAT prints; split/merge work; invoices are stored and viewable.

---

## Sprint 5 — Sat 6 Sep → Sun 7 Sep: Takeaway (all types) + KDS

- [ ] Takeaway flows on the POS screen:
  - **Walk-in** — counter order → bill → collect.
  - **Phone/Interchange** — capture customer name + phone → prepare for pickup.
  - **House delivery** — customer address, assign own rider, delivery status (Preparing → Out for delivery → Delivered).
- [ ] Rider management: add riders, assign to order.
- [ ] **KDS (Kitchen Display System):**
  - Serve a KDS web page from the local Express server so any tablet/screen on the restaurant's LAN can open it (fully offline).
  - Shows: token no., table no., order type, items + qty, special instructions, order time, elapsed time.
  - Kitchen taps status: **New → Preparing → Ready → Completed**; status syncs live to POS (WebSocket/socket.io over LAN).
  - New confirmed orders appear automatically.

**Done when:** All 3 takeaway types work end-to-end; a browser on another device on LAN shows live kitchen orders and status updates reflect in POS.

---

## Sprint 6 — Mon 8 Sep → Tue 9 Sep: Inventory (full) + Expenses

- [ ] Basic inventory: item list with units, **stock in / stock out**, current stock view, **low-stock alerts** (threshold per item, dashboard badge).
- [ ] **Recipe-based deduction:** define recipe per menu item (e.g., 1 burger = 1 bun + 1 patty + 1 cheese + sauces); on sale, ingredients auto-deduct.
- [ ] Suppliers: CRUD + purchase history.
- [ ] Purchase orders: create PO → receive PO → stock-in automatically.
- [ ] Advanced: **batch tracking**, **expiry dates** (expiring-soon alerts), **wastage tracking** with reasons, wastage reports/analytics, **automatic reorder suggestions** (below-threshold list).
- [ ] **Expenses module:** categories (Electricity, Bills, Staff salary, Rent, Others), add expense with date/amount/note, monthly expense report.

**Done when:** Selling a burger deducts its ingredients; PO receive increases stock; expiry/wastage/reorder screens work; expenses are recorded and reportable.

---

## Sprint 7 — Wed 10 Sep → Thu 11 Sep: HR & Payroll + Customer Loyalty

- [ ] Staff profiles (linked to login users where applicable).
- [ ] **Attendance:** daily check-in/out, monthly attendance sheet.
- [ ] **Shifts:** define shifts, assign staff.
- [ ] **Leave management:** leave types, apply/approve, leave balance.
- [ ] **Payroll:** basic salary calc (from attendance), **deductions**, **bonuses**, generate + print **salary slips**.
- [ ] **Loyalty & Points:**
  - Customer profile: name, phone, email (optional), purchase history.
  - Configurable earning rule (e.g., $1 = 1 point), points auto-added when bill completes.
  - Redeem points as discount at billing (max-points-per-bill limit, points required per discount, optional expiry).
  - Full points history (earned/used) + customer lookup at POS by phone number.

**Done when:** A full month's salary slip can be generated; a repeat customer earns and redeems points on a bill.

---

## Sprint 8 — Fri 12 Sep → Sun 13 Sep: QR Menu, Reports, Backup
- [ ] **Scanner/QR menu (Android):** Express serves a mobile-friendly read-only menu page on LAN; generate & print QR code per table.
- [ ] **Reports/Dashboard:** daily/weekly/monthly sales, top items, order-type breakdown, expenses vs sales, VAT report, inventory valuation.
- [ ] **Backup & Restore:** manual backup (choose location/USB), restore from file, optional automatic scheduled backup — SQLite file copy + verify.
- [ ] **Offline licensing (infrastructure only):** license key + device/hardware fingerprint, local validation, feature-flag framework in place (all flags ON for now — tier gating is a later task).
- [ ] **Packaging:** electron-builder → Windows `.exe` (macOS `.dmg` config kept ready).
- [ ] Full end-to-end testing of every flow + bug fixing. Priority order if time runs short: POS/billing/KOT bugs first, then inventory, then HR/reports polish.

**Done when:** A packaged installer runs the whole system offline on a clean Windows and mac machine.

---

## Testing — Monday → Sep 14 (Meeting)






