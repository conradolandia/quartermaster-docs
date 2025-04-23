# Quartermaster Stage 0 Technical Implementation Plan

## Initial Setup & Development Environment

### 1. Repository Setup

- Create the three core repositories on GitHub:

  ```bash
  # Create and initialize repositories
  mkdir -p quartermaster/{backend,frontend,docs}

  # For each repository
  cd quartermaster/backend
  git init
  git add README.md
  git commit -m "Initial commit"
  git branch -M main
  git remote add origin git@github.com:star-fleet/quartermaster-backend.git
  git push -u origin main

  # Create develop branch
  git checkout -b develop
  git push -u origin develop
  ```

- Set up branch protection rules for `main` and `develop` branches
  - Require pull request reviews before merging
  - Require status checks to pass
  - Restrict who can push to matching branches

### 2. Development Environment Configuration

- Clone the full-stack-fastapi-template repository and customize:

  ```bash
  # Clone FastAPI template
  git clone https://github.com/fastapi/full-stack-fastapi-template.git quartermaster
  cd quartermaster
  
  # Remove git history to start fresh
  rm -rf .git
  
  # Initialize new git repository
  git init
  git add .
  git commit -m "Initial commit from full-stack-fastapi-template"
  
  # Set remote
  git remote add origin git@github.com:star-fleet/quartermaster.git
  git push -u origin main
  
  # Create develop branch
  git checkout -b develop
  git push -u origin develop
  ```

- Customize Docker Compose for local development:

  ```bash
  # The full-stack FastAPI template already includes docker-compose.yml
  # We can make any customizations needed for our specific use case
  ```

### 3. Backend Project Customization

- The FastAPI template already includes most of what we need, but we'll need to customize it:

  ```bash
  cd quartermaster/backend
  
  # Install additional dependencies for our project
  pip install qrcode stripe

  # Update requirements.txt
  pip freeze > requirements.txt
  ```

- Adjust configuration in .env files to match our project needs:

  ```bash
  # Customize settings in .env file
  nano .env
  ```

### 4. Frontend Project Customization

- The FastAPI template already includes a React frontend with TypeScript, which we'll customize:

  ```bash
  cd quartermaster/frontend
  
  # Install additional dependencies
  npm install react-qr-code stripe @stripe/react-stripe-js @stripe/stripe-js
  ```

- Setup development workflow:

  - The template uses Vite for frontend development
  - We'll mostly customize existing components and add new ones
  

## Database Design & Implementation

### 1. Create Migration Files

- Create SQL migration files for PostgreSQL:

  ```bash
  cd quartermaster/backend
  
  # Create migrations directory if not exists
  mkdir -p alembic/versions
  
  # Create initial migration file
  alembic revision -m "initial_schema"
  ```

- Implement schema definitions in the migration files according to the database schema defined in the Stage 0 MVP document

#### Complete Database Schema

