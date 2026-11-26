# dMart

A DMart-style grocery shopping app for iPhone, built with **Swift, SwiftUI and MVVM**.
A personal project to learn how a production iOS app is designed, structured, built and maintained.

## Features
- **Home** — search bar, promotional banners, categories, trending, recommended, best deals, recently viewed
- **Categories** — Grocery, Fruits & Vegetables, Dairy & Bakery, Snacks & Beverages, Personal Care,
  Home & Kitchen, Household Essentials
- **Product listing** — prices, discount %, rating, availability, add to cart, sort and filter
- **Product details** — image gallery, description, reviews, quantity, related products
- **Cart** — quantity changes, subtotal, discount, delivery charge, payable amount
- **Wishlist** — save products, move to cart
- **Search** — debounced search, recent searches, empty states
- **Profile** — user info, orders, wishlist, saved addresses, settings, logout
- **Checkout & orders** — address selection, order summary, mock payment, order history

> Payment is a **mock flow only**. No real payment credentials or transactions are used.

## Tech stack
| Area | Choice |
|------|--------|
| Language | Swift |
| UI | SwiftUI |
| Architecture | MVVM + Repository |
| Networking | URLSession + async/await |
| Parsing | Codable (DTOs mapped to domain models) |
| State | `@Observable` (Observation framework) |
| Navigation | `NavigationStack` + route enum |
| Local storage | SwiftData (cart, wishlist, orders, addresses), UserDefaults (recent searches, settings) |
| Images | AsyncImage, later a cached image loader |
| Dependencies | Swift Package Manager — no third-party packages planned |
| Minimum iOS | 17.0 (iPhone only) |

## Data
Products come from the free [DummyJSON](https://dummyjson.com/products) API. Grocery items that the API
does not have (dairy, snacks, household) come from a **local sample-data layer**. Every product is
tagged with its source (`api` or `localSample`).

## Requirements
- Xcode 26 or later
- iOS 17.0+ simulator or device

## Getting started
1. Clone the repo and check out `develop`
2. Open `dMart.xcodeproj`
3. Select an iPhone simulator and run (⌘R)

## Docs
- [ROADMAP.md](ROADMAP.md) — phases, tasks and progress
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — architecture, data flow, navigation and decisions
