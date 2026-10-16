# LLD & SOLID Principles — Summary Notes

## 1. Interface Segregation Principle (ISP): Credit Card Example

The discussion continues from the previous lesson on the SOLID principles and Strategy Pattern.

Suppose we have a base `CreditCard` class with common methods:

- `swipeAndPay()`
- `tapAndPay()`
- `onlinePayment()`

Different credit cards support different additional behaviours.

| Credit card | UPI payment                  | Refund        |
| ----------- | ---------------------------- | ------------- |
| Visa        | Not supported in the example | Supported     |
| MasterCard  | Supported                    | Supported     |
| RuPay       | Supported                    | Supported     |
| Diners      | Not supported                | Not supported |
| Amex        | Not supported                | Supported     |

These are hypothetical examples used in the lecture.

### Why not add every method to the base class?

If `upiPayment()` and `doRefund()` are added to the base class, cards that do not support these behaviours may be forced to implement methods that throw exceptions.

This creates unnecessary dependencies and violates the Interface Segregation Principle (ISP).

### Solution: Separate interfaces

Create focused interfaces for optional behaviours:

```
interface UPICompatibleCreditCard {
    void upiPayment();
}

interface RefundCompatibleCreditCard {
    void doRefund();
}
```

Only the cards that support UPI implement `UPICompatibleCreditCard`. Similarly, only cards that support refunds implement `RefundCompatibleCreditCard`.

Key takeaway: A class should not be forced to implement behaviours it does not need.

## 2. The Next Problem: Duplicate Algorithms

Interface segregation solves the problem of unsupported methods, but another problem appears when multiple credit cards support the same behaviour.

Consider refund implementations:

| Credit cards     | Refund algorithm       |
| ---------------- | ---------------------- |
| Visa, MasterCard | Same-instrument refund |
| RuPay, Amex      | Wallet refund          |

The lecture assumes that Visa and MasterCard use the same algorithm, while RuPay and Amex use another.

If each card implements its refund algorithm independently, the same logic gets duplicated across classes.

### Why is code duplication a problem?

- A bug must be fixed in multiple places.
- Changes to an algorithm require multiple edits.
- Maintenance becomes harder.
- Different copies can accidentally behave differently.

For example, if Visa and MasterCard use identical refund logic, maintaining two copies is unnecessary.

### DRY principle

DRY = Don't Repeat Yourself.

The instructor connects this idea to a fundamental programming technique: functions.

If a piece of logic is used repeatedly, extract it into a reusable function rather than copying it everywhere.

However, in object-oriented languages such as Java, methods belong to classes. This leads to the next design question:

Where should the shared algorithms live?

## 3. Extracting Behaviour into Separate Classes

One possible solution is to create separate classes containing the refund algorithms.

For example:

- `SameInstrumentRefund`
- `WalletRefund`

Each class contains its own refund behaviour.

This is an important object-oriented design idea: classes can represent behaviours, not just real-world entities.

A class does not always have to represent a noun such as `Student`, `Animal`, or `Employee`. It can represent a behaviour or responsibility as well.

### Why not put both algorithms in one utility class?

Consider a single class:

```
class RefundAlgorithm {
    void algorithmOne() {
        // Same-instrument refund logic
    }

    void algorithmTwo() {
        // Wallet refund logic
    }
}
```

This looks simple, but as the number of algorithms grows, the class may become cluttered.

The lecture identifies two potential SOLID violations:

- SRP — Single Responsibility Principle: Multiple independently changing algorithms are grouped into one class.
- OCP — Open/Closed Principle: Adding or changing algorithms may repeatedly require modifying the same class.

The instructor uses sorting algorithms as an analogy. A class containing bubble sort, insertion sort, merge sort, and heap sort could become difficult to maintain as each algorithm evolves.

The proposed improvement is to separate each distinct behaviour into its own class.

## 4. Composition Instead of Inheritance

The credit card classes need to use the refund behaviour, but there is no natural parent-child relationship between a credit card and a refund algorithm.

For example, a Visa card is not a refund algorithm.

Instead, a Visa card has a refund algorithm.

This is a composition relationship.

Credit Card

Visa / MasterCard / RuPay / Amex

Has a refund behaviour

Refund Algorithm

Initially represented by a concrete class

Each credit card can hold a reference to a refund algorithm object and delegate refund processing to it.

This eliminates duplicated algorithm code, but the design still has an important weakness: the credit card classes depend directly on concrete refund classes.

That leads to the Dependency Inversion Principle.

## 5. Dependency Inversion Principle (DIP)

The lecture explicitly distinguishes Dependency Inversion Principle from Dependency Injection.

### What does DIP mean?

The instructor's simplified explanation is:

> Classes should avoid depending directly on other concrete classes. Instead, they should depend on abstractions.

A higher-level class should not be tightly coupled to the concrete implementation of a lower-level dependency.

