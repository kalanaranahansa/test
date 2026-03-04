# UML Diagrams – Online Hotel Reservation System

## System Overview

The **Online Reservation System** is a web-based hotel booking application built with Java/JSP and MySQL. It allows guests to browse available rooms, make reservations, manage their bookings, and receive a bill. An administrator can manage the room inventory and handle booking confirmations or cancellations.

---

## 1. Use Case Diagram

### PlantUML Source

```plantuml
@startuml Use_Case_Diagram
left to right direction
skinparam actorStyle awesome
skinparam packageStyle rectangle

actor "Guest (Customer)" as Guest
actor "Admin" as Admin

rectangle "Online Reservation System" {

  ' ---- Authentication ----
  package "Authentication" {
    usecase "Register / Sign Up" as UC_Register
    usecase "Login" as UC_Login
    usecase "Logout" as UC_Logout
    usecase "Forgot Password" as UC_ForgotPass
    usecase "Change Password" as UC_ChangePass
    usecase "Change Mobile Number" as UC_ChangeMobile
    usecase "Change Security Question" as UC_ChangeSQ
  }

  ' ---- Room Browsing ----
  package "Room Browsing" {
    usecase "Browse Available Rooms" as UC_Browse
    usecase "Search Rooms" as UC_Search
  }

  ' ---- Reservation ----
  package "Reservation Management" {
    usecase "Add Room to Cart" as UC_AddCart
    usecase "View Cart" as UC_ViewCart
    usecase "Remove Room from Cart" as UC_RemoveCart
    usecase "Increase / Decrease Nights" as UC_IncDec
    usecase "Enter Address & Payment" as UC_Payment
    usecase "Place Booking" as UC_PlaceOrder
    usecase "View Bill" as UC_Bill
    usecase "Print Bill" as UC_Print
    usecase "View My Bookings" as UC_MyOrders
  }

  ' ---- Communication ----
  package "Communication" {
    usecase "Contact Us (Message)" as UC_Message
  }

  ' ---- Admin Functions ----
  package "Admin Panel" {
    usecase "Manage Products (Rooms)" as UC_ManageProducts
    usecase "Add New Room" as UC_AddProduct
    usecase "Edit Room Details" as UC_EditProduct
    usecase "View Booking Requests" as UC_ViewOrders
    usecase "Confirm Booking" as UC_Confirm
    usecase "Cancel Booking" as UC_CancelOrder
    usecase "View Delivered/Confirmed Bookings" as UC_ViewDelivered
    usecase "View Messages from Guests" as UC_ViewMessages
  }
}

' Guest associations
Guest --> UC_Register
Guest --> UC_Login
Guest --> UC_Logout
Guest --> UC_ForgotPass
Guest --> UC_ChangePass
Guest --> UC_ChangeMobile
Guest --> UC_ChangeSQ
Guest --> UC_Browse
Guest --> UC_Search
Guest --> UC_AddCart
Guest --> UC_ViewCart
Guest --> UC_RemoveCart
Guest --> UC_IncDec
Guest --> UC_Payment
Guest --> UC_PlaceOrder
Guest --> UC_Bill
Guest --> UC_Print
Guest --> UC_MyOrders
Guest --> UC_Message

' Admin associations
Admin --> UC_Login
Admin --> UC_Logout
Admin --> UC_ManageProducts
Admin --> UC_AddProduct
Admin --> UC_EditProduct
Admin --> UC_ViewOrders
Admin --> UC_Confirm
Admin --> UC_CancelOrder
Admin --> UC_ViewDelivered
Admin --> UC_ViewMessages

' Include relationships
UC_PlaceOrder .> UC_Payment : <<include>>
UC_Bill .> UC_PlaceOrder : <<include>>
UC_Print .> UC_Bill : <<include>>
UC_AddProduct .> UC_ManageProducts : <<include>>
UC_EditProduct .> UC_ManageProducts : <<include>>

' Extend relationships
UC_AddCart .> UC_Login : <<extend>>
UC_PlaceOrder .> UC_Login : <<extend>>

@enduml
```

### Design Decisions

| Decision | Rationale |
|---|---|
| Two primary actors (Guest, Admin) | The system has clearly separate roles: a Guest interacts with booking flows, the Admin manages inventory and processes bookings. |
| Authentication package | All identity-related use cases (login, register, change password, etc.) are grouped to highlight the security boundary. |
| `<<include>>` on PlaceBooking → Address & Payment | Providing address and selecting a payment method is a **mandatory** sub-step whenever a booking is placed. |
| `<<extend>>` on Add-to-Cart / Place-Booking → Login | A guest must be authenticated before performing reservation actions; unauthenticated access extends the Login use case. |
| Admin operates independently of Guest flows | The Admin panel is a completely separate UI path (`/admin/*`), ensuring separation of concerns. |

---

