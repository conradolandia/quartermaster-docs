# Quartermaster Data Model - Simplified for MVP

> **Note**: This document details the simplified database schema and data specifications for the Quartermaster booking system MVP. For a high-level system overview, see [System Overview](./system_overview.md).

## Form fields for user booking

- Phase 0: Launch selection
  - Select launch to book (done on static side via clicking appropriate link/button)
- Phase 1: Trip details
  - Mission selection (if multiple attempts exist for the launch)
  - Number of tickets
  - Boat selection (if offered)
  - Other
    - Discount code (can be percent based, or for a fixed amount)
    - Optional tip (dollar amount)
      - CRITICAL: Check for negative tip frontend and backend and cap at 0!
    - Order Comments/special needs
  - Page displays current subtotal, e.g. top right
- Phase 2: Confirm and pay
  - Itemized list of charges
    - Subtotal
    - Discount (percent/amount)
    - Tip
    - Sales tax, set per-jurisdiction based on the location
    - Total due
  - Personal details
    - First and last name for reservation
    - Email
      - Note it will be used for updates on the launch/trip
      - Checkbox if they would like updates about future launches
    - Phone
  - Card details
    - Stripe widget
    - Name on card
  - Billing address
    - Country
    - Line 1
    - Line 2
    - City
    - State/province
    - Postcode
  - Agreement box(es)
    - Check box that they understand terms and conditions, scrub policy and refund policy (with links to these and brief summary)
    - Disable submission if not checked
- Phase 3: Confirmation/Ticket Page
  - Persistent URL with their last name, confirmation code and mission as params
  - Title: Reservation is confirmed (or cancelled if so)
  - QR Code ticket (leads to endpoint on admin page with their details as params)
  - Confirmation message
  - Their name
  - Boat name, address, checkin/boarding/departure date/time
  - Itemized list of what they booked w/cost

## Simplified Data Model for MVP

For the MVP, we're simplifying the data model to focus on essential functionality. We've reduced unnecessary fields while maintaining important entity distinctions.

### Launch data

- [x] **ID** (auto-generated UUID)
- [x] **Name** (e.g., "SpaceX Starship Flight 9")
- [x] **Slug** (auto-generated from name)
- [x] **Launch timestamp** (when the rocket launches)
- [x] **Summary** (short one-line description)
- [x] **Location_id** (reference to location table)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Location data

- [x] **ID** (e.g. `cape`, `starbase`)
- [x] **Name**
- [x] **Slug** (auto-generated from name)
- [x] **State** (e.g., Florida, Texas)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Jurisdiction data

- [x] **ID** (e.g. `port_canaveral`, `spi`)
- [x] **Name**
- [x] **Slug** (auto-generated from name)
- [x] **State** (e.g., Florida, Texas)
- [x] **Sales tax rate**
- [x] **Location_id** (reference to location table)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Mission data

- [x] **ID** (auto-generated UUID)
- [x] **Launch_id** (connection to associated launch)
- [x] **Name** (e.g., "Falcon Heavy - Take 1")
- [x] **Slug** (auto-generated from name)
- [x] **Active flag** (whether taking reservations)
- [x] **Public flag** (whether shown to the public)
- [x] **Sales open at** (timestamp to open public tickets)
- [x] **Refund cutoff hours** (hours before launch when refunds unavailable, default: 12)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Trip data

- [x] **ID** (auto-generated UUID)
- [x] **Mission_id** (foreign key to missions table)
- [x] **Type** (launch_viewing or pre_launch)
- [x] **Active flag** (whether taking reservations)
- [x] **Check-in time**
- [x] **Boarding time**
- [x] **Departure time**
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Boats data (simplified)

- [x] **ID** (auto-generated UUID)
- [x] **Name**
- [x] **Slug** (auto-generated from name)
- [x] **Capacity** (maximum passenger capacity)
- [x] **Provider name**
- [x] **Provider location**
- [x] **Provider address**
- [x] **Jurisdiction_id** (foreign key to jurisdictions table)
- [x] **Map link**
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Trip Boats data

- [x] **ID** (auto-generated UUID)
- [x] **Trip_id** (foreign key to trips table)
- [x] **Boat_id** (foreign key to boats table)
- [x] **Max capacity** (optional override of boat's standard capacity)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Booking data

- [x] **ID** (auto-generated UUID)
- [x] **Confirmation code** (unique identifier for customer reference)
- [x] **Mission_id** (foreign key to missions table)
- [x] **User name** (full name for reservation)
- [x] **User email**
- [x] **User phone**
- [x] **Billing address** (full address as text)
- [x] **Subtotal** (sum of all items before discounts and taxes)
- [x] **Discount amount** (total discount applied)
- [x] **Tax amount** (calculated sales tax)
- [x] **Tip amount** (optional tip, non-negative value only)
- [x] **Total amount** (final amount charged)
- [x] **Payment intent ID** (Stripe reference)
- [x] **Special requests** (customer comments or special needs)
- [x] **Status** (pending_payment, confirmed, checked_in, completed, cancelled, refunded)
- [x] **Launch updates preference** (whether customer wants updates about future launches)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

### Booking Items data

- [x] **ID** (auto-generated UUID)
- [x] **Booking_id** (foreign key to bookings table)
- [x] **Trip_id** (foreign key to trips table)
- [x] **Boat_id** (foreign key to boats table)
- [x] **Item type** (adult_ticket, child_ticket)
- [x] **Quantity**
- [x] **Price per unit**
- [x] **Status** (active, refunded, fulfilled)
- [x] **Refund reason** (optional)
- [x] **Refund notes** (optional)
- [x] **created_at** (timestamp)
- [x] **updated_at** (timestamp)

## Capacity Tracking

Real-time capacity tracking is implemented through the following mechanism:

1. Each trip is associated with one or more boats through the trip_boats table
2. Each boat has a maximum capacity, which can be overridden at the trip level
3. When a booking is made:
   - The system checks available capacity before confirming
   - Capacity is reduced by the number of tickets purchased
4. When a booking is cancelled or refunded:
   - Capacity is restored for the affected trip/boat
5. Admin interface provides real-time capacity monitoring:
   - Total capacity per trip/boat
   - Booked capacity
   - Available capacity

## QR Code Implementation

QR codes for tickets:

1. **Content**: URL with booking ID parameter
   - Example: `https://admin.star-fleet.tours/check-in?booking=ABC123`
2. **Generation**: Created upon successful booking completion
3. **Delivery**: Embedded in confirmation page and email
4. **Validation**: Admin selects active trip/boat context before scanning
5. **Check-in process**:
   - QR code is scanned
   - System validates booking against selected trip/boat context
   - On successful validation, relevant item status updated to "fulfilled"

## Data Retention

1. **User data**:
   - Basic contact information retained for operational communications
   - For guest checkouts: sensitive data (address, phone) purged after one year
2. **Payment information**:
   - No credit card details stored in our system (handled by Stripe)
   - Payment references retained for refund processing
3. **Deletion requests**:
   - Manual process via contact form (MVP) 
