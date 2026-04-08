# ParkingApp

ParkingApp is now a desktop parking management application built with Java 17, Maven, Swing, and SQLite. It provides a professional operator interface for managing customers, parking spots, reservations, occupancy, and daily revenue while keeping the original domain logic.

## What Changed

- Replaced the console-first workflow with a desktop UI
- Added a dashboard with occupancy and revenue metrics
- Split persistence from business logic through repository interfaces
- Moved storage from CSV files to a local SQLite database
- Preserved service-level tests and Maven execution

## User Interface

The application starts in a control-center style desktop window with four work areas:

- `Dashboard`: total spots, availability, occupied spots, active reservations, and today's revenue
- `Customers`: register customers and search by name or phone number
- `Parking Spots`: add spots and filter by area, spot number, availability, and quick area
- `Reservations`: create reservations, complete checkout, search reservations, and review reservation history

## Architecture

- `com.parkingapp.model`: domain objects and enums
- `com.parkingapp.service`: business rules and reporting
- `com.parkingapp.repository`: persistence contracts
- `com.parkingapp.repository.sqlite`: SQLite-backed repository implementations
- `com.parkingapp.repository.csv`: legacy CSV repository implementations kept in the repo but not used by default
- `com.parkingapp.ui`: Swing desktop interface and styling

## Run

```bash
mvn compile
mvn exec:java
```

## Test

```bash
mvn test
```

## Data Storage

Runtime data is stored in SQLite under:

- `data/parkingapp.db`

You can override the database path with:

```bash
mvn exec:java -Dparkingapp.db.path=data/custom-parking.db
```

`data/` is ignored by git so local operator data does not pollute the repository.

## Validation Rules

- Customer phone numbers must be exactly 10 digits and start with `07`
- The customer phone input only accepts digits and blocks entry beyond 10 characters
- Customer search currently filters by `name` and `phone number`

## Class Diagram

```mermaid
classDiagram
    class Main {
        +main(String[] args)
    }

    class ParkingAppFrame
    class UiTheme

    class ParkingService {
        +registerCustomer(String name, String phoneNumber, String plateNumber) Customer
        +addParkingSpot(int areaCode, int spotNumber) void
        +getAllParkingSpots() List~ParkingSpot~
        +getAvailableSpotsByArea(int areaCode) List~ParkingSpot~
        +searchParkingSpots(Integer areaCode, Integer spotNumber, Boolean availableOnly) List~ParkingSpot~
        +createReservation(int customerId, int areaCode, int spotNumber, int durationHours) Reservation
        +completeReservation(String reservationId) Reservation
        +findCustomerByPhoneNumber(String phoneNumber) Customer
        +searchCustomers(String nameQuery, String phoneQuery, String plateQuery) List~Customer~
        +findCustomer(int customerId) Customer
        +findParkingSpot(int areaCode, int spotNumber) ParkingSpot
        +findReservation(String reservationId) Reservation
        +getAllCustomers() List~Customer~
        +getOccupiedSpots() List~ParkingSpot~
        +getAllReservations() List~Reservation~
        +getActiveReservations() List~Reservation~
        +searchReservations(String reservationIdQuery, String plateQuery, Integer areaCode, Integer spotNumber) List~Reservation~
        +calculateParkingFee(Reservation reservation) double
        +getTotalSpotsCount() int
        +getAvailableSpotsCount() int
        +getOccupiedSpotsCount() int
        +getActiveReservationCount() int
        +getTodayRevenue() double
        +ensureDefaultParkingSpots() void
    }

    class CustomerRepository
    class ParkingSpotRepository
    class ReservationRepository
    class SQLiteCustomerRepository
    class SQLiteParkingSpotRepository
    class SQLiteReservationRepository
    class SQLiteDatabase
    class CsvCustomerRepository
    class CsvParkingSpotRepository
    class CsvReservationRepository

    class Customer {
        -int customerId
        -String name
        -String phoneNumber
        -Vehicle vehicle
    }

    class Vehicle {
        -String plateNumber
    }

    class ParkingSpot {
        -int areaCode
        -int spotNumber
        -SpotStatus status
        +isAvailable() boolean
        +occupy() void
        +vacate() void
    }

    class Reservation {
        -String reservationId
        -Customer customer
        -ParkingSpot parkingSpot
        -LocalDateTime startTime
        -LocalDateTime endTime
        -ReservationStatus status
        -LocalDateTime completedAt
        +complete(LocalDateTime completedAt) void
    }

    class ReservationIdGenerator {
        +generate() String
    }

    class ReservationStatus {
        <<enumeration>>
        ACTIVE
        COMPLETED
    }

    class SpotStatus {
        <<enumeration>>
        AVAILABLE
        OCCUPIED
    }

    Main --> ParkingService
    Main --> ParkingAppFrame
    ParkingAppFrame --> ParkingService
    ParkingAppFrame --> UiTheme
    ParkingService --> CustomerRepository
    ParkingService --> ParkingSpotRepository
    ParkingService --> ReservationRepository
    ParkingService --> ReservationIdGenerator
    SQLiteCustomerRepository ..|> CustomerRepository
    SQLiteParkingSpotRepository ..|> ParkingSpotRepository
    SQLiteReservationRepository ..|> ReservationRepository
    SQLiteCustomerRepository --> SQLiteDatabase
    SQLiteParkingSpotRepository --> SQLiteDatabase
    SQLiteReservationRepository --> SQLiteDatabase
    CsvCustomerRepository ..|> CustomerRepository
    CsvParkingSpotRepository ..|> ParkingSpotRepository
    CsvReservationRepository ..|> ReservationRepository
    Customer *-- Vehicle
    ParkingSpot --> SpotStatus
    Reservation --> Customer
    Reservation --> ParkingSpot
    Reservation --> ReservationStatus
```

The PlantUML source is also available in `docs/class-diagram.puml`.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
______________________________________
Author
Fadi Yosef
