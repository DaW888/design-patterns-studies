# Creational Patterns

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

Na tym diagramie sekwencji przedstawiono proces zakupu w systemie sklepu internetowego. Diagram ilustruje następujące etapy:
1. Dodawanie przedmiotów do koszyka
    - Klient (`Customer`) dodaje przedmioty (`Item`) do koszyka (`ShoppingCart`).
    - Koszyk tworzy obiekt `ShoppingCartItem` z informacjami o dodanym przedmiocie oraz jego ilości.
    - Koszyk aktualizuje łączną cenę i przekazuje ją do klienta.
2. Składanie zamówienia:
    - Klient korzysta z `OrderBuilder` (`OBuilder`) do ustawienia informacji o sobie, adresie wysyłki oraz metodzie płatności.
    - W pętli, klient dodaje wszystkie przedmioty z koszyka do OrderBuilder.
    - Następnie klient wywołuje metodę `build()` na `OBuilder`, który tworzy nowe zamówienie (`Order`) i zwraca je do klienta.
    - Klient wywołuje metodę create() na obiekcie `Order`, który następnie korzysta z metody `pay()` na wybranej metodzie płatności (`PaymentMethod`).
    - Diagram pokazuje dwie możliwe metody płatności: kartą kredytową (`CreditCardPayment`) lub PayPal (`PayPalPayment`).
    - Po potwierdzeniu płatności, przedmioty są dodawane do zamówienia, a także przypisywane do odpowiednich kategorii (`Category`).
3. Dodawanie recenzji:
    - W pętli, klient dodaje recenzje (`Review`) do przedmiotów.
    - Recenzja zawiera informacje o kliencie, treść oraz ocenę
    - Recenzja jest dodawana do odpowiedniego przedmiotu (`Item`)


Diagram sekwencji ilustruje interakcje pomiędzy klientem, koszykiem, zamówieniem, przedmiotami, metodami płatności, recenzjami oraz kategoriami w trakcie całego procesu zakupu w sklepie internetowym.


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

## Abstract Factory Pattern
Wzorzec **Fabryki Abstrakcyjnej** warto zastosować w tym przypadku, ze względu na obecność różnych typów płatności: `CreditCardPayment` oraz `PayPalPayment`.
Wzorzec ten pozwala na odseparowanie tworzenia obiektów płatności od głównych klas, takich jak `Customer` czy `Order`.

Główne zalety użycia wzorca **Fabryki Abstrakcyjnej** w tym kontekście to:

1. **Łatwe rozszerzanie**:
   Jeśli pojawią się nowe metody płatności, wystarczy dodać nową klasę fabryki, która będzie implementować interfejs `PaymentMethodFactory`. Nie będzie konieczne modyfikowanie istniejącego kodu, co zwiększa elastyczność systemu i ułatwia jego rozwijanie.
2. **Oddzielenie logiki tworzenia obiektów**
   Wzorzec **Fabryki Abstrakcyjnej** pozwala na przeniesienie logiki tworzenia obiektów płatności do dedykowanych klas fabryk. Dzięki temu klasa `Order` może skupić się na swojej głównej funkcji, czyli zarządzaniu zamówieniem, a nie na tworzeniu obiektów płatności.
3. **Ułatwione testowanie**
   Dzięki temu, że tworzenie obiektów płatności jest odseparowane od pozostałych klas, łatwiej będzie napisać testy jednostkowe sprawdzające różne metody płatności. Można wtedy łatwo podmienić konkretne implementacje fabryk na ich testowe wersje (np. z użyciem mocków), co pozwala na kontrolowanie zachowań w trakcie testowania.
4. **Łatwość konfiguracji**
   Wykorzystując wzorzec Fabryki Abstrakcyjnej, łatwo można zmienić metodę płatności używaną przez system, podmieniając tylko odpowiednią klasę fabryki. Dzięki temu system staje się bardziej elastyczny i łatwy w konfiguracji.



### Class diagram

```mermaid
classDiagram
    class Order {
        +Int orderID
        +Customer customer
        +Address shippingAddress
        +PaymentMethod paymentMethod
        +List~Item~ items
        +create() void
        +cancel() Void
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

    class PaymentMethodFactory {
        <<abstract>>
        +createPaymentMethod() PaymentMethod
    }
    class CreditCardPaymentFactory {
        +createPaymentMethod() CreditCardPayment
    }
    class PayPalPaymentFactory {
        +createPaymentMethod() PayPalPayment
    }

    CreditCardPayment --|> PaymentMethod
    PayPalPayment --|> PaymentMethod
    CreditCardPaymentFactory --|> PaymentMethodFactory : inherits
    PayPalPaymentFactory --|> PaymentMethodFactory : inherits
    Order "1" --* "1" PaymentMethod
    Order "1" --* "1" PaymentMethodFactory
    CreditCardPaymentFactory ..> CreditCardPayment : creates
    PayPalPaymentFactory ..> PayPalPayment : creates
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
   participant PaymentMethodFactory
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
   Order->>PaymentMethodFactory: createPaymentMethod()
   activate PaymentMethodFactory
   alt Credit Card
      PaymentMethodFactory->>CreditCardPayment: create()
      PaymentMethodFactory->>Order: return CreditCardPayment
   else PayPal
      PaymentMethodFactory->>PayPalPayment: create()
      PaymentMethodFactory->>Order: return PayPalPayment
   end
   deactivate PaymentMethodFactory
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

