# Structural Patterns

## Decorator Pattern

Wzorzec dekorator został użyty w przypadku klasy "Item" 
w celu dynamicznego dodawania nowych właściwości (w tym przypadku: kolor, rozmiar, materiał i marka) 
do obiektów tej klasy bez konieczności modyfikowania jej kodu źródłowego. 
Wykorzystanie wzorca dekoratora przynosi wiele korzyści:
1. **Elastyczność**: Dekorator pozwala na dodawanie lub usuwanie nowych właściwości obiektów w czasie wykonywania programu, co daje dużą elastyczność w zarządzaniu obiektami. 
Zamiast tworzyć wiele różnych podklas dla różnych kombinacji właściwości, możemy korzystać z pojedynczej klasy bazowej "Item" i dekorować ją za pomocą konkretnych 
dekoratorów w miarę potrzeb.
2. Otwartość na rozszerzenia, zamknięcie na modyfikacje (zasada "**Open/Closed**"): 
Wzorzec dekorator pozwala na rozszerzanie funkcjonalności obiektów bez konieczności modyfikowania ich kodu źródłowego. 
Dzięki temu możemy dodawać nowe funkcje i właściwości bez ryzyka wprowadzenia błędów do istniejącego kodu.
3. Zachowanie **jednorodności** interfejsów: 
Dekorator umożliwia dodawanie nowych właściwości, jednocześnie utrzymując jednorodność interfejsów klas. 
Wszystkie dekorowane obiekty implementują ten sam interfejs ("ItemInterface"), co ułatwia ich użycie w różnych częściach systemu.
4. **Rozdzielenie obowiązków**: Wzorzec dekorator promuje rozdzielenie obowiązków, ponieważ każdy dekorator zajmuje się tylko jednym aspektem rozszerzenia 
funkcjonalności klasy. Ułatwia to zarządzanie kodem, ponieważ każda klasa dekoratora ma jedno konkretne zadanie.

W podanym przykładzie, zastosowanie wzorca dekoratora pozwala na łatwe dodawanie, usuwanie lub modyfikowanie właściwości obiektów klasy "Item" bez konieczności modyfikowania ich kodu źródłowego ani tworzenia licznych podklas.

### Class Diagram

```mermaid
classDiagram
    class Order {
        +Int orderID
        +Customer customer
        +Address shippingAddress
        +PaymentMethod paymentMethod
        +List~ItemInterface~ items
        +create() void
        +cancel() Void
    }
    class ItemInterface {
        <<interface>>
        +getItemID() Int
        +getName() String
        +getDescription() String
        +getPrice() Float
    }
    class Item {
        +Int itemID
        +String name
        +String description
        +Float price
    }
    class ItemDecorator {
        <<abstract>>
        -ItemInterface item
    }
    class ColorDecorator {
        -String color
    }
    class SizeDecorator {
        -String size
    }
    class MaterialDecorator {
        -String material
    }
    class BrandDecorator {
        -String brand
    }
    class ShoppingCartItem {
        +ItemInterface item
        +Int quantity
    }
    class Review {
        +Int reviewID
        +Customer customer
        +ItemInterface item
        +String title
        +String content
        +Int rating
        +Date datePosted
    }
    class Category {
        +Int categoryID
        +String name
        +String description
        +List~ItemInterface~ items
    }

    Item --|> ItemInterface
    ItemDecorator --|> ItemInterface
    ItemDecorator "1" *-- "1" ItemInterface
    ColorDecorator --|> ItemDecorator
    SizeDecorator --|> ItemDecorator
    MaterialDecorator --|> ItemDecorator
    BrandDecorator --|> ItemDecorator

    ShoppingCartItem "1" --o "1" ItemInterface
    Order "1" --o "1..*" ItemInterface
    ItemInterface "1" --o "*" Review
    ItemInterface "1" --> "1" Category
    Category "1" --o "1..*" ItemInterface
```

### Sequence Diagram

W celu zaimplementowania wzorca dekoratora w diagramie sekwencji, wprowadziliśmy następujące zmiany:
1. Zamiast korzystać bezpośrednio z obiektów Item, teraz używamy obiektów `ItemInterface`, 
które mogą być dekorowane za pomocą konkretnych klas dekoratorów, takich jak `ColorDecorator`, `SizeDecorator`, `MaterialDecorator` i `BrandDecorator`. 
Wprowadzenie ItemInterface pozwala na dodanie nowych właściwości do obiektów Item bez modyfikowania ich kodu.
2. Przed dodaniem przedmiotów do koszyka, klient tworzy dekorowane obiekty ItemInterface za pomocą klas `ItemDecorator`. 
To umożliwia dodanie pożądanych właściwości do obiektów Item w elastyczny sposób, bez konieczności modyfikowania ich oryginalnych klas.
3. W momencie dodawania przedmiotów do koszyka, klient dodaje teraz dekorowane obiekty `ItemInterface` do koszyka. 
Pozwala to na przechowywanie dodatkowych właściwości związanych z przedmiotami w koszyku i ich uwzględnienie podczas składania zamówienia.
4. Diagram sekwencji uwzględnia teraz korzystanie z dekoratorów w kontekście dodawania przedmiotów do koszyka, składania zamówień i dodawania recenzji. Wszystkie te interakcje odnoszą się teraz do dekorowanych obiektów `ItemInterface`,
co pozwala na większą elastyczność i rozszerzalność systemu.

Wprowadzenie wzorca dekoratora w diagramie sekwencji pozwala na lepsze zarządzanie dodatkowymi właściwościami przedmiotów, bez konieczności modyfikowania istniejących klas. Daje to większą elastyczność i łatwiejsze utrzymanie kodu w przyszłości.


```mermaid
sequenceDiagram
    actor Customer
    participant ShoppingCart
    participant ShoppingCartItem
    participant ItemInterface
    participant ItemDecorator
    participant Order
    participant Address
    participant PaymentMethod
    participant CreditCardPayment
    participant PayPalPayment
    participant Review
    participant Category

    Note over Customer, ItemInterface: Add Items to Cart
    loop Add items to cart
        Customer->>ItemDecorator: Create decorated ItemInterface with desired properties
        activate ItemDecorator
        ItemDecorator->>ItemInterface: Decorate ItemInterface
        deactivate ItemDecorator
        Customer->>ShoppingCart: addItem(decorated ItemInterface)
        activate ShoppingCart
        ShoppingCart->>ShoppingCartItem: create ShoppingCartItem with decorated ItemInterface details
        ShoppingCart->>ItemInterface: reference decorated ItemInterface
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
    Order->>ItemInterface: add items to order
    ItemInterface->>Category: assign item to category
    deactivate Order

    Note over Customer, Review: Add Review
    loop Add review
        Customer->>Review: write review
        activate Review
        Review->>Customer: get customer details
        Review->>ItemInterface: add review to item
        deactivate Review
    end
```