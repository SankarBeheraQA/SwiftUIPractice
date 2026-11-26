# dMart — Development Roadmap

A DMart-style grocery shopping app built with **Swift, SwiftUI and MVVM**, step by step, to learn how a
production iOS app is designed, built and maintained.

- Architecture and design decisions: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- Tick a box (`- [x]`) when a task is done and verified.
- Work on **one phase at a time**. A phase is approved before the next one starts.

---

## Working agreement

1. Before coding: inspect existing files, list every file to create or modify.
2. Explain the concept first, then implement.
3. Every phase ends with a **compile check** (`xcodebuild`, generic simulator destination) and a
   **manual test in Xcode**.
4. Never claim a feature is done without verifying it.
5. Ask for approval before starting the next phase.

For every feature: overview · architecture · files · complete code · explanation of key blocks ·
execution flow · expected UI · testing steps · common mistakes.

---

## Branching & tracking

```
main      ← stable, tagged releases (v0.1, v0.2 …)
develop   ← integration branch, always builds
feature/<JIRA-KEY>-<short-name>   ← one per Jira story, merged into develop via PR
```

Tracking: Jira (Epic = phase, Story = task).

Workflow per story:
1. Move the Jira story to **In Progress**
2. `git checkout develop && git pull`
3. `git checkout -b feature/DM-2-project-settings`
4. Build + commit small steps
5. Push, open PR into `develop` (title starts with the Jira key), self-review, merge
6. Move the story to **Done**, tick the box below
7. When a phase is done: merge `develop` → `main` and tag a version

Commit format: `DM-2 Set deployment target to iOS 17`

---

## Progress

| #  | Phase                        | Status |
|----|------------------------------|--------|
| 01 | Project setup                | ⬜ |
| 02 | Design system                | ⬜ |
| 03 | Networking layer             | ⬜ |
| 04 | Home screen                  | ⬜ |
| 05 | Categories & product listing | ⬜ |
| 06 | Product details              | ⬜ |
| 07 | Cart & wishlist              | ⬜ |
| 08 | Search                       | ⬜ |
| 09 | Profile & checkout           | ⬜ |
| 10 | Persistence & refinement     | ⬜ |
| 11 | Testing                      | ⬜ |
| 12 | Final code review            | ⬜ |
| 13 | Advanced (optional)          | ⬜ |

Status: ⬜ not started · 🟨 in progress · ✅ done

---

## Phase 01 — Project setup
**Learn:** Xcode project settings, app lifecycle, `@main`, `App`/`Scene`, MVVM overview, access control,
MainActor default isolation.
- [ ] Commit tracking files (`.gitignore`, README, ROADMAP, docs) to `develop`
- [ ] Set minimum iOS version to **17.0**
- [ ] Set bundle id (`com.sankar.dmart`) and display name `dMart`
- [ ] Set device family to **iPhone only**
- [ ] Create folder structure (App, Navigation, Core, Domain, Data, Features, Resources, Preview Content)
- [ ] Rename entry point to `DMartApp.swift`, add `AppDependencies.swift` (composition root)
- [ ] Add `RootTabView` with 5 tabs: Home, Categories, Search, Cart, Profile (placeholder screens)
- [ ] Remove template `ContentView.swift`
- [ ] Build succeeds; app launches to the tab bar

## Phase 02 — Design system
**Learn:** view composition, `ViewModifier`, `@ViewBuilder`, closures as callbacks, previews, color assets
with dark mode.
- [ ] Colors (asset catalog, light/dark): brand green, accent, background, surface, text, discount, error
- [ ] Typography scale (`Font` extension)
- [ ] Spacing and corner radius constants
- [ ] `PrimaryButton` / `SecondaryButton`
- [ ] `PriceView` (MRP strike-through, price, discount %)
- [ ] `RatingView`
- [ ] `QuantityStepper`
- [ ] `SearchBarView` (editable + tappable read-only mode for Home)
- [ ] `ProductCard` (image, name, prices, rating, availability, Add to Cart / stepper)
- [ ] `LoadingView`, `EmptyStateView`, `ErrorStateView` (with retry)
- [ ] `SectionHeader` ("Trending" + "See all")
- [ ] Previews for every component in light and dark mode

## Phase 03 — Networking layer
**Learn:** protocols, generics, `Codable` / `CodingKeys`, async/await, `throws`, error mapping,
`nonisolated`, DTO vs domain model.
- [ ] Verify DummyJSON response structure (`/products`, `/products/{id}`, `/products/category/{slug}`,
      `/products/search`)
- [ ] `HTTPMethod`, `APIEndpoint` protocol, `ProductEndpoint` enum
- [ ] `APIClient` protocol + `URLSessionAPIClient` (generic `send<T: Decodable>`)
- [ ] `NetworkError` with user-friendly messages
- [ ] DTOs: `ProductDTO`, `ProductListResponseDTO`, `ReviewDTO`
- [ ] Domain models: `Product`, `AppCategory`, `Review`, `DataSource` (`.api` / `.localSample`)
- [ ] `ProductMapper` (₹ conversion, MRP, availability) and `CategoryMapper` (API → 7 app categories)
- [ ] `SampleDataSource` for missing grocery items (IDs ≥ 100000)
- [ ] `ProductService` + `ProductRepository` protocol + `DefaultProductRepository`
- [ ] `MockProductRepository` for previews
- [ ] Debug-only request logging
- [ ] Temporary test screen proves the API call works

