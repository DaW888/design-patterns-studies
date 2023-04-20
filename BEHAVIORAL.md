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

