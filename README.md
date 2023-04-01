# design-patterns-studies

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


## Builder pattern

W tym diagramie warto użyć wzorca **Budowniczy** (_Builder_) ze względu na złożoność obiektów, takich jak `Order`, które mają wiele atrybutów i zależności. 
Wzorzec **Builder** może pomóc w uporządkowaniu procesu tworzenia obiektów `Order` poprzez zapewnienie bardziej elastycznego i czytelnego podejścia do 
konstrukcji tych obiektów.

Oto kilka powodów, dla których warto użyć wzorca Builder w tym przypadku:
1. Łatwiejsza konstrukcja obiektów: **Budowniczy** pozwala na tworzenie obiektów krok po kroku, umożliwiając konstrukcję obiektów z różnymi kombinacjami atrybutów i zależnościami. 
   Dzięki temu można uniknąć zbyt skomplikowanych konstruktorów lub wielu konstruktorów z różnymi argumentami.
2. Oddzielenie logiki konstrukcji od reprezentacji: Wzorzec **Builder** oddziela proces tworzenia obiektów od ich reprezentacji. 
   Umożliwia to utrzymanie kodu bardziej przejrzystym i modularnym, ułatwiając jednocześnie utrzymanie i rozwój.
3. Łatwiejsza modyfikacja i rozbudowa: Jeśli chcemy dodać nowy atrybut lub zależność do klasy Order, wystarczy zmodyfikować odpowiednią metodę w Builderze, zamiast modyfikować wiele konstruktorów. Dzięki temu modyfikacje są łatwiejsze do zarządzania, a kod jest bardziej elastyczny.
4. Niezmienność obiektów: Wzorzec Builder może pomóc w utrzymaniu niezmienności obiektów, co jest szczególnie ważne w przypadku klas, które mają wiele atrybutów. Niezmienne obiekty są bezpieczniejsze i łatwiejsze w utrzymaniu, ponieważ ich stan nie może być zmieniony po utworzeniu.

W kontekście tego diagramu, warto zastosować wzorzec **Builder** w klasie `Order`, która ma wiele zależności, takich jak `Customer`, `Address`, `PaymentMethod` i `Item`. 
Implementacja Builder'a w klasie Order pozwoli na elastyczne tworzenie zamówień z różnymi kombinacjami atrybutów, jednocześnie utrzymując kod czytelnym i łatwym w utrzymaniu.


### Class diagram

> Wyłącznie modyfikowana część (nie cały diagram)

```mermaid
classDiagram
    class Customer {
        +Int customerID
        +String email
        +String password
        +register() Void
        +login() Void
        +placeOrder() Order
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

    class IOrderBuilder {
        <<interface>>
        +setCustomer(Customer) IOrderBuilder
        +setShippingAddress(Address) IOrderBuilder
        +setPaymentMethod(PaymentMethod) IOrderBuilder
        +addItem(Item) IOrderBuilder
        +build() Order
    }

    class AbstractOrderBuilder {
        <<abstract>>
        -Customer customer
        -Address shippingAddress
        -PaymentMethod paymentMethod
        -List~Item~ items
    }

    class OrderBuilder {
        +setCustomer(Customer) OrderBuilder
        +setShippingAddress(Address) OrderBuilder
        +setPaymentMethod(PaymentMethod) OrderBuilder
        +addItem(Item) OrderBuilder
        +build() Order
    }

    Customer "1" --o "*" Order : customer
    Order "1" --> "1" Customer : contains
    Order "1" --* "1" Address : shippingAddress
    Order "1" --* "1" PaymentMethod : paymentMethod
    Order "1" --o "1..*" Item : items
    OrderBuilder --|> AbstractOrderBuilder : extends
    AbstractOrderBuilder ..|> IOrderBuilder : implements
    OrderBuilder o-- "1" Customer : uses
    OrderBuilder o-- "1" Address : uses
    OrderBuilder o-- "1" PaymentMethod : uses
    OrderBuilder o-- "1..*" Item : uses
```

### Sequence diagram

```mermaid
sequenceDiagram
    actor Customer
    participant ShoppingCart
    participant ShoppingCartItem
    participant Item
    participant OBuilder as OrderBuilder
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
    Customer->>OBuilder: setCustomer(Customer)
    Customer->>OBuilder: setShippingAddress(Address)
    Customer->>OBuilder: setPaymentMethod(PaymentMethod)
    loop Add items
        Customer->>OBuilder: addItem(Item)
    end
    Customer->>OBuilder: build()
    OBuilder->>Order: create Order
    OBuilder->>Customer: return Order
    Customer->>Order: create()
    activate Order
    Order->>PaymentMethod: pay(amount)
    alt Credit Card
        PaymentMethod->>CreditCardPayment: process payment
    else PayPal
        PaymentMethod->>PayPalPayment: process payment
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
