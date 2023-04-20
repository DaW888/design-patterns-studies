# Behavioral Patterns

## Observer

Wzorzec observer może być użyteczny w naszym diagramie klas w kontekście relacji między klasami `Customer` i `Order`. 
Gdy klient składa zamówienie, może chcieć być powiadamiany o zmianach w statusie tego zamówienia (np. gdy zamówienie jest wysyłane, opóźnione lub anulowane). 
Implementacja wzorca obserwatora pozwala na utrzymanie spójności informacji między klientem a jego zamówieniami, jednocześnie unikając sztywnego powiązania między tymi klasami.

W tym przypadku klasa `Order` (**subject**) reprezentuje obiekt powiadamiający, który przechowuje informacje o stanie zamówienia, a klasa `Customer` (**observer**) reprezentuje obiekt obserwujący, który jest zainteresowany otrzymywaniem informacji o zmianach stanu zamówienia.

Zastosowanie wzorca observer w tym diagramie ma następujące zalety:
1. **Oddzielenie klientów od zamówień**: 
Implementacja wzorca observer pozwala na oddzielenie logiki klientów od logiki zamówień. 
Dzięki temu można modyfikować jedną stronę zależności (np. wprowadzać zmiany w klasie `Order`), nie wpływając na drugą stronę (klasę `Customer`).
2. **Dynamiczne powiadamianie o zmianach**: 
Wzorzec **observer** umożliwia automatyczne powiadamianie klientów o zmianach w ich zamówieniach, bez konieczności sprawdzania tych zmian w sposób aktywny. 
Klient zostanie powiadomiony natychmiast po wystąpieniu zmiany stanu zamówienia.
3. **Skalowalność**: 
Wzorzec **observer** pozwala na łatwe dodawanie nowych klientów i zamówień, bez konieczności modyfikacji istniejącego kodu. 
Nowe obiekty klientów mogą być łatwo zarejestrowane jako obserwatorzy dla istniejących lub nowych zamówień.
4. **Elastyczność**: 
Dzięki wzorcowi **observer** istnieje możliwość dodawania nowych funkcjonalności, które również będą wykorzystywać mechanizm powiadamiania o zmianach w zamówieniach. 
Na przykład, można dodać funkcję przesyłania e-maili z powiadomieniami lub inne kanały komunikacji bez konieczności modyfikacji istniejących klas `Customer` i `Order`.

Podsumowując, zastosowanie wzorca observer w diagramie klas związanych z klientami i zamówieniami pozwala na utrzymanie spójności informacji, jednocześnie unikając sztywnych powiązań między klasami. Daje to większą elastyczność i łatwość w rozbudowie i modyfikacji systemu.


### Class diagram

W powyższym diagramie sekwencji, dodajemy powiadomienie obserwatorów (klientów) poprzez wywołanie metody "notify()" w klasie `Order`. 
Następnie, dla każdego klienta, wywołuję metodę "update()" w celu poinformowania ich o zmianach w zamówieniu.

