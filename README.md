# 🔗 Northwind OData Integration — SAP CAP + XSUAA + BTP

A **SAP CAP** application that integrates with the external **Northwind OData API**,
exposes enriched projections with custom associations, and secures endpoints using
**XSUAA** role-based authentication — deployed on **SAP BTP** via Cloud Foundry.

---

## 🚀 Tech Stack

| Layer | Technology |
|---|---|
| Backend Framework | SAP CAP (Node.js) |
| External API | Northwind OData V4 (public API) |
| Authentication | XSUAA (OAuth2 / JWT) |
| API Protocol | OData V4 |
| Routing | SAP AppRouter |
| Deployment | SAP BTP – Cloud Foundry (MTA) |

> ⚡ No database — all data is proxied live from the external Northwind API via CAP's
> remote service consumption feature.

---

## 📐 Data Model

All entities are **CDS projections** on top of the external `northwind_Api_Service`,
enriched with custom **associations** for relational navigation.
```
Categories
  └── Products (Association to many)
        └── Order_Details (Association to many)
              └── Orders (Association to one)
                    └── Customers (Association to one)
```

### Entities & Access Control

| Entity | Source | Role Required |
|---|---|---|
| `Categories` | `northwind_Api_Service.Categories` | `Viewer` |
| `Products` | `northwind_Api_Service.Products` | `Viewer` (via Categories) |
| `Customers` | `northwind_Api_Service.Customers` | `Admin` |
| `Orders` | `northwind_Api_Service.Orders` | `Admin` (via Customers) |
| `Order_Details` | `northwind_Api_Service.Order_Details` | — |

> 🔒 `@requires: 'Viewer'` on Categories and `@requires: 'Admin'` on Customers
> ensures role-level entity access control out of the box.

### Custom Associations Added

- `Categories` → `Products` (one-to-many)
- `Products` → `Categories` (many-to-one)
- `Products` → `Order_Details` (one-to-many)
- `Orders` → `Customer` (many-to-one)
- `Order_Details` → `Order` and `Product` (many-to-one each)

---

## 🔐 Security — XSUAA

| Role | Scope | Access |
|---|---|---|
| `Viewer` | `$XSAPPNAME.Viewer` | Categories, Products |
| `Admin` | `$XSAPPNAME.Admin` | Customers, Orders, full access |

Role collections (`CAP Viewer PRODUCTION`, `CAP Admin PRODUCTION`) are defined in
`xs-security.json` and assigned to BTP users via the BTP cockpit.

---

## 🌐 OData V4 API Examples

| Operation | Endpoint |
|---|---|
| List Categories | `GET /odata/v4/northwind/Categories` |
| Products with Category | `GET /odata/v4/northwind/Products?$expand=Category` |
| Orders with Customer | `GET /odata/v4/northwind/Orders?$expand=Customer` |
| Order Details with Product | `GET /odata/v4/northwind/Order_Details?$expand=Product` |
| Filter Products | `GET /odata/v4/northwind/Products?$filter=CategoryID eq 1` |
| Customer Orders | `GET /odata/v4/northwind/Customers?$expand=Orders` |

---

## 🏗️ BTP Architecture
```
Browser
  └── AppRouter (myNorthwindApp-AppRouter)   ← Auth redirect, token forwarding
        └── CAP Service (srv)                ← OData V4 projections + associations
              └── Northwind External API     ← Remote OData source (no local DB)

XSUAA ←──────────────────────────────────── OAuth2 token validation
```

> Unlike typical CAP apps, this project uses **no HDI/HANA container** — CAP
> delegates all data fetching to the remote Northwind service at runtime.

---

## 📁 Project Structure
```
sap-cap-northwind-odata-integration/
├── approuter/
│   └── xs-app.json              # AppRouter routing config
├── db/
│   └── schema.cds               # CDS projections on external service
├── srv/
│   ├── northwind-service.cds    # Service exposure & @requires annotations
│   └── external/
│       └── northwind-Api-Service.cds   # Imported external service definition
├── xs-security.json             # XSUAA scopes, roles, role-collections
├── mta.yaml                     # BTP Multi-Target Application config
├── package.json
└── README.md
```

---

## ⚙️ Local Setup

### Prerequisites
- Node.js >= 18
- `@sap/cds-dk` installed globally → `npm install -g @sap/cds-dk`

### Run Locally
```bash
git clone https://github.com/Piyush2276/sap-cap-northwind-odata-integration
cd sap-cap-northwind-odata-integration
npm install
cds run    # starts at http://localhost:4004
```

> 💡 In local mode, CAP will use a mocked version of the external service
> unless you configure a live remote destination. XSUAA is bypassed locally.

### Connect to Live Northwind API (optional)

Add to your `.env` or `default-env.json`:
```json
{
  "destinations": [
    {
      "name": "Northwind",
      "url": "https://services.odata.org/V4/Northwind/Northwind.svc/"
    }
  ]
}
```

---

## ☁️ BTP Deployment (MTA)
```bash
# Login to Cloud Foundry
cf login -a https://api.cf.<region>.hana.ondemand.com

# Build MTA archive
mbt build

# Deploy to BTP
cf deploy mta_archives/myNorthwindApp_1.0.0.mtar
```

### BTP Services Provisioned

| Service | Plan | Purpose |
|---|---|---|
| `xsuaa` | `application` | Auth & authorization |

> No HANA service needed — this app is fully stateless, proxying external data.

---

## 📝 Key Concepts Demonstrated

- ✅ **Remote service consumption** — CAP connecting to an external OData API
- ✅ CDS **projections** on external service entities
- ✅ **Custom associations** added on top of external entity projections
- ✅ Entity-level access control with `@requires` annotations
- ✅ XSUAA Viewer/Admin role-based security
- ✅ **Stateless BTP deployment** — no database, pure API proxy pattern
- ✅ AppRouter integration with JWT token forwarding
- ✅ MTA deployment with only XSUAA (no HDI container needed)

---

## 👤 Author

**Piyush Kumar**
SAP BTP Backend Developer
📧 piyush2582002@gmail.com
🔗 [LinkedIn](https://linkedin.com/in/piyush-kumar-267367229) |
[GitHub](https://github.com/Piyush2276)
