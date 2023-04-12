# design-patterns-studies

### TOC

 - [Creational Patterns](./CREATIONAL.md) 
 - [Structural Patterns](./STRUCTURAL.md)
 - [Behavioral Patterns](./BEHAVIORAL.md)


## Main diagram without patterns

### Class diagram

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

### Sequence diagram

```mermaid
sequenceDiagram
    actor Customer
    participant ShoppingCart
    participant ShoppingCartItem
    participant Item
    participant Order
    participant Address
    participant PaymentMethod
    participant CreditCardPayment
    participant PayPalPayment
    participant Review
    participant Category

    Note over Customer, Item: Add Items to Cart
    loop Add items to cart
        Customer->>ShoppingCart: addItem(Item)
        activate ShoppingCart
        ShoppingCart->>ShoppingCartItem: create ShoppingCartItem with Item details
        ShoppingCart->>Item: reference Item
        ShoppingCart->>ShoppingCartItem: set quantity
        ShoppingCart->>Customer: update total price
        deactivate ShoppingCart
    end

    Note over Customer, Order: Place Order
    Customer->>Order: placeOrder()
    activate Order
    Order->>ShoppingCart: get items and total price
    Order->>Customer: get customer details
    Order->>Address: create Address
    Order->>PaymentMethod: select payment method
    alt Credit Card
        PaymentMethod->>CreditCardPayment: pay(amount)
    else PayPal
        PaymentMethod->>PayPalPayment: pay(amount)
    end
    PaymentMethod->>Order: confirm payment
    Order->>Item: add items to order
    Item->>Category: assign item to category
    deactivate Order

    Note over Customer, Review: Add Review
    loop Add review
        Customer->>Review: write review
        activate Review
        Review->>Customer: get customer details
        Review->>Item: add review to item
        deactivate Review
    end
```
