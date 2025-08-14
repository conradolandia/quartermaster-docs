The relationships between boats, trips, and trip boats in the Quartermaster system follow a many-to-many pattern with an association table. 

### Boat Entity
- Represents a physical vessel with properties like capacity, provider info, etc.
- A boat can be used for multiple trips (on different dates/times)
- Belongs to a jurisdiction for tax purposes
- Doesn't directly reference trips

### Trip Entity
- Represents a scheduled excursion for a specific mission (e.g., "launch viewing trip" for the "Falcon Heavy - Take 1" mission)
- Has schedule information (check-in, boarding, departure times)
- Has a trip type (either "launch_viewing" or "pre_launch")
- Belongs to a mission
- Can use multiple boats (for example, if you need more capacity)

### TripBoat Entity (Association Table)
- Implements the many-to-many relationship between trips and boats
- Contains:
  - Foreign key to Trip (`trip_id`)
  - Foreign key to Boat (`boat_id`)
  - Optional `max_capacity` to override the boat's standard capacity for a specific trip
  - Timestamps and ID

### How They Work Together

1. **Many-to-Many Relationship**:
   - A single boat can be used across multiple trips (e.g., same boat for different missions)
   - A single trip can use multiple boats (e.g., if more capacity is needed)

2. **Capacity Management**:
   - Boats have a standard capacity
   - The TripBoat association allows overriding this capacity for specific trips
   - This is useful if a boat's usable capacity varies based on trip type or safety requirements

3. **Database Relationships**:
   - Trip has `trip_boats: list["TripBoat"]` (one-to-many)
   - TripBoat has `trip: "Trip"` (many-to-one)
   - TripBoat has `boat: "Boat"` (many-to-one)

This design adds flexibility to assign boats to trips as needed, potentially using the same boat for multiple trips or multiple boats for a single trip, while managing capacity independently for each trip-boat combination.
