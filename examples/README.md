# GreenCart — System Requirements Documentation

> This document is the input to a performance test design exercise. It intentionally makes no reference to "Operational Profiles," test cases, or test design — it describes the system the way a business analyst and architect actually would.

**Document type:** Combined Business Requirements, Functional Requirements, Non-Functional Requirements, and Architecture Overview  
**Version:** 3.2  
**Date:** 2026-05-18  
**Status:** Approved for release planning

## 1. Business Overview

GreenCart is an online grocery delivery platform operating in three metropolitan regions. Customers order groceries via a web storefront or mobile app; orders are picked and packed at regional fulfillment centers ("warehouses") and delivered within customer-selected time slots. GreenCart has been live for 14 months and currently serves approximately 210,000 registered customer accounts across its three regions.

### 1.1 Business Goals for This Release Cycle

- **BG-1:** Reduce cart-to-delivery time by streamlining the checkout and delivery-slot-selection flow.
- **BG-2:** Support the upcoming Q3 regional expansion, which is projected to increase overall order volume by approximately 40% within 6 months of launch.
- **BG-3:** Improve warehouse staff tooling to reduce order-picking errors, which currently run at ~2.1% of orders requiring a post-pick correction.
- **BG-4:** Maintain nightly inventory accuracy across all three regional warehouses without extending the current overnight processing window.

## 2. Actors

| Actor                           | Type                       | Description                                                                                                                                                                   |
| ------------------------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Registered Customer             | Primary, human             | A customer with a GreenCart account, saved payment methods, and order history. Places orders via web or mobile app.                                                           |
| Guest Customer                  | Primary, human             | A customer checking out without creating an account. Web only (guest checkout is not currently supported in the mobile app).                                                  |
| Warehouse Picker/Packer         | Primary, human             | Warehouse staff who receive picking assignments, scan items, pack orders, and hand off to the delivery dispatch queue.                                                        |
| Delivery Dispatcher             | Primary, human             | Warehouse staff who assign packed orders to delivery drivers and confirm dispatch. (Low headcount, small transaction volume — noted here for completeness.)                   |
| Regional Inventory Sync Service | Secondary, external system | An overnight batch process that reconciles each warehouse's physical stock counts (from the warehouse management system, WMS) against GreenCart's central inventory database. |
| Payment Gateway                 | Secondary, external system | Third-party payment processor (PCI-compliant, hosted). GreenCart's backend calls out to it; GreenCart does not store card data directly.                                      |
| SMS/Push Notification Provider  | Secondary, external system | Third-party service used to send order-status notifications. Fire-and-forget from GreenCart's perspective.                                                                    |

## 3. Functional Requirements

### 3.1 Product Discovery

- **FR-1.1:** Customers can browse products by category.
- **FR-1.2:** Customers can search products by keyword, with autocomplete suggestions.
- **FR-1.3:** Product listings show real-time stock availability per the customer's selected region/warehouse.

### 3.2 Cart & Checkout

- **FR-2.1:** Customers can add/remove items and adjust quantities in a persistent cart.
- **FR-2.2:** Customers select a delivery time slot from those available at their region's warehouse for the current day or next 3 days.
- **FR-2.3:** Registered customers may apply a saved payment method or a promotional code at checkout.
- **FR-2.4:** Guest customers must enter payment details manually each time (no saved payment methods).
- **FR-2.5:** On checkout completion, the system reserves inventory, submits payment authorization, and creates an order record.

### 3.3 Order Fulfillment (Warehouse Staff)

- **FR-3.1:** Warehouse Pickers receive a picking list generated from newly placed orders, grouped by delivery slot.
- **FR-3.2:** Pickers scan each item's barcode to confirm the pick; the system validates the scanned item against the order line.
- **FR-3.3:** Once all items are picked, the order moves to a packing queue; packing staff confirm packaging completion.
- **FR-3.4:** Delivery Dispatchers assign packed orders to available drivers and confirm dispatch, triggering a customer notification.

### 3.4 Inventory Reconciliation

