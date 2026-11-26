# dMart — Architecture

This document explains how the app is structured and why. Update it whenever an architectural
decision changes.

---

## 1. Layers

MVVM with a Repository layer. Each layer only knows the layer directly below it.

```
┌─────────────────────────── Presentation ───────────────────────────┐
│  View (SwiftUI)  ──observes──▶  ViewModel (@Observable, @MainActor)│
│      │ user action ─────────▶  method call                         │
│  Shared stores: CartStore, WishlistStore (app-wide state, injected) │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │ depends on protocols only
┌─────────────────────────── Domain ─────────────────────────────────┐
│  Models: Product, AppCategory, CartItem, Order, Address            │
│  Protocols: ProductRepository, CartRepository, OrderRepository …   │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │ implemented by
┌─────────────────────────── Data ───────────────────────────────────┐
│  Repositories (impl) ─▶ Remote: ProductService ─▶ APIClient         │
│                      ─▶ Local: SwiftData, UserDefaults              │
│                      ─▶ SampleData: local grocery products/banners  │
│  DTOs (API shape)  ──Mapper──▶  Domain models (our shape)           │
└──────────────────────────────────┬──────────────────────────────────┘
┌─────────────────────────── Core ───────────────────────────────────┐
│  APIClient, APIEndpoint, NetworkError, design system, components,   │
│  extensions, formatters                                             │
└─────────────────────────────────────────────────────────────────────┘
```

| Layer | Responsibility | Must NOT |
|-------|----------------|----------|
| **View** | Render state, forward user actions to the ViewModel | Call APIs, hold business logic |
| **ViewModel** | Screen state (`LoadState`), presentation logic, call repositories, format data for the UI | Import URLSession, know about DTOs |
| **Store** (`CartStore`, `WishlistStore`) | App-wide shared state used by many screens | Contain screen-specific logic |
| **Repository** | Combine remote, local and sample data; return domain models | Know about SwiftUI |
| **Service / APIClient** | Build requests, call URLSession, decode DTOs, map HTTP errors | Know about domain models or UI |
| **Mapper** | Convert DTOs into domain models (₹ prices, MRP, app category) | Make network calls |

---

## 2. Folder structure

```
dMart/
├── App/
│   ├── DMartApp.swift            entry point
│   ├── AppDependencies.swift     composition root: builds services/repositories once
│   └── RootTabView.swift
├── Navigation/
│   ├── AppRoute.swift            enum of every pushable screen
│   └── Router.swift              @Observable, holds NavigationPath per tab
├── Core/
│   ├── Network/                  APIClient, APIEndpoint, NetworkError, HTTPMethod
│   ├── DesignSystem/             Colors, Typography, Spacing, Radius
│   ├── Components/               ProductCard, SearchBarView, PriceView, RatingView,
│   │                             PrimaryButton, QuantityStepper, LoadingView,
│   │                             EmptyStateView, ErrorStateView, CachedAsyncImage
│   ├── Extensions/
│   └── Utilities/                CurrencyFormatter, LoadState, Logger
├── Domain/
│   ├── Models/                   Product, AppCategory, CartItem, Order, Address
│   └── Repositories/             protocols only
├── Data/
│   ├── DTOs/                     ProductDTO, ProductListResponseDTO, ReviewDTO
│   ├── Mappers/                  ProductMapper, CategoryMapper
│   ├── Remote/                   ProductService (endpoints → APIClient)
│   ├── Local/                    SwiftData @Model entities, UserDefaults stores
│   ├── SampleData/               sample grocery products, banners
│   └── Repositories/             DefaultProductRepository, …
├── Features/
│   ├── Home/{Views, ViewModels, Components}
│   ├── Categories/  Products/  ProductDetails/  Cart/
│   ├── Wishlist/  Search/  Profile/  Checkout/  Orders/
├── Resources/                    Assets, sample JSON
└── Preview Content/              preview mocks
```

**Changes from the original requested structure, and why**

1. **`AppDependencies` instead of singletons** — creates the APIClient and repositories once and passes
   them down. Real dependency injection: mocks can be swapped in for tests and previews.
2. **`Router` + `AppRoute` instead of `AppCoordinator`** — coordinators come from UIKit. In SwiftUI the
   native equivalent is `NavigationStack(path:)` with a route enum: type-safe and deep-link ready.
3. **`Domain/` separate from `Data/`** — ViewModels depend only on domain models and repository
   protocols. Changing the API only touches `Data/`.
4. **`Mappers/`, `Remote/`, `Local/`, `SampleData/` instead of `Services/`** — one clear responsibility per
   folder, and API data stays visibly separate from local sample data.

---

## 3. API: DummyJSON

Checked live on 2026-11-25. Fake Store API was rejected: it only has electronics, jewelery and clothing
(about 20 products, no groceries).

**Endpoints used**

| Purpose | Endpoint |
|---------|----------|
| Product list (paged) | `GET /products?limit=&skip=&sortBy=&order=` |
| Product by id | `GET /products/{id}` |
| Products by category | `GET /products/category/{slug}` |
| Search | `GET /products/search?q=` |
| User profile | `GET /users/{id}` |

**Grocery-relevant API categories:** `groceries` (27), `kitchen-accessories` (30), `beauty` (5),
`skin-care` (3), `fragrances` (5), `home-decoration` (5) — about 75 products.

**Limitations and how we handle them**

