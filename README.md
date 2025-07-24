# Kotia
# Developer Handoff: Poultry Management System

## 1. Project Overview
A web/mobile app to manage a free-range Kienyeji egg farm.  
Core capabilities:
- Daily egg & feed logs  
- Broody-hen incubation & hatch tracking  
- Chick grow-out to point-of-lay  
- Flock roster & status management  
- Sales (table eggs, fertilized eggs, chicks, hens)  
- Expense tracking  
- Summary dashboards (counts, feed, revenue, costs)

---

## 2. Modules & Screens

1. **Egg & Feed Logger**  
   - Date picker, eggs collected, broken eggs, layers active, feed (kg), water (L), notes  
2. **Broody Batch Manager**  
   - Create/edit batch: henID, date set, eggs set, candling D7/D14, hatch date, chicks hatched, return-to-lay date, notes  
3. **Chick Grow-Out**  
   - Track individual chick: chickID, batchID, hatch date, week-4/8 weight, POL date, status, notes  
4. **Flock Roster**  
   - Bird profile: birdID, source, hatch/acq date, batchID (nullable), type, status, statusDate, ageWeeks (calc), notes  
5. **Sales Ledger**  
   - Record sale: date, product type, quantity, unit price, total (calc), buyerID, notes  
6. **Expense Ledger**  
   - Record cost: date, category, detail, amount, notes  
7. **Customer Directory**  
   - BuyerID, name, contact info, type, notes  
8. **Feed Inventory**  
   - PurchaseID, date, feed type, quantity (kg), unit cost, total cost (calc), notes  
9. **Dashboard**  
   - Aggregate views: total birds, active layers, pullets, brooder chicks, broody hens, revenue vs. expense, weekly feed vs. eggs, expense breakdown pie  

---

## 3. Database Schema

### 3.1 Tables & Fields

#### FlockRoster
- **BirdID** (PK, string)  
- Breed (string)  
- Sex (enum: “Hen”, “Rooster”)  
- HatchAcqDate (date)  
- BatchID (FK → BroodyBatch.BatchID, nullable)  
- Type (enum: “Chick”, “Pullet”, “Layer”, “Rooster”)  
- Status (enum: “Active”, “Brooder”, “Pullet”, “Layer”, “Sold”, “Culled”, “Dead”)  
- StatusDate (date)  
- Notes (text)  

#### BroodyBatch
- **BatchID** (PK, autoincrement)  
- HenID (FK → FlockRoster.BirdID)  
- DateSet (date)  
- EggsSet (int)  
- CandledD7 (int)  
- CandledD14 (int)  
- HatchDate (date)  
- ChicksHatched (int)  
- HatchRate (decimal, formula: ChicksHatched/EggsSet)  
- ReturnToLay (date)  
- Notes (text)  

#### ChickProfile
- **ChickID** (PK, string)  
- BatchID (FK → BroodyBatch.BatchID)  
- HatchDate (date)  
- Week4Weight_g (int)  
- Week8Weight_g (int)  
- POLDate (date)  
- CurrentStatus (enum: “Brooder”, “Pullet”, “Layer”)  
- Notes (text)  

#### EggProduction
- **EntryID** (PK, autoincrement)  
- Date (date)  
- EggsCollected (int)  
- BrokenEggs (int)  
- LayersActive (int)  
- Feed_kg (decimal)  
- Water_L (decimal)  
- EggsPerLayer (decimal, formula: EggsCollected / LayersActive)  
- Notes (text)  

#### SalesLedger
- **SaleID** (PK, autoincrement)  
- Date (date)  
- Product (enum: “Table Eggs”, “Fertilized Eggs”, “Chicks”, “Hens”)  
- Quantity (int)  
- UnitPrice (decimal)  
- TotalAmount (decimal, formula: Quantity * UnitPrice)  
- BuyerID (FK → CustomerList.BuyerID)  
- Notes (text)  

#### CustomerList
- **BuyerID** (PK, autoincrement)  
- Name (string)  
- ContactInfo (string)  
- Type (enum: “Farmer”, “Incubator”, “Consumer”, “Hotel”)  
- Notes (text)  

#### ExpenseLedger
- **ExpenseID** (PK, autoincrement)  
- Date (date)  
- Category (enum: “Feed Purchase”, “Packaging”, “Labor”, “Repairs”, “Utilities”, “Other”)  
- Detail (string)  
- Amount (decimal)  
- Notes (text)  

#### FeedInventory
- **PurchaseID** (PK, autoincrement)  
- Date (date)  
- FeedType (string)  
- Qty_kg (decimal)  
- UnitCost (decimal)  
- TotalCost (decimal, formula: Qty_kg * UnitCost)  
- Notes (text)  

---

### 3.2 Relationships

- **FlockRoster → BroodyBatch**  
  one-to-many (BirdID)  
- **BroodyBatch → ChickProfile**  
  one-to-many (BatchID)  
- **EggProduction → SalesLedger**  
  match on Date for reporting  
- **SalesLedger → CustomerList**  
  many-to-one (BuyerID)  
- **ExpenseLedger ↔ FeedInventory**  
  feed purchase reconciliation  

---

## 4. Sample API Endpoints (RESTful)

- `GET /api/flock` → list all birds  
- `POST /api/flock` → add new bird  
- `PATCH /api/flock/{birdID}` → update status/type  
- `GET /api/broody` → list batches  
- `POST /api/broody` → create new batch  
- `GET /api/chicks?batchID={id}` → list chicks  
- `GET /api/eggs?start=YYYY-MM-DD&end=YYYY-MM-DD` → production log  
- `POST /api/eggs` → record daily log  
- `GET /api/sales` → sales ledger  
- `POST /api/sales` → record transaction  
- `GET /api/expenses` → expense ledger  
- `POST /api/expenses` → record expense  
- `GET /api/dashboard` → aggregated metrics  

---

## 5. UI Wireframe & Workflow Notes

- **Egg & Feed Logger**: simple form → saves to `EggProduction`  
- **Broody Batch**: date picker + numeric fields + auto-calc hatch rate  
- **Chick Profile**: list by batch, edit grow-out milestones  
- **Roster View**: table filterable by status/type, click to edit bird profile  
- **Sales & Expenses**: quick-entry forms + tables with totals at top  
- **Dashboard**: cards for headcounts, line chart (weekly feed vs eggs), bar chart (monthly rev vs expense), pie chart (expense breakdown)
