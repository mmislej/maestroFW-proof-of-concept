# Maestro UI Testing — Proof of Concept

![Maestro Tests](https://github.com/mmislej/maestroFW-proof-of-concept/actions/workflows/maestro.yml/badge.svg)
[![Test Report](https://img.shields.io/badge/Test%20Report-Allure-blue)](https://mmislej.github.io/maestroFW-proof-of-concept)

A mobile UI test automation proof of concept using [Maestro](https://maestro.mobile.dev), demonstrating how to build a scalable test framework **without access to the app's source code**.

The app under test is [provider_shopper](https://github.com/flutter/samples/tree/main/provider_shopper), a Flutter sample app featuring a login screen, product catalog, and shopping cart.

---

## What This Project Demonstrates

- **Black-box UI testing** — tests written purely from a user perspective, no source code access required
- **Reusable action architecture** — modular YAML subflows that eliminate code duplication across tests
- **Parameterized actions** — dynamic flows driven by environment variables
- **Element inspection** — using Maestro Studio to locate UI elements without code access
- **CI/CD integration** — automated test execution on every push via GitHub Actions

---

## Project Structure

```
├── actions/                           # Reusable subflows (atomic UI actions)
│   ├── do_login.yaml                  # Fills credentials and submits the login form
│   ├── add_items_and_go_to_cart.yaml  # Adds N items to cart and navigates to it
│   └── assert_cart_visible.yaml       # Verifies the Cart screen is displayed
│
├── tests/                             # End-to-end test scenarios
│   ├── add_to_cart.yaml               # Add item → verify price → tap BUY
│   ├── remove_from_cart.yaml          # Add item → remove it → verify total is $0
│   └── multiple_items.yaml            # Add 3 items → verify names and total price
│
├── samples/provider_shopper/          # Flutter app under test
├── .github/workflows/maestro.yml      # GitHub Actions CI pipeline
├── FLUTTER_SETUP.md                   # Flutter + iOS Simulator setup guide
└── MAESTRO_SETUP.md                   # Maestro installation guide
```

---

## Test Architecture: Action-Based Pattern

Instead of the traditional Page Object Model (POM) — which relies on classes and methods — this project uses **composable YAML subflows**. Each file in `actions/` represents an atomic interaction that can be called from any test using `runFlow`.

This approach keeps tests readable, maintainable, and DRY: if a UI flow changes, only the corresponding action file needs to be updated.

### How a test composes actions

```yaml
# tests/add_to_cart.yaml
appId: dev.flutter.providerShopper
---
- launchApp
- runFlow:
    file: ../actions/do_login.yaml
    env:
      USERNAME: "user"
      PASSWORD: "pass"
- runFlow:
    file: ../actions/add_items_and_go_to_cart.yaml
    env:
      ADD_ITEMS_COUNT: 1
- runFlow: ../actions/assert_cart_visible.yaml
- assertVisible: "Code Smell"
- assertVisible: "$42"
- tapOn: "BUY"
- assertVisible: "Buying not supported yet."
- stopApp
```

---

## Test Scenarios

| Test | Scenario |
|------|----------|
| `add_to_cart` | Login → add 1 item → verify price → tap BUY → verify confirmation |
| `remove_from_cart` | Login → add 1 item → remove it from cart → verify total is $0 |
| `multiple_items` | Login → add 3 items → verify all item names → verify total price ($126) |

---

## Setup

**1. Get the app**
```bash
git clone https://github.com/flutter/samples.git
cd samples/provider_shopper
flutter pub get
flutter run
```

**2. Install Maestro**
```bash
curl -fsSL "https://get.maestro.mobile.dev" | bash
```

## Running the Tests

**Prerequisites:** iPhone Simulator running with the app installed.

```bash
# Run all tests
maestro test tests/

# Run a single test
maestro test tests/add_to_cart.yaml

# Inspect UI elements visually
maestro studio
```

---

## CI Pipeline & Test Report

The `.github/workflows/maestro.yml` workflow runs on every push and pull request to `master`.

**Pipeline steps:**
1. Install Flutter and app dependencies
2. Boot iPhone Simulator / Android Emulator
3. Build and install the app
4. Install Maestro
5. Run all test suites and generate JUnit XML report
6. Generate Allure HTML report and publish to GitHub Pages
7. Upload debug artifacts (screenshots + logs) on failure

**Live test report:** [mmislej.github.io/maestroFW-proof-of-concept](https://mmislej.github.io/maestroFW-proof-of-concept)

[**📊 View Test Report →**](https://mmislej.github.io/maestroFW-proof-of-concept)

---

## Handling Elements Without Accessibility Labels

Since this project simulates a **no source code access** scenario, some UI elements (icon buttons without labels) are targeted using relative screen coordinates obtained via Maestro Studio.

| Element | Selector used |
|---------|---------------|
| Shopping cart icon | `point: "94%,10%"` |
| Remove item button | `point: "80%,20%"` |

Using percentage-based coordinates makes selectors resolution-independent across different simulator sizes.