| Limitation | Solution |
|------------|----------|
| Few grocery items (dairy, snacks, household almost empty) | Local `SampleDataSource` (~40 products). Every product has `source: .api` or `.localSample`; sample IDs start at 100000 |
| API categories ≠ DMart categories | `CategoryMapper` maps API categories/products into the 7 app categories |
| Search covers all 194 products (laptops, watches …) | Filter results to allowed API categories, then merge sample matches |
| Prices are USD-like ($9.99) | `ProductMapper` converts to ₹ with one documented constant; MRP = `price / (1 - discount%)` |
| `minimumOrderQuantity` is wholesale (e.g. 48) | Ignored |
| Reviews are dummy text | Shown as-is |
| Write endpoints are simulated, nothing is saved | Cart, wishlist, orders and addresses stored locally |
| No banners/offers API | Local sample banners |
| Images are `.webp` | Supported natively by AsyncImage |

**Category mapping**

| App category | Source |
|--------------|--------|
| Grocery (staples) | API `groceries` (rice, oil, honey, coffee …) + sample |
| Fruits & Vegetables | API `groceries` (apple, kiwi, potatoes, onions …) |
| Dairy & Bakery | API `groceries` (milk, eggs, ice cream) + sample |
| Snacks & Beverages | API `groceries` (juice, soft drinks, water) + sample |
| Personal Care | API `beauty`, `skin-care`, `fragrances` |
| Home & Kitchen | API `kitchen-accessories`, `home-decoration` |
| Household Essentials | API `groceries` (tissue box, pet food) + sample |

---

## 4. Data flow

**API → View (loading Home)**
```
HomeView.task { await viewModel.load() }
  → HomeViewModel.load()             state = .loading
    → ProductRepository.fetchHome()   (protocol)
      → DefaultProductRepository      async let for trending, deals, …
        → ProductService → APIClient.send(endpoint)
          → URLSession.data(for:)      off the main thread (nonisolated)
          ← Data → JSONDecoder → ProductListResponseDTO
        ← ProductMapper: DTO → [Product]
        + SampleDataSource products
    ← HomeContent
  state = .loaded(content)            on the MainActor
HomeView re-renders automatically (Observation tracking)
```

**User action → data (Add to Cart)**
```
ProductCard tap → onAddToCart(product) closure
  → CartStore.add(product)            @MainActor
    → CartRepository.save(...)        SwiftData insert + save
  ← items change → badge, cart screen and card button update everywhere
```

**Screen state** — every screen uses one type, so loading, empty and error are never forgotten:
```swift
enum LoadState<Value> {
    case idle, loading, loaded(Value), empty, failed(String)
}
```

---

## 5. Navigation

```
DMartApp
└── RootTabView (TabView, each tab has its own NavigationStack + Router)
    ├── Home ──▶ Search ──▶ ProductDetail
    │        ├─▶ ProductList(category) ──▶ ProductDetail ──▶ ProductDetail(related)
    │        └─▶ ProductDetail (from any card)
    ├── Categories ──▶ ProductList(category) ──▶ ProductDetail
    ├── Search ──▶ ProductDetail
    ├── Cart (badge) ──▶ Checkout: Address ▶ Summary ▶ Payment ▶ Confirmation
    │                                                   └─▶ "View order" → OrderDetail
    └── Profile ──▶ My Orders ──▶ OrderDetail
                ├─▶ Wishlist ──▶ ProductDetail
                ├─▶ Saved Addresses ──▶ Add/Edit Address
                ├─▶ Settings
                └─▶ Logout (confirmation alert)
```

---

## 6. Persistence

| Data | Storage | Why |
|------|---------|-----|
| Cart items | SwiftData | Structured, must survive relaunch |
| Wishlist | SwiftData | Structured, must survive relaunch |
| Orders | SwiftData | Structured, has line items (relationship) |
| Addresses | SwiftData | Structured, editable |
| Recent searches | UserDefaults | Small list of strings |
| Recently viewed product IDs | UserDefaults | Small list of IDs |
| Settings | UserDefaults | Simple flags |

---

## 7. Architectural decisions

| # | Decision | Reason |
|---|----------|--------|
| 1 | Minimum **iOS 17** | SwiftData and `@Observable` need iOS 17. `@StateObject` / `@ObservedObject` / `@EnvironmentObject` are still explained next to their modern replacements |
| 2 | **iPhone only** | Spec targets iPhone sizes; iPad / visionOS layouts are out of scope |
| 3 | Keep Xcode 26 **MainActor default isolation** | Prevents UI updates off the main thread. Networking and decoding are marked `nonisolated` to run in the background |
| 4 | DTOs separate from domain models | UI does not break when the API changes; the mapper is the only place that knows the API shape |
| 5 | Repository protocols + constructor injection, no singletons | Testable; mocks for previews and tests |
| 6 | Shared `CartStore` / `WishlistStore` in the environment | One source of truth for state used by many screens |
| 7 | `Decimal` for money, shown in ₹ | `Double` has rounding errors |
| 8 | No third-party packages | Native URLSession, SwiftData, AsyncImage, custom cached image loader |
| 9 | SwiftData for structured data, UserDefaults for small values | Right tool for each kind of data |
| 10 | Mock payment only | No real credentials or transactions |

---

## 8. Code quality rules

- No business logic or API calls in Views; keep Views small and split into components
- No force unwraps; use `guard let`, `if let`, `??`
- No unnecessary singletons; inject dependencies through `init`
- Access control: `private` by default, `internal` only when needed
- Protocols at layer boundaries
- UI state changes on the MainActor
- No duplicated business logic (prices, cart math live in one place)
- Every screen handles loading, empty and error states