## Phase 04 — Home screen
**Learn:** `@Observable`, `@State`, `.task`, `async let`, `LoadState`, dependency injection through `init`,
full data flow API → Repository → ViewModel → View.
- [ ] `LoadState<Value>` enum
- [ ] `HomeViewModel` loading sections in parallel
- [ ] Search bar (opens Search tab/screen)
- [ ] Banner carousel (sample banners)
- [ ] Category grid
- [ ] Trending (top rated), Best Deals (highest discount), Recommended
- [ ] Recently Viewed section (empty until Phase 6)
- [ ] Loading, error (retry) and pull-to-refresh states

## Phase 05 — Categories & product listing
**Learn:** `NavigationStack`, `Router` + `AppRoute`, `LazyVGrid`, `@Binding`, sorting/filtering in the
ViewModel.
- [ ] `AppRoute` enum + `Router` (one per tab)
- [ ] Categories screen: 7 DMart categories with images
- [ ] Product listing screen by category (API + sample products)
- [ ] Sort: price low→high, high→low, popularity, discount
- [ ] Filter: discount range, price range, in stock only
- [ ] Filter/sort sheet
- [ ] Empty state when filters match nothing

## Phase 06 — Product details
**Learn:** passing data vs IDs through navigation, async detail loading, local view state vs ViewModel state.
- [ ] Image gallery (paging)
- [ ] Name, brand, description
- [ ] Price and discount block
- [ ] Rating summary and reviews list
- [ ] Quantity selection
- [ ] Add to Cart (calls the cart; cart itself built in Phase 7)
- [ ] Related products (same category)
- [ ] Record Recently Viewed (UserDefaults) and show it on Home

## Phase 07 — Cart & wishlist
**Learn:** SwiftData (`@Model`, `ModelContainer`, `ModelContext`), shared state with `@Environment`,
`Decimal` money math.
- [ ] SwiftData entities: `CartItemEntity`, `WishlistItemEntity`
- [ ] `CartRepository` / `WishlistRepository` (protocol + SwiftData implementation)
- [ ] `CartStore` and `WishlistStore` (`@Observable`) injected into the environment
- [ ] Cart screen: list, increase/decrease, remove, empty state
- [ ] Price summary: subtotal, product savings, coupon discount, delivery charge (free above ₹500), payable amount
- [ ] Cart badge on the tab bar
- [ ] Wishlist: heart toggle on cards and details, wishlist screen, move to cart
- [ ] Cart and wishlist survive app relaunch

## Phase 08 — Search
**Learn:** `Task` cancellation, debouncing with `Task.sleep`, `.searchable`, UserDefaults.
- [ ] Search screen with debounced query
- [ ] API search filtered to grocery categories + sample data matches
- [ ] Recent searches (save, tap to reuse, clear)
- [ ] Loading, empty ("No results for …") and error states

## Phase 09 — Profile & checkout
**Learn:** forms and validation, multi-step flows, enums for state machines, alerts and confirmation dialogs.
- [ ] Profile screen: user info (DummyJSON `/users/1`), My Orders, Wishlist, Saved Addresses, Settings, Logout
- [ ] Address management: add, edit, delete, set default (SwiftData)
- [ ] Checkout: address selection → order summary → payment method
- [ ] **Mock payment** (UPI / Card / COD UI only — no real credentials or transactions)
- [ ] Place order → generate order (SwiftData) → clear cart
- [ ] Order confirmation screen
- [ ] Order history and order details
- [ ] Logout with confirmation (clears user data)

## Phase 10 — Persistence & refinement
**Learn:** app lifecycle and relaunch, image caching (`NSCache`), performance, accessibility, animations.
- [ ] Review what is persisted and where (SwiftData vs UserDefaults)
- [ ] Relaunch scenarios: cart, wishlist, orders, recent searches, recently viewed
- [ ] `CachedAsyncImage` with memory cache
- [ ] Performance pass (lazy stacks, avoid extra re-renders)
- [ ] Accessibility: VoiceOver labels, Dynamic Type, contrast
- [ ] Meaningful animations (add to cart, badge, transitions)
- [ ] Test on small (iPhone SE) and large (Pro Max) screens

## Phase 11 — Testing
**Learn:** XCTest, test doubles (mocks/stubs), `URLProtocol` stubbing, async tests, UI tests.
- [ ] Add unit test target
- [ ] ViewModel tests with mock repositories (loading, success, empty, error)
- [ ] Repository tests (mapping, category merge, search filtering)
- [ ] Networking tests with stubbed `URLProtocol`
- [ ] Cart calculation tests (subtotal, discount, delivery, payable)
- [ ] UI test: browse → add to cart → checkout → order confirmation

## Phase 12 — Final code review
- [ ] Architecture review (layers respected, no API calls in Views)
- [ ] Code duplication
- [ ] Memory management (retain cycles, task lifetimes)
- [ ] Force unwraps and unsafe code
- [ ] Unnecessary dependencies and singletons
- [ ] Performance review
- [ ] Production-quality checklist and final cleanup

## Phase 13 — Advanced (optional)
Extra topics so the project covers the full iOS skill set.
- [ ] One UIKit screen embedded with `UIViewControllerRepresentable`
- [ ] Real login with DummyJSON `/auth/login` + Keychain token storage
- [ ] Local notifications for order status + deep links
- [ ] Home screen widget (WidgetKit) for latest order
- [ ] Localization (English + Hindi) with String Catalog
- [ ] GitHub Actions CI (build + test on PR)

---

## Learning notes
Write down what you learned after each phase — this becomes your interview prep.

| Phase | Key learnings | Problems I hit |
|-------|---------------|----------------|
| 01 | | |
| 02 | | |
| 03 | | |