```sql
-- Admin Users Table
CREATE TABLE admin_users (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,  -- Hashed
    role VARCHAR(50) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Locations Table
CREATE TABLE locations (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Jurisdictions Table
CREATE TABLE jurisdictions (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL UNIQUE,
    state VARCHAR(100) NOT NULL,
    sales_tax_rate DECIMAL(5,2) NOT NULL,
    location_id UUID NOT NULL REFERENCES locations(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Boat Providers Table
CREATE TABLE boat_providers (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL UNIQUE,
    location_description TEXT,
    address TEXT,
    jurisdiction_id UUID NOT NULL REFERENCES jurisdictions(id),
    map_link VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Boats Table
CREATE TABLE boats (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL UNIQUE,
    capacity INTEGER NOT NULL,
    provider_id UUID NOT NULL REFERENCES boat_providers(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Launches Table
CREATE TABLE launches (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    date_time TIMESTAMP WITH TIME ZONE NOT NULL,
    location_id UUID NOT NULL REFERENCES locations(id),
    summary TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Missions Table
CREATE TABLE missions (
    id UUID PRIMARY KEY,
    launch_id UUID NOT NULL REFERENCES launches(id),
    name VARCHAR(255) NOT NULL,  -- e.g., "Falcon Heavy - Take 1"
    slug VARCHAR(255) NOT NULL UNIQUE,
    active BOOLEAN NOT NULL DEFAULT true,  -- Taking reservations
    public BOOLEAN NOT NULL DEFAULT false,  -- Visible without special link
    sales_open_at TIMESTAMP WITH TIME ZONE,
    refund_cutoff_hours INTEGER DEFAULT 12,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Trips Table
CREATE TABLE trips (
    id UUID PRIMARY KEY,
    mission_id UUID NOT NULL REFERENCES missions(id),
    type VARCHAR(50) NOT NULL,  -- "launch_viewing" or "pre_launch"
    active BOOLEAN NOT NULL DEFAULT true,
    check_in_time TIMESTAMP WITH TIME ZONE,
    boarding_time TIMESTAMP WITH TIME ZONE,
    departure_time TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Trip Boats Table
CREATE TABLE trip_boats (
    id UUID PRIMARY KEY,
    trip_id UUID NOT NULL REFERENCES trips(id),
    boat_id UUID NOT NULL REFERENCES boats(id),
    max_capacity INTEGER,  -- Optional override of boat capacity
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(trip_id, boat_id)  -- Prevent duplicate assignments
);

-- Swag Table
CREATE TABLE swag (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    default_quantity_available INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Bookings Table
CREATE TABLE bookings (
    id UUID PRIMARY KEY,
    confirmation_code VARCHAR(50) NOT NULL UNIQUE,
    mission_id UUID NOT NULL REFERENCES missions(id),
    user_name VARCHAR(255) NOT NULL,
    user_email VARCHAR(255) NOT NULL,
    user_phone VARCHAR(50) NOT NULL,
    billing_address TEXT NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) NOT NULL DEFAULT 0,
    tax_amount DECIMAL(10,2) NOT NULL,
    tip_amount DECIMAL(10,2) NOT NULL DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    payment_intent_id VARCHAR(255) NOT NULL,  -- Stripe payment intent ID
    special_requests TEXT,
    status VARCHAR(50) NOT NULL DEFAULT 'pending_payment',  -- "pending_payment", "confirmed", "checked_in", "completed", "cancelled", "refunded"
    launch_updates_preference BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Booking Items Table
CREATE TABLE booking_items (
    id UUID PRIMARY KEY,
    booking_id UUID NOT NULL REFERENCES bookings(id),
    trip_id UUID REFERENCES trips(id),  -- Optional, only for trip tickets
    boat_id UUID REFERENCES boats(id),  -- Optional, only for trip tickets
    item_type VARCHAR(50) NOT NULL,  -- "adult_ticket", "child_ticket", etc.
    quantity INTEGER NOT NULL,
    price_per_unit DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'active',  -- "active", "refunded", "fulfilled"
    refund_reason VARCHAR(100),  -- "change_in_party_size", "could_not_make_date", "unsatisfied", "other"
    refund_notes TEXT,
    refunded_at TIMESTAMP WITH TIME ZONE,
    attributes JSONB,  -- Type-specific attributes
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

### 2. Create Models

- Create SQLModel models for the database schema:

  ```bash
  cd quartermaster/backend/app
  
  # Create models directory if not exists
  mkdir -p models
  
  # Create model files
  touch models/admin_user.py
  touch models/launch.py
  touch models/mission.py
  touch models/location.py
  touch models/jurisdiction.py
  touch models/boat_provider.py
  touch models/boat.py
  touch models/trip.py
  touch models/trip_boat.py
  touch models/swag.py
  touch models/booking.py
  touch models/booking_item.py
  ```

- Implement the models using SQLModel with relationships and validations

## Core Features Implementation

### 1. Authentication Module

- Implement admin authentication using FastAPI's security features:

  ```bash
  # Create auth module
  mkdir -p app/api/endpoints
  touch app/api/endpoints/auth.py
  
  # Create dependencies for authentication
  touch app/api/dependencies/auth.py
  ```

- Configure JWT authentication with FastAPI security

### 2. Booking Module

- Create API endpoints for the booking process:

  ```bash
  # Create booking endpoints
  touch app/api/endpoints/bookings.py
  touch app/api/endpoints/launches.py
  touch app/api/endpoints/trips.py
  ```

- Implement Stripe integration service for payment processing
- Create QR code generation service

### 3. Admin Dashboard

- Create admin-specific endpoints:

  ```bash
  # Create admin endpoints
  touch app/api/endpoints/admin/dashboard.py
  touch app/api/endpoints/admin/booking_management.py
  touch app/api/endpoints/admin/export_data.py
  ```

- Implement CSV export functionality using pandas

## Frontend Components

### 1. Multi-step Booking Form

- Create the booking form components:

  ```bash
  # Create directories structure
  mkdir -p src/components/booking

  # Create form step components
  touch src/components/booking/TripDetails.svelte
  touch src/components/booking/PaymentInfo.svelte
  touch src/components/booking/Confirmation.svelte
  ```

### 2. Admin Dashboard

- Create admin dashboard components:

  ```bash
  # Create directories structure
  mkdir -p src/components/admin

  # Create admin components
  touch src/components/admin/Dashboard.svelte
  touch src/components/admin/BookingList.svelte
  touch src/components/admin/BookingDetails.svelte
  touch src/components/admin/ExportData.svelte
  ```

## Embeddable Booking Form

While the main frontend for the admin dashboard will be built using React (from the FastAPI template), we'll create a standalone Svelte form for embedding in the existing static site. This provides an early adoption of Svelte (planned for Stage 2) while limiting its scope to just the booking form component.

### 1. Svelte Form Implementation

- Create a separate SvelteKit project for the embeddable form:

  ```bash
  # Create a new SvelteKit project for the form
  npm create svelte@latest quartermaster-form
  cd quartermaster-form
  
  # Select the minimal SvelteKit template with TypeScript
  
  # Install dependencies
  npm install
  
  # Add Stripe and other necessary packages
  npm install @stripe/stripe-js
  ```

- Configure the Svelte project for producing a standalone component:

  ```bash
  # Create a rollup config that outputs a single JS file
  # This will allow the form to be embedded with a script tag
  ```

### 2. Form Structure

- Create the multi-step form in Svelte:

  ```bash
  # Create core components
  mkdir -p src/lib/components
  touch src/lib/components/BookingForm.svelte
  touch src/lib/components/TripSelection.svelte
  touch src/lib/components/PersonalDetails.svelte
  touch src/lib/components/PaymentForm.svelte
  touch src/lib/components/Confirmation.svelte
  ```

- Implement API communication with the FastAPI backend:

  ```bash
  # Create services for API interaction
  mkdir -p src/lib/services
  touch src/lib/services/api.ts
  touch src/lib/services/stripe.ts
  ```

### 3. Embedding Implementation

- Create the embedding script that initializes the form component:

  ```bash
  # Create the entry point for the embeddable component
  touch src/embed.ts
  ```

- Example implementation of the embed script:

  ```typescript
  // src/embed.ts
  import BookingForm from './lib/components/BookingForm.svelte';
  
  // Find container element
  const container = document.getElementById('booking-form-container');
  
  if (container) {
    // Extract data attributes
    const launchId = container.getAttribute('data-launch-id');
    const discountCode = container.getAttribute('data-discount-code') || null;
    
    // Initialize Svelte component
    new BookingForm({
      target: container,
      props: {
        launchId,
        discountCode
      }
    });
  }
  ```

### 4. Build and Deployment

- Configure build process for the embeddable form:

  ```bash
  # Build the embeddable script
  npm run build
  
  # Output will be a single JS file that can be included on the static site
  ```

- Example usage on the static site:

  ```html
  <!-- On the static website -->
  <div id="booking-form-container" data-launch-id="falcon9-123" data-discount-code="EARLYBIRD"></div>
  <script src="https://booking.star-fleet.tours/form.js"></script>
  ```

### 5. Integration with Backend

- Configure CORS in the FastAPI backend to allow cross-origin requests from the static site:

  ```python
  # In main.py
  from fastapi.middleware.cors import CORSMiddleware
  
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["https://star-fleet.tours", "http://localhost:3000"],
      allow_credentials=True,
      allow_methods=["*"],
      allow_headers=["*"],
  )
  ```

- Use existing API endpoints with optional authentication to handle both public and admin requests:

  ```python
  # In app/api/dependencies/auth.py
  def get_optional_current_user(
      db: Session = Depends(get_db),
      token: str = Depends(oauth2_scheme_optional)
  ) -> Optional[User]:
      """
      Similar to get_current_user but doesn't raise an exception if not authenticated.
      Returns None for unauthenticated requests, allowing public access.
      """
      if token is None:
          return None
      return get_current_user(db=db, token=token)
  
  # In app/api/endpoints/bookings.py
  @router.post("/")
  async def create_booking(
      booking_data: BookingCreate,
      db: Session = Depends(get_db),
      current_user: Optional[User] = Depends(get_optional_current_user)
  ):
      # If user is authenticated (admin), they might have additional privileges
      # If user is None (public form), apply standard validation
      
      # Process booking the same way regardless of source
      booking = create_booking_in_db(db=db, booking_data=booking_data)
      
      return booking
  ```

This approach maintains DRY principles by reusing the same endpoints for both the admin interface and the public form. It adds flexibility through optional authentication that can distinguish between authenticated admins and public users without duplicating API logic.

This approach allows us to leverage Svelte's strengths for form handling while maintaining the React-based admin interface. It also creates a smooth transition path to Stage 2, where the entire frontend will be migrated to Svelte.

## Integration Points

### 1. Stripe Integration

- Set up Stripe API keys and webhook handling:
  ```bash
  # Create Stripe service
  mkdir -p app/services
  touch app/services/stripe_service.py
  ```

### 2. Email Service Integration

- Configure SendGrid for email notifications:
  ```bash
  # Create email service
  touch app/services/email_service.py
  
  # Create email templates directory
  mkdir -p app/email-templates
  touch app/email-templates/booking_confirmation.html
  ```

### 3. Static Site Integration

- Create API endpoints for the integration with the existing static site:
  ```bash
  # Create endpoints for URL parameter handling
  touch app/api/endpoints/integration.py
  ```

### 4. Mission Configuration Mechanism

For mission configuration via YAML:

```bash
# Create service for YAML configuration handling
mkdir -p app/services
touch app/services/mission_config_service.py
```

### 5. QR Code Implementation

```bash
# Create QR code service
touch app/services/qr_code_service.py