- **FR-4.1:** Each night between 01:00 and 04:00 local warehouse time, the Regional Inventory Sync Service reconciles WMS-reported physical stock counts against GreenCart's central inventory database for all three regions.
- **FR-4.2:** Discrepancies beyond a configured tolerance are flagged for morning review by warehouse operations staff (this review flow is out of scope for this document — it's a low-volume manual process).

## 4. Non-Functional Requirements

### 4.1 Response Time & Throughput (Customer-Facing)

- **NFR-4.1.1:** Product browsing and search pages must render with a P95 response time of 600ms or less under normal peak load.
- **NFR-4.1.2:** Add to Cart must complete with a P95 response time of 400ms or less.
- **NFR-4.1.3:** Checkout completion (from "Place Order" click to confirmation) must complete with a P95 response time of 3 seconds or less, and a P99 of no more than 6 seconds.
- **NFR-4.1.4:** The system must sustain at least 1,800 concurrent active shopping sessions across all regions during evening peak hours (6–9pm local time) without breaching the above targets.
- **NFR-4.1.5:** During the Q3 regional expansion launch (see BG-2), the system must additionally sustain a projected 40% increase in concurrent sessions and order throughput within 6 months of expansion launch, without requiring a redesign of the checkout flow.

### 4.2 Response Time & Throughput (Warehouse Staff Tooling)

- **NFR-4.2.1:** The item-scan confirmation action (FR-3.2) must respond within a P95 of 300ms — pickers perform this action dozens of times per order and delays compound across a shift.
- **NFR-4.2.2:** The picking-list generation action (FR-3.1) must respond within a P95 of 2 seconds even during the highest-volume order-placement periods of the day.

### 4.3 Payment Gateway Constraint

- **NFR-4.3.1:** The third-party Payment Gateway contractually guarantees a response within 8 seconds for 99% of requests, and enforces a hard 12-second timeout on GreenCart's connection. This is an external constraint, not a GreenCart-controlled target — checkout completion's P99 target (NFR-4.1.3) must account for it.

### 4.4 Availability & Reliability

- **NFR-4.4.1:** Customer-facing services must maintain 99.9% availability during business hours (6am–midnight local time, all regions).
- **NFR-4.4.2:** The nightly Inventory Reconciliation process (FR-4.1) must complete within its 01:00–04:00 window with no more than a 0.5% record-level failure rate requiring manual intervention.
- **NFR-4.4.3:** Warehouse staff tooling must maintain 99.5% availability during active warehouse operating hours (5am–11pm local time).

### 4.5 Error Rate

- **NFR-4.5.1:** Customer-facing checkout-path transactions (Cart, Checkout, Payment) must maintain an error rate of 0.1% or lower.
- **NFR-4.5.2:** Browse/Search transactions must maintain an error rate of 0.5% or lower.
- **NFR-4.5.3:** Warehouse scan confirmations must maintain an error rate of 0.2% or lower (a failed scan blocks a physical worker mid-task, so reliability here has an outsized operational cost relative to its raw transaction volume).

## 5. Architecture Overview

### 5.1 Tiers and Protocols

- Customer-facing web app and mobile app communicate with a backend API Gateway over HTTPS/REST.
- The API Gateway routes to internal microservices: Product Catalog Service, Cart Service, Order Service, Inventory Service — all HTTPS/REST internally.
- Order Service calls the external Payment Gateway over HTTPS/REST (see NFR-4.3.1 for its timeout behavior).
- Order Service publishes order-status events to a message queue (AMQP) consumed by: the Notification Service (which calls the external SMS/Push Provider, fire-and-forget), and the Warehouse Management System integration layer.
- Warehouse handheld scanner devices communicate with a dedicated Warehouse API (HTTPS/REST, separate from the customer-facing API Gateway, though backed by some of the same internal services — notably Inventory Service).
- The Regional Inventory Sync Service runs as a scheduled batch job (not user-triggered) that connects to each region's WMS via a batch file transfer (SFTP-delivered CSV extracts) and reconciles against the central Inventory Service's database via direct JDBC access (a legacy integration pattern predating the REST API layer, retained for this release).

### 5.2 Known Architectural Constraints

- The central Inventory Database is a single primary instance with read replicas per region; all three regions' nightly reconciliation jobs write to the same primary, which has historically caused lock contention when reconciliation windows overlap across regions (regions are in different time zones, so their 01:00–04:00 local windows only partially overlap in UTC).
- The Order Service's checkout code path has not been load tested since a major rework 3 months ago that changed how inventory reservation is handled during checkout (previously optimistic, now pessimistic locking) — this is a recent, untested change.
- Auto-scaling is configured for the API Gateway and Product Catalog Service (scales on CPU utilization, evaluated every 3 minutes) but is explicitly NOT configured for the Order Service or Inventory Service, which run on a fixed-size cluster this release.

## 6. Usage Data Available

- Product analytics platform (90-day rolling window) covering browse/search/cart behavior for both Registered and Guest customers.
- APM tooling (full transaction-level tracing) covering all customer-facing and warehouse-facing API traffic for the trailing 90 days.
- WMS-side operational logs covering picking/packing throughput per warehouse for the trailing 6 months.
- No dedicated analytics currently exist for the Inventory Reconciliation batch job beyond basic job-duration and success/failure logging — this is noted as a data gap for whoever picks this system up for performance test design.

## 7. Out of Scope for This Release

- Loyalty points / rewards program (planned for a future release, not yet built).
- Subscription/recurring order functionality.
- The morning discrepancy-review flow mentioned in FR-4.2 (low-volume, manual, not performance-sensitive).