### Example: Order service and database repository

Suppose an application has an `OrderService` responsible for creating orders.

Creating an order requires database interaction.

A poor design would put the business logic and database queries into `OrderService`. This mixes responsibilities and violates SRP.

Instead, separate the responsibilities:

- `OrderService` contains order-related business logic.
- A repository class handles database interactions.

A repository may contain methods such as `insertOrder()` and other database operations.

### Problem with depending directly on a concrete repository

Imagine `OrderService` creates its own MySQL repository:

```
class OrderService {
    private MySQLRepository repository =
        new MySQLRepository();
}
```

This creates tight coupling between the service and MySQL.

If the application switches from MySQL to MongoDB, `OrderService` may need changes. Related tests may also need modification.

This makes the design less flexible and violates DIP.

### Solution: Introduce an abstraction

Define an interface:

```
interface OrderDBRepository {
    void insertOrder();
}
```

Then implement it using different repositories:

```
class MySQLRepository implements OrderDBRepository {
    public void insertOrder() {
        // MySQL database operation
    }
}

class MongoRepository implements OrderDBRepository {
    public void insertOrder() {
        // MongoDB database operation
    }
}
```

Now the service depends on the interface instead of a specific database implementation.

```
class OrderService {
    private final OrderDBRepository repository;

    OrderService(OrderDBRepository repository) {
        this.repository = repository;
    }

    void createOrder() {
        repository.insertOrder();
    }
}
```

The concrete repository can be supplied when the service is created.

Benefits:

- Database implementations can be switched without changing the business logic.
- Unit tests can provide alternative implementations or test doubles.
- The service is less tightly coupled to infrastructure.
- The design is easier to extend.

The interface guarantees that implementations provide `insertOrder()`, allowing the service to work with any compatible repository.

## 6. Dependency Inversion vs. Dependency Injection

These concepts are related, but they are not the same.

| Concept                              | Meaning                                                                                            |
| ------------------------------------ | -------------------------------------------------------------------------------------------------- |
| Dependency Inversion Principle (DIP) | A design principle that encourages depending on abstractions rather than concrete implementations. |
| Dependency Injection (DI)            | A technique for supplying a class's dependencies from outside the class.                           |

### Constructor-based dependency injection

In the preceding example:

```
OrderService(OrderDBRepository repository) {
    this.repository = repository;
}
```

The constructor receives the dependency instead of creating a concrete repository internally.

This is constructor-based dependency injection.

The lecture also mentions property-based injection and notes that Spring Boot supports different dependency-injection approaches.

Remember: DIP describes the design goal; DI is a technique that can help achieve it.

## 7. Applying DIP to the Refund Example

Return to the credit card example.

The design currently has concrete refund classes such as:

- `SameInstrumentRefund`
- `WalletRefund`

If `Visa` directly depends on `SameInstrumentRefund`, it is tightly coupled to that specific implementation.

### Step 1: Create a refund abstraction

```
interface RefundLogic {
    void doRefund();
}
```

### Step 2: Implement the abstraction

```
class SameInstrumentRefund implements RefundLogic {
    public void doRefund() {
        // Refund to the original payment instrument
    }
}

class WalletRefund implements RefundLogic {
    public void doRefund() {
        // Refund to a wallet
    }
}
```

### Step 3: Make credit cards depend on the interface

```
class Visa extends CreditCard {
    private final RefundLogic refundLogic;

    Visa(RefundLogic refundLogic) {
        this.refundLogic = refundLogic;
    }

    public void refundAmount() {
        refundLogic.doRefund();
    }
}
```

A compatible refund implementation can now be supplied when creating a card.

```
Visa visa = new Visa(new SameInstrumentRefund());
```

The same overall design can be used for MasterCard, RuPay, and Amex.

The key change is that the card classes depend on `RefundLogic`, not on a specific concrete refund class.

### Result

- Duplicate refund algorithms are extracted into reusable classes.
- Each class has a focused responsibility.
- Credit cards can use different refund implementations.
- Refund behaviour can be selected when the card object is constructed.
- Concrete implementations can be changed without rewriting the credit card classes.

This is a practical application of DIP and composition.

## 8. How This Leads to the Strategy Pattern

The instructor highlights that the solution arrived at through SOLID principles is essentially the Strategy Pattern.

The pattern has three main parts:

1\. Strategy interface

`RefundLogic` defines the common operation, `doRefund()`.

2\. Concrete strategies

`SameInstrumentRefund` and `WalletRefund` implement different refund algorithms.

3\. Context

The credit card holds a strategy reference and delegates refund processing to it.

The important idea is to encapsulate a family of algorithms behind a common interface and make them interchangeable.

The credit card does not need to know the internal details of the refund algorithm. It only needs to invoke the common operation.

The instructor also discusses how this design supports the Liskov Substitution Principle (LSP): implementations should honour the contract expected by their abstraction. Simply implementing an interface is not enough if an implementation breaks that contract.

