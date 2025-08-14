# Quartermaster Stage 0: Minimal Viable Product (MVP) - Simplified

## Overview

Stage 0 focuses on delivering a streamlined booking system that integrates with the existing Star✦Fleet Tours static website. This MVP will enable users to book tickets for upcoming launches, process payments securely, and receive confirmation with digital tickets. The admin interface will provide basic management capabilities for bookings and passenger manifests.

## Timeline

**Duration**: 1-2 months

- Weeks 1-2: Setup & Core Structure
- Weeks 3-5: Booking System & Admin View
- Weeks 6-8: Testing & Refinement

## Core Functionality

### User-Facing Features

#### 1. Booking Form Integration

- Integration with existing static site via URL parameters
- Parameters: launch ID and optional discount code
- Example URL: `https://book.star-fleet.tours/?launch=falcon9-123`
- Single page displays all booking options (missions and their trips)
- Mobile-first responsive design

#### 2. Booking Process

- **Step 0**: Launch Selection (via existing static site)
- **Step 1**: Trip Details Entry
  - Mission selection (if multiple missions exist for the launch)
  - Number of tickets selection
  - Boat selection (if multiple options)
  - Discount code field
  - Optional tip field with suggested amounts
  - Special requests text area
- **Step 2**: Confirmation and Payment
  - Itemized cost breakdown
  - Payment formula: [Subtotal - Discount] × (1 + Tax Rate) + Tip
  - Personal details collection (name, email, phone)
  - Billing address collection
  - Stripe payment processing
  - Terms and conditions acceptance
- **Step 3**: Confirmation/Ticket Delivery
  - Confirmation page with booking details
  - QR code ticket generation
  - Email confirmation with booking details
  - Persistent URL for accessing booking details

#### 3. Digital Tickets

- QR code ticket accessible via:
  - Confirmation page
  - Email link
  - Booking lookup (confirmation code + last name)
- QR code contains a URL link with booking ID parameter
- Example: `https://admin.star-fleet.tours/check-in?booking=ABC123`
- Admin selects active trip/boat before scanning to establish check-in context
- System validates if passenger has items for the selected trip/boat
- On successful validation, system updates relevant item status to "fulfilled"

### Admin-Facing Features

#### 1. Authentication System

- Secure admin login
- No public user accounts in MVP
- Password reset functionality

#### 2. Dashboard

- Summary statistics:
  - Total bookings
  - Revenue metrics
  - Passenger counts per trip
- Table of all bookings with:
  - Filtering options (date, refund_status, etc.)
  - Basic search functionality
  - Sorting capabilities

#### 3. Booking Management

- View booking details
- Context-aware check-in process:
  - Admin selects active trip/boat to establish check-in context
  - Admin scans QR code or enters booking ID manually
  - System validates booking against selected trip/boat
  - System displays detailed booking information and statistics
  - Admin confirms check-in, updating item status to "fulfilled"
- Simple "mark as refunded" functionality:
  - Select items to mark as refunded
  - Select refund reason from dropdown
  - Enter explanation notes (required for "Other" reason)
  - System automatically processes refund through Stripe API
  - Display confirmation with refund details

#### 4. CSV Export

- Export passenger manifest data to CSV
- Filterable by trip/mission
- Standard fields: name, email, phone, ticket quantity, boat

#### 5. Email Notifications

- Simple email confirmation for new bookings
- Basic email templates for booking confirmations

#### 6. Configuration

- Configure launches, missions, and trips via YAML file
- Basic configuration parameters:
  - Launch information (name, date/time, location)
  - Mission details (name, active status, public visibility, sales opening time, refund cutoff hours)
  - Trip details (type, times, boat assignments)
  - Ticket pricing

## Technical Implementation

### Backend (FastAPI)

#### Core Components

1. **Authentication Module**

   - Admin login system
   - JWT token implementation
   - Password management

2. **Booking Module**

   - Booking creation and validation
   - Payment processing integration
   - Confirmation generation
   - QR code creation

3. **Admin Module**

   - Dashboard data aggregation
   - Booking management endpoints
   - CSV export functionality
   - Configuration from YAML

4. **Email Module**
   
   - SendGrid integration
   - Basic email templates
   - Transactional email delivery