```mermaid
classDiagram
    class Subject {
        <<interface>>
        +attach(observer: Observer) void
        +detach(observer: Observer) void
        +notify() void
    }
    class Observer {
        <<interface>>
        +update() void
    }
    class Customer~Observer~ {
        +Int customerID
        +String email
        +String password
        +register() Void
        +login() Void
        +placeOrder() Order
        +update() void
    }

    class Order~Subject~ {
        +Int orderID
        +Customer customer
        +Address shippingAddress
        +PaymentMethod paymentMethod
        +List~Item~ items
        +create() void
        +cancel() Void
    }

    Customer "1" --o "*" Order
    Order "1" --> "1" Customer

    Customer --|> Observer
    Order --|> Subject
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
    Order->>Order: notify()
    activate Order
    loop For each Customer
        Order->>Customer: update()
    end
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


## Chain of Responsibility

Wzorzec Chain of Responsibility (łańcuch odpowiedzialności) może być użyty w naszym diagramie klas, aby zdecentralizować logikę obsługi uprawnień związanych z anulowaniem zamówienia przez klienta. 
Zamiast umieszczać całą logikę sprawdzania uprawnień w jednej klasie, ten wzorzec pozwala na utworzenie łańcucha obiektów obsługujących uprawnienia. 
Każdy obiekt w łańcuchu odpowiada za sprawdzenie uprawnień na swoim poziomie i przekazanie sprawdzenia dalej, jeśli nie jest w stanie udzielić odpowiedniej decyzji.

W naszym diagramie klas, klient może mieć różne poziomy uprawnień, 
np. podstawowe, premium i admin. W związku z tym, możemy utworzyć trzy różne klasy obsługujące te poziomy uprawnień: `BasicPermissionHandler`, `PremiumPermissionHandler` i `AdminPermissionHandler`. 
Każda z tych klas implementuje interfejs `PermissionHandler`, który definiuje wspólne metody do ustawienia następnego obiektu w łańcuchu (`setNextHandler`) i obsługi uprawnień (`handlePermission`).

W przypadku anulowania zamówienia przez klienta, łańcuch odpowiedzialności rozpoczyna się od `BasicPermissionHandler`, 
który sprawdza, czy klient ma podstawowe uprawnienia do anulowania zamówienia. Jeśli nie jest w stanie udzielić odpowiedniej decyzji, 
przekazuje sprawdzenie dalej do `PremiumPermissionHandler`, który z kolei sprawdza uprawnienia na poziomie premium. Jeśli również nie jest w stanie udzielić odpowiedniej decyzji, przekazuje sprawdzenie do `AdminPermissionHandler`, 
który sprawdza uprawnienia na poziomie administratora. Proces ten kontynuowany jest, aż jeden z obiektów obsługujących uprawnienia podejmie decyzję lub do momentu, gdy cały łańcuch zostanie sprawdzony.


Korzystając z tego wzorca w naszym diagramie klas, możemy łatwo rozszerzać i modyfikować logikę sprawdzania uprawnień bez ingerowania w inne klasy, takie jak `Customer` czy `Order`. 
Wzorzec ten wprowadza elastyczność i łatwość zarządzania, gdy potrzebujemy wprowadzić zmiany w systemie uprawnień.


### Class diagram

```mermaid
classDiagram
    class Customer {
        +Int customerID
        +String email
        +String password
        +register() Void
        +login() Void
        +placeOrder() Order
        +canCancelOrder() Boolean
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

    class PermissionHandler {
        <<interface>>
        +setNextHandler(PermissionHandler nextHandler) void
        +handlePermission(Customer customer) Boolean
    }
    class BasicPermissionHandler~PermissionHandler~ 
    class PremiumPermissionHandler~PermissionHandler~ 
    class AdminPermissionHandler~PermissionHandler~

    BasicPermissionHandler --|> PermissionHandler
    PremiumPermissionHandler --|> PermissionHandler
    AdminPermissionHandler --|> PermissionHandler
    BasicPermissionHandler "1" --> "0..1" PremiumPermissionHandler : nextHandler
    PremiumPermissionHandler "1" --> "0..1" AdminPermissionHandler : nextHandler

    Customer "1" --o "*" Order
    Order "1" --> "1" Customer
    Customer "1" --> "1" BasicPermissionHandler : checkPermission
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
    participant PermissionHandler
    participant BasicPermissionHandler
    participant PremiumPermissionHandler
    participant AdminPermissionHandler

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

    Note over Customer, PermissionHandler: Check Permission to Cancel Order
    Customer->>BasicPermissionHandler: canCancelOrder()
    activate BasicPermissionHandler
    alt BasicPermissionHandler can handle
        BasicPermissionHandler->>Customer: return permission result
        deactivate BasicPermissionHandler
    else BasicPermissionHandler can't handle
        BasicPermissionHandler->>PremiumPermissionHandler: handlePermission(Customer)
        activate PremiumPermissionHandler
        alt PremiumPermissionHandler can handle
            PremiumPermissionHandler->>Customer: return permission result
            deactivate PremiumPermissionHandler
        else PremiumPermissionHandler can't handle
            PremiumPermissionHandler->>AdminPermissionHandler: handlePermission(Customer)
            activate AdminPermissionHandler
            AdminPermissionHandler->>Customer: return permission result
            deactivate AdminPermissionHandler
        end
    end
```