### SOLID principles covered so far

| Principle | Role in the example                                                    |
| --------- | ---------------------------------------------------------------------- |
| SRP       | Keep refund algorithms separate from credit card responsibilities.     |
| OCP       | Add new refund strategies without modifying the existing card classes. |
| LSP       | Implementations must honour their declared contracts.                  |
| ISP       | Keep optional behaviours in focused interfaces.                        |
| DIP       | Depend on the `RefundLogic` abstraction.                               |

The instructor recommends revising the five SOLID principles daily for five days to strengthen understanding for both interviews and real software engineering.

# 9. New Problem: Expensive Object Creation

The lecture then moves from the Strategy Pattern to another design problem, using chess pieces and email objects.

The key requirement is different: creating many similar objects may be expensive or unnecessarily repetitive.

## Example A: Chess pawns

In a chess game, there are 16 pawns.

A straightforward approach creates each pawn separately:

```
Pawn p1 = new Pawn();
Pawn p2 = new Pawn();
Pawn p3 = new Pawn();
// ...continue for all 16 pawns
```

The constructor may require different configuration for each pawn, such as:

- Coordinates: A2, B2, C2, and so on.
- Colour.
- Other piece-specific properties.

A loop may not be suitable for every setup if each object needs distinct configuration.

The interviewer introduces an additional constraint: creating each pawn object is potentially expensive.

This motivates finding a more efficient way to create similar objects.

## Example B: Email objects

Consider an email class with properties such as:

```
class Email {
    String recipient;
    String subject;
    String body;
}
```

An e-commerce application may send emails for:

- Successful orders.
- Account creation or login.
- Refunds.
- Support tickets.

Each email type can have a different HTML template. The template may contain complex markup, images, styling, and placeholders for dynamic information.

For example, two order-confirmation emails might share the same layout but have different recipients, names, order IDs, and delivery details.

### Why can creating each email be expensive?

The lecture explains that email creation may involve:

1. Reading an HTML template file.
2. Loading the template into memory.
3. Replacing placeholders with dynamic values.
4. Configuring or reusing connections and other required data.

Repeating all these operations for every email may be redundant, particularly when most of the configuration remains unchanged.

## 10. Reusing an Existing Object as a Template

Instead of rebuilding an object from scratch every time, the lecture proposes creating an initial, configured object and then making copies of it.

For example, an email template object could already contain:

- Loaded HTML content.
- Existing configuration.
- An established connection object, where appropriate.

A new email can then be created by copying that object and changing only the necessary properties, such as the recipient or subject.

Original configured object

Template, configuration, reusable data

Clone or copy

Copy A

Recipient A

Shared template

Copy B

Recipient B

Shared template

The intended benefit is to avoid repeating expensive setup work when an object already has reusable data or configuration.

The instructor compares this with graph cloning: constructing an original graph may involve reading a test case, parsing vertices and edges, and building the graph. Creating a copy of an already-loaded graph may avoid repeating the input-reading and parsing process.

Another example mentioned is an AWS S3 client object, where the region and access configuration may remain the same across repeated uses.

### Important distinction

The lecture's point is that cloning or copying can be more efficient than repeating expensive initialization. The actual benefit depends on how the object is copied and which resources or properties need to be recreated.

The available transcript ends while this object-copying problem is being introduced. It does not include the complete implementation or final discussion of the pattern.

# Quick Revision

| Topic                     | Remember this                                                         |
| ------------------------- | --------------------------------------------------------------------- |
| ISP                       | Split optional behaviours into focused interfaces.                    |
| DRY                       | Avoid repeating the same logic across multiple classes.               |
| SRP                       | Give each class a focused responsibility.                             |
| Composition               | Model a `has-a` relationship instead of forcing inheritance.          |
| DIP                       | Depend on abstractions, not concrete implementations.                 |
| Dependency Injection      | Supply dependencies from outside the class.                           |
| Strategy Pattern          | Encapsulate interchangeable algorithms behind a common interface.     |
| Expensive object creation | Reusing or copying a configured object may avoid repeated setup work. |

## Interview-ready definitions

- Dependency Inversion Principle: High-level modules should not depend directly on low-level modules; both should depend on abstractions.
- Dependency Injection: A technique for supplying an object's dependencies externally rather than having the object construct them itself.
- Strategy Pattern: A behavioural design pattern that encapsulates interchangeable algorithms behind a common interface.
- Composition: Building a class using references to other objects that provide parts of its behaviour.
- DRY principle: Don't Repeat Yourself; avoid maintaining multiple copies of the same logic.

Main lesson: Good object-oriented design is not just about creating classes and inheritance hierarchies. It is about separating responsibilities, extracting reusable behaviours, depending on abstractions, and choosing object relationships that keep the code flexible and maintainable.