# Create check-in endpoint
touch app/api/endpoints/admin/check_in.py
```

QR code implementation:

- Content: URL with booking ID parameter
  - Example: `https://admin.star-fleet.tours/check-in?booking=ABC123`
- Context-aware check-in system:
  - Admin interface to select active trip/boat as check-in context
  - Check-in endpoint validates booking against selected context
  - Detailed booking information displayed including:
    - Trip name and time
    - Boat name
    - Passenger count and details
    - Special requests
    - Tip amount
    - Boat statistics (X/Y passengers fulfilled)
  - Admin confirms check-in, updating relevant item status to "fulfilled"
  - Warning shown if passenger has items for different trip/boat than selected

```python
# Example check-in endpoint in FastAPI
@router.post("/set-check-in-context")
def set_check_in_context(
    context: CheckInContext,
    session: SessionDependency
):
    session["check_in_context"] = context.dict()
    return {"success": True}

@router.post("/process-check-in")
async def process_check_in(
    booking_id: str,
    session: SessionDependency,
    db: DatabaseDependency
):
    context = session.get("check_in_context")
    if not context:
        raise HTTPException(status_code=400, detail="Check-in context not set")
    
    # Fetch booking and validate against context
    # Update item status to "fulfilled" if valid
    # Return appropriate response with booking details
```

