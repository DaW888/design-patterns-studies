# design-patterns-studies

## Main diagram without patterns

```mermaid
---
title: Web Store
---

classDiagram
    class Customer {
        +Int customerID
        +String email
        +String password
        +register() Void
        +login() Void
        +placeOrder() Order
    }
    class ShoppingCart {
        +List~ShoppingCartItem~ items
        +addItem(Item item) void
        +removeItem(Item item) void
        +calculateTotal() Float
    }
    class Order {
        +Int orderID
        +Customer customer
        +Address shippingAddress
        +PaymentMethod paymentMethod
        +List~Item~ items
        +create() void
        +cancel() Void
    }
    class Item {
        +Int itemID
        +String name
        +String description
        +Float price
    }
    class ShoppingCartItem {
        +Item item
        +Int quantity
    }
    class Address {
        +String street
        +String city
        +String state
        +String postalCode
        +String country
    }
    class PaymentMethod {
        <<interface>>
        +pay(Float amount) Boolean
    }
    class CreditCardPayment~PaymentMethod~ {
        +String cardNumber
        +Date expirationDate
        +Int cvv
    }
    class PayPalPayment~PaymentMethod~ {
        +String email
        +String password
    }
    class Review {
        +Int reviewID
        +Customer customer
        +Item item
        +String title
        +String content
        +Int rating
        +Date datePosted
    }
    class Category {
        +Int categoryID
        +String name
        +String description
        +List~Item~ items
    }

    CreditCardPayment --|> PaymentMethod
    PayPalPayment --|> PaymentMethod

%%    Customer "1" --> "1" ShoppingCart
%%    Customer "1" --> "*" Order
%%    Customer "1" --> "*" Review
%%    ShoppingCart "1" --> "1..*" ShoppingCartItem
%%    ShoppingCartItem "1" --> "1" Item
%%    Order "1" --> "1" Customer
%%    Order "1" --> "1" Address
%%    Order "1" --> "1" PaymentMethod
%%    Order "1" --> "1..*" Item
%%    Item "1" --> "*" Review
%%    Item "1" --> "1" Category
%%    Category "1" --> "1..*" Item

    Customer "1" --* "1" ShoppingCart
    Customer "1" --o "*" Order
    Customer "1" --o "*" Review
    ShoppingCart "1" --* "1..*" ShoppingCartItem
    ShoppingCartItem "1" --o "1" Item
    Order "1" --> "1" Customer
    Order "1" --* "1" Address
    Order "1" --* "1" PaymentMethod
    Order "1" --o "1..*" Item
    Item "1" --o "*" Review
    Item "1" --> "1" Category
    Category "1" --o "1..*" Item
```
