# Quartermaster - System Overview (Simplified for MVP)

> **Note**: This document provides a high-level overview of the simplified Quartermaster booking system MVP. For detailed technical specifications, see [Data Model Simplified](./data_model_simplified.md) and [Implementation Plan Simplified](./stages/stage0_mvp_simplified.md).

## About Star✦Fleet Tours

We're a collective of space fans organizing educational trips and launch experiences, and our continuing mission is to inspire kids and adults alike to dream big, work hard and keep looking toward the stars.

It all started with a simple idea: what if a group of space fans chartered a boat to get the best view possible of the first Falcon Heavy launch and landing? Since those humble beginnings with a few dozen enthusiasts, interest in the concept has taken off, with hundreds of kids, adults and families from all around the world going out with us, as well as attracting media attention and interest from other local organizations.

Our core mission: to share the wonder and excitement of spaceflight with people of all ages, backgrounds and previous launch experiences.

## Trips Overview

Each mission represents Star Fleet's organized viewing opportunity for a specific launch attempt. A mission contains one or more trips: typically a launch viewing trip and optional pre-launch trips.

Each trip has scheduled times: one for the boat's departure and another determined by the mission's launch time.

- Cancellation Policy for Registered Users:
  - Before the boat departs (or before a designated safety threshold), cancellations are eligible for a refund.
  - Once the boat has sailed or the safety threshold has been crossed, refunds cannot be issued because payment for the boat has already been processed.

## Entity Hierarchy and Relationships

The core hierarchy and relationship between key entities is:

1. **Launches**

   - Represents the actual rocket launch event (e.g., "SpaceX Starship Flight 9")
   - Each Launch belongs to a Location (launch site)
   - The external entity that Star Fleet missions revolve around

2. **Missions**

   - Star Fleet's organized viewing opportunity for a specific launch attempt
   - If a launch scrubs (cancels after boats go out), a new Mission is created for the next attempt
   - Each Mission references a specific Launch
   - Has its own active status, public visibility, and sales opening time
   - The central organizing entity for Star Fleet operations

3. **Trips**

   - A specific boat excursion within a Mission
   - Two main types:
     - Launch Viewing Trip: To watch the actual rocket launch
     - Pre-Launch Trip: To view the rocket on the pad before launch
   - Each Trip references a specific Mission
   - Trips have specific details like boat assignments, departure times, capacity

4. **Locations**

   - Represents a launch site (e.g., Cape Canaveral, Starbase)
   - Referenced by Launches

5. **Jurisdictions**

   - Represents legal/tax jurisdiction (e.g., Port Canaveral, South Padre Island)
   - Each Jurisdiction is associated with a Location
   - Contains sales tax information

6. **Bookings**

   - A customer's reservation record
   - Contains items for one specific mission
   - Contains payment information, passenger details, status
   - Links to specific items through BookingItems

7. **BookingItems**

   - Individual line items within a Booking
   - References specific Trips and Boats
   - Includes quantity, price, and status information
   - Uses the status field to track refunds and fulfillment

This hierarchy ensures that:

- Multiple Mission attempts for the same Launch can be properly tracked
- Proper tax rates are applied based on Jurisdiction
- Trips are correctly associated with specific Mission attempts
- Bookings are made at the Mission level, with items potentially spanning multiple Trips
- Refund and fulfillment status can be tracked at the individual item level

## Description of requirements

We need a new bookings website with an administrative interface so users can buy tickets to scheduled trips, and admins can manage bookings, issue refunds and check in passengers.

For the MVP, we'll create a simple booking form that integrates with our existing static site.

We will use Stripe for payments and refunds.

## Tech stack

We have selected the following tech stack:

### Backend: FastAPI (Python)