### 6. Refund Process Implementation

```bash
# Create refund management endpoint
touch app/api/endpoints/admin/refund_management.py
```

Refund process implementation:

1. Admin views booking details
2. Selects items to mark as refunded
3. Selects refund reason from dropdown:
   - Change in party size
   - Could not make launch date
   - Unsatisfied with experience
   - Other (requires explanation)
4. Enters notes in text field (required if reason is "Other")
5. System updates item status to 'refunded' and sets refunded_at timestamp
6. System calculates total refund amount
7. System automatically processes refund through Stripe API
8. System displays confirmation of successful refund

Stripe API integration for refunds:

```python
# Example refund processing in FastAPI
@router.post("/process-refund")
async def process_refund(
    refund_data: RefundRequest, 
    db: DatabaseDependency,
    stripe_service: StripeServiceDependency
):
    # Fetch booking and items
    booking = await db.get(Booking, refund_data.booking_id)
    if not booking:
        raise HTTPException(status_code=404, detail="Booking not found")
    
    items = await db.execute(
        select(BookingItem)
        .where(
            BookingItem.id.in_(refund_data.item_ids),
            BookingItem.booking_id == refund_data.booking_id
        )
    )
    items = items.scalars().all()
    
    # Calculate refund amount
    refund_amount = sum(item.price_per_unit * item.quantity for item in items)
    
    # Process refund through Stripe API
    refund = await stripe_service.create_refund(
        payment_intent=booking.payment_intent_id,
        amount=int(refund_amount * 100),  # Convert to cents
        reason="requested_by_customer" if refund_data.reason != "other" else None
    )
    
    # Update items status and refund information
    for item in items:
        item.status = "refunded"
        item.refund_reason = refund_data.reason
        item.refund_notes = refund_data.notes
        item.refunded_at = datetime.now()
    
    await db.commit()
    
    return {
        "success": True,
        "refund_id": refund.id,
        "amount": refund_amount
    }
```