#### Database Schema (PostgreSQL)

1. **Admin Users Table**

   - ID (UUID), name, username, email, password (hashed), role, created_at, updated_at

2. **Launches Table**

   - ID (UUID), name, slug, date_time, location_id, summary, created_at, updated_at

3. **Missions Table**

   - ID (UUID), launch_id, name, slug, active, public, sales_open_at, refund_cutoff_hours, created_at, updated_at

4. **Locations Table**

   - ID (UUID), name, slug, state, created_at, updated_at

5. **Jurisdictions Table**

   - ID (UUID), name, slug, state, sales_tax_rate, location_id, created_at, updated_at

6. **Boats Table** (simplified)

   - ID (UUID), name, slug, capacity, provider_name, provider_location, provider_address, jurisdiction_id, map_link, created_at, updated_at

7. **Trips Table**

   - ID (UUID), mission_id, type (launch_viewing/pre_launch), active, check_in_time, boarding_time, departure_time, created_at, updated_at

8. **Trip Boats Table**

   - ID (UUID), trip_id, boat_id, max_capacity (optional override), created_at, updated_at

9. **Bookings Table**

   - ID (UUID), confirmation_code, mission_id, user_name, user_email, user_phone, billing_address, subtotal, discount_amount, tax_amount, tip_amount, total_amount, payment_intent_id, special_requests, status, launch_updates_preference, created_at, updated_at

10. **Booking Items Table**
    
   - ID (UUID), booking_id, trip_id, boat_id, item_type, quantity, price_per_unit, status, refund_reason, refund_notes, created_at, updated_at

### Frontend (React)

#### Core Components

1. **Booking Form**

   - Multi-step form implementation
   - Real-time validation
   - Responsive design for mobile and desktop
   - Ticket selection interface
   - Integrated Stripe Elements

2. **Confirmation Page**

   - Booking summary display
   - QR code generation
   - Print-friendly layout
   - Save-to-device functionality

3. **Admin Dashboard**

   - Responsive dashboard layout
   - Data visualization components
   - Booking table with filtering
   - CSV export interface

4. **Booking Lookup**
   - Simple form for retrieving booking by confirmation code + name
   - Ticket display and management

#### Design System

1. **Component Library**

   - Chakra UI components with custom styling
   - Consistent color scheme and typography
   - Responsive grid system
   - Form components with validation states

2. **Responsive Design**
   - Mobile-first approach
   - Breakpoints for tablet and desktop
   - Optimized layouts for different device sizes

## Implementation Priorities

1. Core booking form with Stripe integration
2. Confirmation page with QR code functionality
3. Basic admin dashboard with booking table and counts
4. CSV export functionality
5. Simple email confirmations
6. YAML configuration for launches, missions, and trips
7. Simple check-in and refund marking functionality
8. Tip functionality in booking form

## Testing Approach

1. **Unit Testing**

   - Core business logic
   - API endpoints
   - Form validation

2. **Integration Testing**

   - Booking flow
   - Payment processing
   - Email delivery

3. **User Acceptance Testing**
   - Admin dashboard functionality
   - Booking process
   - Mobile compatibility

## Deployment Plan

1. **Infrastructure Setup**

   - VPS configuration
   - Database setup
   - Domain and SSL configuration

2. **Deployment Process**

   - Git-based deployment
   - Docker Compose for service orchestration
   - Uvicorn as the FastAPI process manager
   - Database migration

## Benefits of the Simplified Approach

1. **Focused Development**
   - Core functionality prioritized
   - Essential entity relationships maintained
   - Fewer unnecessary fields to manage

2. **Domain Integrity**
   - Proper distinction between launches and missions maintained
   - Jurisdiction-based tax handling preserved
   - Clear separation of concerns in data model

3. **Easier Maintainability**
   - Streamlined database schema with proper relationships
   - Simplified attribute set for each entity
   - Clearer data flow through system

4. **Faster Deployment**
   - Field reduction allows quicker implementation
   - YAML configuration simplifies setup
   - Focused feature set speeds delivery

5. **Clear Migration Path**
   - Foundation established for future enhancement
   - Core domain model already in place
   - Critical relationships already established 