[FastAPI](https://fastapi.tiangolo.com/) provides:

- **Speed of Development**: Automatic docs, type validation, and async support
- **Type Safety**: Python type hints and Pydantic for data validation
- **SQLModel ORM**: Modern ORM for working with PostgreSQL
- **Authentication System**: JWT-based authentication
- **Email Service**: Email sending via SendGrid

### Database: PostgreSQL

PostgreSQL is our chosen database for its reliability and features.

### Frontend

For the MVP, we'll use React + TypeScript + Chakra UI based on the FastAPI full-stack template.

### Additional Infrastructure

- **Authentication**: JWT-based authentication for admin users
- **API Documentation**: Automatic OpenAPI docs via FastAPI
- **Email Service**: SendGrid integration
- **Deployment**: Docker Compose with Uvicorn and Traefik
- **Monitoring**: Sentry for error tracking

## Functionality requirements for the MVP

### Public-Facing Website

- Integration with existing static site via parameters to select launch/mission/trip
- Booking form for selected trips
- Confirmation page with booking details and QR code

### Passenger-Facing

- Access booking details via confirmation code + last name or confirmation link
- View ticket information and QR code

### Admin Interface

- Simple dashboard showing:
  - Table of all bookings
  - Passenger counts per trip
  - Revenue metrics
- Basic CSV export of passenger manifest data
- Simple email notifications to users
- Basic search for bookings by confirmation code, name, email
- Admin forms for creating and managing launches, missions, and trips
- Simple buttons to mark passengers as "fulfilled" or "refunded"

## Data Model Specifications

### Launch Data

- Launch name (e.g., `SpaceX Starship Flight 9`)
- Launch ID (auto-generated)
- Launch timestamp
- Summary (short description)
- Location reference

### Mission Data

- Mission ID (auto-generated)
- Launch reference
- Name (e.g., "Falcon Heavy - Take 1")
- Active flag (taking reservations)
- Public flag (visible without special link)
- Sales open timestamp
- Refund cutoff hours (default: 12)

### Location Data

- ID (e.g., `cape`, `starbase`)
- Name
- State (e.g. Florida, Texas)

### Jurisdiction Data

- ID (e.g., `port_canaveral`, `spi`)
- Name
- Location reference
- Sales tax rate

### Trip Data

- ID
- Mission ID (connection to associated mission)
- Type (launch_viewing or pre_launch)
- Active flag (taking reservations)
- Check-in, boarding, departure times
- Available boats with capacity

### Boat Data

- ID
- Name
- Maximum passenger capacity
- Provider information (name, location, address)
- Jurisdiction reference
- Map link

### Booking Data

- Mission ID (what mission this booking is for)
- Ticket quantities
- Boat selections
- Payment details:
  - Discount codes (if applicable)
  - Tips (non-negative values only)
  - Special requests
  - Total charges and tax
- Status tracking for individual items:
  - `active` - Default state
  - `refunded` - Item has been refunded
  - `fulfilled` - Item has been fulfilled (checked-in)

## Booking Process Flow

### Step 0: Launch/Mission Selection

- User selects launch from existing static site (for MVP)
- System identifies active mission for that launch

### Step 1: Trip Details Entry

- Number of tickets for Launch Viewing Trip
- Boat selection (if multiple options)
- Special requests or needs
- Discount code application
- Optional tip (with validation to ensure non-negative values)

### Step 2: Confirmation and Payment

- Itemized cost breakdown (subtotal, discount, tip, tax, total)
  - Formula: [Subtotal - Discount] × (1 + Tax Rate) + Tip
- Personal details entry
  - Name
  - Email
  - Phone
  - Billing address
- Payment processing via Stripe
- Terms and conditions acceptance

### Step 3: Confirmation/Ticket Delivery

- Confirmation page with booking details
- QR code ticket generation
- Email confirmation with persistent URL for booking access

## QR Code Implementation

For the MVP, QR codes generated for tickets will:

- Contain a URL link with booking ID parameter
- Example: `https://admin.star-fleet.tours/check-in?booking=ABC123`
- Allow for context-aware check-in by admin users
  - Admin selects active trip/boat before scanning
  - System validates booking against selected trip/boat
  - Warning shown if mismatch occurs
- Update relevant item status to "fulfilled" upon successful check-in

## Security Considerations

- Implement comprehensive authentication system for admin accounts
- Ensure data protection through encryption
- Implement API security best practices
- Follow Stripe security best practices for payment processing

## Development Timeline

### Phase 0: Minimal MVP (1-2 months)

#### Weeks 1-2: Setup & Core Structure

- Project structure and repository setup
- Database schema definition for essential tables
- Basic API endpoints for bookings
- Admin authentication system

#### Weeks 3-5: Booking System & Admin View

- Implement Stripe payment integration
- Create booking form with validation
- Build confirmation page with QR code
- Develop basic admin dashboard
- Implement CSV export of booking data
- Create simple email confirmation system

#### Weeks 6-8: Testing & Refinement

- Testing and bug fixes
- Deployment to production
- Integration with existing static site
- Documentation for admins

## Implementation Priorities

1. Booking form with Stripe integration
2. Confirmation page with QR code
3. Basic admin dashboard with booking table
4. CSV export functionality
5. Simple email confirmations
6. Admin interface for launch/mission/trip management
7. Simple "mark as fulfilled" and "mark as refunded" admin functionality
8. Tip functionality in booking form

## Benefits of the Simplified Approach

### 1. Reduced Development Time

- **Focused entity model** with proper domain distinctions
- **Streamlined UI requirements** targeting core user needs
- **Reduced validation complexity** focusing on essential fields
- **Clear domain model** respecting important business distinctions

### 2. Lower Complexity

- **Well-structured data flow** with focused entity relationships
- **Proper domain separation** between launches/missions and locations/jurisdictions
- **Reduced edge cases** by simplifying non-essential features
- **Easier to understand** for both developers and non-technical stakeholders

### 3. Easier Maintainability

- **Optimized SQL queries** for common operations
- **Clear entity relationships** for important domain concepts
- **Reduced points of failure** with streamlined functionality
- **More straightforward debugging** with clearer data flow

### 4. Faster Deployment

- **Streamlined admin forms** for entity management
- **Reduced testing scope** focusing on core functionality
- **Well-defined deployment goals** with essential features prioritized
- **Maintained domain integrity** for future expansion

### 5. Improved User Experience

- **Intuitive booking flow** with proper domain terminology
- **Focused admin interface** targeting essential operations
- **Faster page loads** due to simplified data requirements
- **Direct customer interactions** with streamlined booking process

## Future Expansion Potential

The simplified MVP design intentionally allows for expansion in future phases:

1. **Launch and Mission separation** already maintained to support tracking multiple missions per launch
2. **Location and Jurisdiction separation** already maintained for proper tax handling
3. **Boat and Provider full separation** can be implemented when more detailed provider management is needed
4. **Swag sales functionality** can be added by extending the booking items table
5. **User accounts system** can be implemented without disrupting the guest checkout flow

This approach allows us to deliver a functional MVP quickly while still maintaining important domain distinctions for future development.

## Implementation Order

1. Locations & Jurisdictions first
  - These are foundational reference data
  - Relatively simple with few dependencies
2. Launches
  - Build on Locations
  - Core entity that Missions depend on
3. Missions
  - Depend on Launches
  - Critical for organizing the business logic
4. Boats
  - Independent from the above flow
  - Can be implemented in parallel
5. Trips
  - Depend on Missions and Boats
  - Connect the physical resources to events
  
------------------ WE'RE HERE ------------------

6. Bookings & BookingItems
  - Final layer that ties everything together
  - Most complex with dependencies on all above entities

### Missing/Incomplete Features
- [x] CRITICAL: Email Notifications: Missing email confirmation to customers
- [ ] CRITICAL: Payment Integration: The Stripe payment flow is not implemented (marked with TODO)
- [ ] CSV Export: No endpoint for exporting passenger manifests
- [ ] Advanced Filtering: The list_bookings endpoint lacks filtering by status, date, name, etc.
- [ ] Check-in Endpoint: Missing the actual endpoint that would be triggered when scanning a QR code
- [ ] Refund Processing: No dedicated endpoint for processing refunds through Stripe