## 2. Class Diagram

### PlantUML Source

```plantuml
@startuml Class_Diagram
skinparam classAttributeIconSize 0
skinparam classFontStyle bold

' ---- Utility ----
class ConnectionProvider {
  +{static} getCon() : Connection
}

' ---- Domain Model ----
class User {
  -guestId : int
  -guestName : String
  -email : String
  -password : String
  -address : String
  -city : String
  -state : String
  -country : String
  -contactNumber : String
  -roomType : String
  -checkInDate : Date
  -checkOutDate : Date
  +register() : void
  +login() : boolean
  +logout() : void
  +changePassword(newPassword : String) : void
  +changeMobileNumber(newNumber : String) : void
  +updateAddress(address : String, city : String, state : String, country : String) : void
}

class Product {
  -id : int
  -name : String
  -category : String
  -price : int
  -active : String
  +getAvailableRooms() : List<Product>
  +searchByName(query : String) : List<Product>
}

class Cart {
  -email : String
  -roomId : int
  -numberOfNights : int
  -price : int
  -total : int
  -address : String
  -city : String
  -state : String
  -country : String
  -contactNumber : String
  -checkInDate : Date
  -checkOutDate : Date
  -paymentMethod : String
  -transactionId : String
  -status : String
  +addItem(email : String, roomId : int, price : int) : void
  +removeItem(email : String, roomId : int) : void
  +updateNights(email : String, roomId : int, nights : int) : void
  +checkout(address : String, paymentMethod : String, transactionId : String) : void
  +getCartItems(email : String) : List<Cart>
}

class Booking {
  ' Booking is the Cart record after checkout (status = "processing"/"delivered")
  -email : String
  -roomId : int
  -numberOfNights : int
  -price : int
  -total : int
  -address : String
  -checkInDate : Date
  -checkOutDate : Date
  -paymentMethod : String
  -transactionId : String
  -status : String
  +confirmBooking(email : String, roomId : int) : void
  +cancelBooking(email : String, roomId : int) : void
  +getBookingsByStatus(status : String) : List<Booking>
}

class Message {
  -id : int
  -email : String
  -subject : String
  -body : String
  +sendMessage(email : String, subject : String, body : String) : void
  +getAllMessages() : List<Message>
}

' ---- Controller / JSP Action Pages ----
class LoginAction <<JSP>> {
  +processLogin(email : String, password : String) : void
}

class SignupAction <<JSP>> {
  +processSignup(guestName : String, email : String, password : String, ...) : void
}

class AddToCartAction <<JSP>> {
  +addToCart(email : String, roomId : String) : void
}

class AddressPaymentAction <<JSP>> {
  +processCheckout(email : String, address : String, paymentMethod : String, transactionId : String) : void
}

class AdminOrdersAction <<JSP>> {
  +confirmBooking(roomId : String, email : String) : void
  +cancelBooking(roomId : String, email : String) : void
}

class AdminProductAction <<JSP>> {
  +addProduct(id : String, name : String, category : String, price : String, active : String) : void
  +editProduct(id : String, name : String, category : String, price : String, active : String) : void
}

' ---- Relationships ----
User "1" --> "0..*" Cart : places
User "1" --> "0..*" Booking : has
User "1" --> "0..*" Message : sends

Product "1" --> "0..*" Cart : reserved in
Product "1" --> "0..*" Booking : booked as

Cart ..> ConnectionProvider : uses
Booking ..> ConnectionProvider : uses
User ..> ConnectionProvider : uses
Product ..> ConnectionProvider : uses
Message ..> ConnectionProvider : uses

LoginAction ..> User : authenticates
SignupAction ..> User : creates
AddToCartAction ..> Cart : manages
AddressPaymentAction ..> Cart : updates
AddressPaymentAction ..> User : updates
AdminOrdersAction ..> Booking : manages
AdminProductAction ..> Product : manages

@enduml
```

### Design Decisions

| Decision | Rationale |
|---|---|
| `ConnectionProvider` as a static utility class | Centralises JDBC connection creation (Singleton-like pattern), ensuring consistent driver loading and database URL across all JSP pages. |
| `Cart` and `Booking` modelled separately (conceptually) | Although stored in the same `cart` table, a record transitions through statuses (`NULL address → bill → processing → delivered/cancel`). Separating them conceptually clarifies the lifecycle. |
| `Product` as a first-class entity | Rooms/products are independently managed by the admin (add, edit, deactivate) and referenced by Cart/Booking, so a standalone class avoids duplication. |
| `Message` as a standalone class | Guest feedback/queries are unrelated to booking flows and stored independently, making them easy to extend (e.g., reply functionality) without affecting core booking logic. |
| JSP pages modelled as `<<JSP>>` controller classes | JSP action pages act as thin controllers (like Servlets), bridging UI input to domain-model operations. This mirrors an MVC pattern without a formal framework. |