### 7. Backup Strategy

```bash
# Create backup script
touch scripts/backup-database.sh
chmod +x scripts/backup-database.sh
```

Backup implementation:

- Quarterly scheduled backups
- Script exports database to SQL file with timestamp
- Verification procedure to test backup integrity
- Documentation for manual backup process before major changes

## Testing Setup

### 1. Set Up Testing Environment

- Configure testing framework:

  ```bash
  cd quartermaster/backend
  
  # Install testing dependencies
  pip install pytest pytest-asyncio httpx
  ```

### 2. Write Initial Tests

- Create tests for core functionality:
  ```bash
  # Create test files
  mkdir -p tests/api
  touch tests/api/test_bookings.py
  touch tests/api/test_auth.py
  ```

## CI/CD Setup

### 1. GitHub Actions Workflow

- Create GitHub Actions workflow files:

  ```bash
  # Create GitHub Actions directory
  mkdir -p .github/workflows

  # Create workflow files
  touch .github/workflows/ci.yml
  touch .github/workflows/deploy-dev.yml
  ```

## Development Kickoff Tasks

1. **Week 1 First Tasks**:

   - Repository setup and access for team members
   - Development environment configuration and testing
   - Database schema implementation and review
   - Initial API endpoint structure design

2. **Week 2 Tasks**:

   - Authentication system implementation
   - Basic booking form structure development
   - Stripe sandbox integration
   - Static site integration parameters finalization

3. **First Sprint Targets**:
   - Functional local development environment for all team members
   - Working database connections and migrations
   - Admin login capability
   - Skeleton API for the booking process
   - Integration point with existing site defined and tested

## Project Management Setup

1. Set up issue tracking in GitHub:

   - Create labels for different types of work (feature, bugfix, etc.)
   - Set up project boards for task management
   - Create milestone for Stage 0 MVP

2. Define weekly check-in schedule:

   - Code review sessions
   - Progress tracking meetings
   - Technical debt review

3. Create documentation structure:
   - API documentation
   - Setup instructions
   - Development guidelines

This plan provides the technical foundation for starting Stage 0 of the Quartermaster project, focusing on concrete steps to establish repositories, set up development environments, and begin implementing core features.