---

## 3. Sequence Diagram

Three key scenarios are illustrated below.

---

### 3a. Guest Registration and Login

```plantuml
@startuml Sequence_Registration_Login
actor Guest
participant "signup.jsp" as SignupPage
participant "signupAction.jsp" as SignupAction
participant "ConnectionProvider" as CP
database "MySQL DB\n(user table)" as DB
participant "login.jsp" as LoginPage
participant "loginAction.jsp" as LoginAction

== Registration ==
Guest -> SignupPage : Fill registration form\n(name, email, password, address, contact,\nroom_type, check_in, check_out)
SignupPage -> SignupAction : POST form data
SignupAction -> CP : getCon()
CP -> DB : JDBC connect
DB --> CP : Connection
SignupAction -> DB : INSERT INTO user (...)
DB --> SignupAction : success
SignupAction --> Guest : redirect signup.jsp?msg=valid

== Login ==
Guest -> LoginPage : Enter email & password
LoginPage -> LoginAction : POST credentials
LoginAction -> CP : getCon()
CP -> DB : JDBC connect
DB --> CP : Connection

alt Admin login (email = admin@gmail.com)
  LoginAction --> Guest : redirect admin/adminHome.jsp\n[session email set]
else Guest login
  LoginAction -> DB : SELECT * FROM user WHERE email=? AND password=?
  DB --> LoginAction : ResultSet
  alt User found
    LoginAction --> Guest : redirect home.jsp\n[session email set]
  else Not found
    LoginAction --> Guest : redirect login.jsp?msg=notexist
  end
end
@enduml
```

---

### 3b. Room Browsing, Add to Cart, and Place Booking

```plantuml
@startuml Sequence_Booking_Flow
actor Guest
participant "home.jsp" as Home
participant "addToCartAction.jsp" as AddCart
participant "myCart.jsp" as MyCart
participant "addressPaymentForOrder.jsp" as AddrPage
participant "addressPaymentForOrderAction.jsp" as AddrAction
participant "bill.jsp" as BillPage
participant "ConnectionProvider" as CP
database "MySQL DB" as DB

== Browse Rooms ==
Guest -> Home : View available rooms
Home -> CP : getCon()
CP -> DB : JDBC connect
DB --> CP : Connection
Home -> DB : SELECT * FROM product WHERE active='yes'
DB --> Home : Room list
Home --> Guest : Display room cards

== Add Room to Cart ==
Guest -> AddCart : Click "Book Now" (room id)
AddCart -> CP : getCon()
AddCart -> DB : SELECT * FROM product WHERE id=?
DB --> AddCart : room price
AddCart -> DB : SELECT * FROM cart WHERE room_id=? AND email=? AND address IS NULL
DB --> AddCart : existing cart row (if any)

alt Room already in cart
  AddCart -> DB : UPDATE cart SET total=?, Number_of_Nights=? WHERE room_id=? AND email=?
  AddCart --> Guest : redirect home.jsp?msg=exist
else New cart entry
  AddCart -> DB : INSERT INTO cart (email, room_id, Number_of_Nights, price, total)
  AddCart --> Guest : redirect home.jsp?msg=added
end

== View Cart & Checkout ==
Guest -> MyCart : View cart
MyCart -> DB : SELECT cart JOIN product WHERE email=? AND address IS NULL
DB --> MyCart : Cart items with room details
MyCart --> Guest : Display cart summary

Guest -> AddrPage : Proceed to Checkout
Guest -> AddrAction : Submit address & payment details\n(address, city, state, country, contact, paymentMethod, transactionId)
AddrAction -> CP : getCon()
AddrAction -> DB : UPDATE user SET address, city, state, country, contact WHERE email=?
AddrAction -> DB : UPDATE cart SET address, city, ..., check_in_date=NOW(),\ncheck_out_date=DATE_ADD(NOW(), INTERVAL 3 DAY),\npaymentMethod, transactionId, status='bill'\nWHERE email=? AND address IS NULL
DB --> AddrAction : success
AddrAction --> Guest : redirect bill.jsp

== View Bill ==
Guest -> BillPage : View booking confirmation
BillPage -> DB : SELECT SUM(total) FROM cart WHERE email=? AND status='bill'
BillPage -> DB : SELECT user JOIN cart WHERE email=? AND status='bill'
BillPage -> DB : SELECT cart JOIN product WHERE email=? AND status='bill'
DB --> BillPage : Bill details
BillPage --> Guest : Display full bill (name, address,\npayment info, room details, total)
Guest -> BillPage : Click "Print"
BillPage --> Guest : window.print()
@enduml
```

---

### 3c. Admin – Confirm or Cancel a Booking

```plantuml
@startuml Sequence_Admin_Booking_Management
actor Admin
participant "adminHome.jsp" as AdminHome
participant "ordersReceived.jsp" as OrdersPage
participant "deliveredOrdersAction.jsp" as ConfirmAction
participant "cancelOrdersAction.jsp" as CancelAction
participant "ConnectionProvider" as CP
database "MySQL DB\n(cart table)" as DB

== Admin Login (already authenticated) ==

Admin -> AdminHome : Navigate to admin panel

== View Pending Bookings ==
Admin -> OrdersPage : Open "Bookings Received"
OrdersPage -> CP : getCon()
CP -> DB : JDBC connect
OrdersPage -> DB : SELECT cart JOIN product\nWHERE status='processing'\nAND check_in_date IS NOT NULL
DB --> OrdersPage : Pending booking rows
OrdersPage --> Admin : Display bookings table

== Confirm Booking ==
Admin -> ConfirmAction : Click "Confirmed" link\n(room_id, email)
ConfirmAction -> CP : getCon()
ConfirmAction -> DB : UPDATE cart SET status='delivered'\nWHERE room_id=? AND email=?
DB --> ConfirmAction : success
ConfirmAction --> Admin : redirect ordersReceived.jsp?msg=confirmed

== Cancel Booking ==
Admin -> CancelAction : Click "Cancel" link\n(room_id, email)
CancelAction -> CP : getCon()
CancelAction -> DB : UPDATE cart SET status='cancel'\nWHERE room_id=? AND email=?
DB --> CancelAction : success
CancelAction --> Admin : redirect ordersReceived.jsp?msg=cancel
@enduml
```

### Design Decisions

| Decision | Rationale |
|---|---|
| Three separate sequence diagrams | Each diagram focuses on a distinct system flow (authentication, booking lifecycle, admin operations) to keep them readable and targeted. |
| `ConnectionProvider` shown as an explicit lifeline | Since every database operation goes through this utility, surfacing it in the sequence diagram makes the data-access pattern explicit. |
| `alt/else` fragments for conditional paths | Login and add-to-cart both have branching behaviour that must be captured to show the complete flow (not just the happy path). |
| `status` field progression (`NULL → bill → processing → delivered/cancel`) | Using a single field to track the cart/booking lifecycle avoids multiple tables, but the sequence diagrams make the state transitions visible. |
| Booking confirmation/cancellation done by Admin only | Business rule: guests cannot directly confirm their own bookings; the Admin acts as the hotel front-desk approving or rejecting reservations. |

---

## 4. Entity-Relationship Summary

The following table summarises the four database tables and their relationships, complementing the class diagram above.

| Table | Primary Key | Foreign Keys / Relations |
|---|---|---|
| `user` | `guest_id` (AUTO_INCREMENT) | Referenced by `cart.email` (via `email`) |
| `product` | `id` | Referenced by `cart.room_id` |
| `cart` | *(composite: email + room_id)* | `email → user.email`, `room_id → product.id` |
| `message` | `id` (AUTO_INCREMENT) | `email → user.email` (logical) |

> **⚠️ Known Design Issues in the Current Implementation**
>
> The following limitations exist in the codebase and should be addressed in a production-grade redesign:
>
> 1. **Hardcoded admin credentials** – The admin account is checked with a hardcoded email (`admin@gmail.com`) and password in `loginAction.jsp`. A proper implementation should store admin accounts in the database with hashed passwords and role-based access control (RBAC).
>
> 2. **Auto-assigned check-in/check-out dates** – The checkout action (`addressPaymentForOrderAction.jsp`) sets `check_in_date = NOW()` and `check_out_date = DATE_ADD(NOW(), INTERVAL 3 DAY)`, overriding any dates the guest provided during registration. A production system should respect the guest's chosen dates end-to-end.
>
> 3. **Mutable `status` in composite key** – The `cart` table lacks a dedicated surrogate primary key (`AUTO_INCREMENT id`). Using `(email, room_id)` as a logical key alongside a mutable `status` column makes updates fragile and complicates foreign-key constraints. Adding an `id INT AUTO_INCREMENT PRIMARY KEY` column is recommended.
>
> 4. **Inconsistent column naming** – The `cart` schema mixes naming conventions (`Number_of_Nights` in PascalCase vs. `room_id`, `check_in_date` in snake_case). All column names should follow snake_case for consistency.

---

## How to Render the Diagrams

The PlantUML code blocks above can be rendered using any of the following tools:

1. **[PlantUML Online Server](https://www.plantuml.com/plantuml/uml/)** – paste the code between `@startuml` and `@enduml`.
2. **VS Code** – install the *PlantUML* extension and press `Alt+D` to preview.
3. **IntelliJ IDEA** – install the *PlantUML Integration* plugin.
4. **GitHub** – use a Markdown renderer that supports PlantUML (e.g., `mermaid` fenced code blocks, or a GitHub Action that generates SVG/PNG images from `.puml` files).
