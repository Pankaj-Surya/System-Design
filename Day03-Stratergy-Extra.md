# Summary Notes: Strategy Pattern and Code Reusability

These notes continue from the previous transcript, preserving the terminology and examples used by the instructor. The main topic is the Strategy Design Pattern, which helps solve code duplication and maintainability problems.

## 1. The Problem: Code Duplication

In the credit card example, different cards can use different refund algorithms.

For example:

- Visa and MasterCard may use the same refund logic.
- RuPay may use a different refund logic.
- Diners may use another refund strategy.

If the same algorithm is implemented separately in multiple classes, the code becomes duplicated.

### Why is code duplication a problem?

- If the algorithm changes, every duplicated implementation may need to be updated.
- Different implementations can accidentally become inconsistent.
- Adding new payment methods or changing existing behaviours becomes harder.
- Copying common logic into a parent class does not solve every case, because some child classes may share one algorithm while others require a different algorithm.

The instructor explains that design patterns are coding solutions to common code maintainability problems.

## 2. What Is the Strategy Pattern?

The Strategy Pattern is useful when there are multiple ways to perform the same operation.

Instead of implementing the same algorithm repeatedly inside different classes, we create separate strategy classes and reuse them wherever required.

Examples mentioned in the transcript:

- Different refund algorithms in a payment system.
- Different sorting algorithms.
- Different searching algorithms.
- Different movement strategies in a chess game.
- Different ways of listing doctors in a hospital management system.

### The basic approach

1. Create an interface representing the operation or strategy.
2. Create separate implementations for the different algorithms.
3. Let the classes that need the behaviour use a strategy object.
4. Select the appropriate strategy for each class.

This separates the algorithm from the class that uses it.

## 3. Implementing the Strategy Pattern: Refund Example

Suppose we have two ways to process a refund:

- Same-instrument refund strategy
- Bank-account refund strategy

Instead of writing the refund algorithm separately in each credit card class, we can create a common interface.

### Step 1: Create the strategy interface

```
interface RefundStrategy {
    void refund();
}
```

This interface defines the common operation that every refund strategy must implement.

### Step 2: Create different strategy implementations

```
class SameInstrumentRefundStrategy
        implements RefundStrategy {

    @Override
    public void refund() {
        // Refund to the original payment instrument
    }
}
```

```
class BankAccountRefundStrategy
        implements RefundStrategy {

    @Override
    public void refund() {
        // Refund to the bank account
    }
}
```

Now each refund algorithm has its own dedicated class.

### Step 3: Use the strategy inside credit card classes

Instead of implementing the refund algorithm directly, a credit card class holds a reference to `RefundStrategy`.

```
class VisaCreditCard {

    private RefundStrategy refundStrategy;

    VisaCreditCard() {
        this.refundStrategy =
            new BankAccountRefundStrategy();
    }

    public void refund() {
        refundStrategy.refund();
    }
}
```

Another card can use a different strategy:

```
class RupayCreditCard {

    private RefundStrategy refundStrategy;

    RupayCreditCard() {
        this.refundStrategy =
            new SameInstrumentRefundStrategy();
    }

    public void refund() {
        refundStrategy.refund();
    }
}
```

The examples are simplified to demonstrate the design pattern; actual refund behaviour depends on the system's requirements.

### What is happening here?

- Visa uses `BankAccountRefundStrategy`.
- RuPay uses `SameInstrumentRefundStrategy`.
- The refund algorithms are implemented only once.
- Each credit card delegates the refund operation to its strategy object.

If Visa and MasterCard use the same algorithm, both can reuse the same strategy implementation.

Key takeaway: The class that uses an algorithm does not need to implement the algorithm itself.

## 4. Strategy Pattern Uses Composition

The instructor explains that the Strategy Pattern uses composition.

Composition means that a class contains or holds a reference to another object that provides some of its functionality.

In this example:

- `VisaCreditCard` has a `RefundStrategy`.
- `RupayCreditCard` has a `RefundStrategy`.
- The strategy object provides the refund behaviour.

Conceptually:

Credit Card Class

Uses a RefundStrategy

RefundStrategy Interface

Bank Account Refund

Same-Instrument Refund

The credit card class delegates the operation, while the strategy implementation determines how the refund is performed.

### Why is composition useful?

- Algorithms can be reused across multiple classes.
- Changing an algorithm does not require rewriting every class that uses it.
- Different classes can use different implementations of the same interface.
- The design is easier to extend when new algorithms are introduced.

## 5. Strategy Pattern and Sorting Algorithms

The instructor also gives sorting as an example.

Suppose multiple classes need sorting, and the system supports:

- Bubble sort
- Insertion sort
- Merge sort

If two classes need bubble sort, implementing bubble sort independently in both classes creates duplication. The same issue applies to merge sort.

Instead, each sorting algorithm can be implemented as a separate strategy.

```
interface SortStrategy {
    void sort();
}
```

```
class BubbleSortStrategy implements SortStrategy {
    @Override
    public void sort() {
        // Bubble sort algorithm
    }
}
```

```
class MergeSortStrategy implements SortStrategy {
    @Override
    public void sort() {
        // Merge sort algorithm
    }
}
```

Classes that need sorting can use the appropriate `SortStrategy`.

This allows the same algorithm to be reused by different classes without duplicating its implementation.

## 6. Strategy Pattern and Searching Algorithms

Another example is a searching system.

Suppose a search operation can be performed through:

- An indexed database search
- Elasticsearch

Both approaches solve the same general problem, but their algorithms and implementations differ.

A strategy interface can represent the search operation, while separate implementations handle each search method.

The class that performs the search can use the appropriate strategy without being tightly coupled to a particular search implementation.

The same idea can be applied to a hospital management system, where doctors can be listed according to:

- Rating
- Availability
- Experience

Each option represents a different strategy for determining the order of the results.

## 7. Strategy Pattern With the Chess Example

The instructor connects the Strategy Pattern back to chess.

Suppose a modified chess game allows a Rook to move like both a Rook and a Knight.

A design based only on subclasses can become complicated when pieces support different combinations of movements.

One possible approach is to separate movement capabilities into interfaces, such as:

- Rook movement interface
- Knight movement interface

A piece can implement the movement capabilities it needs.

If the same movement algorithm is shared by several pieces, that algorithm can be implemented as a reusable strategy.

This combines two ideas:

- Interface Segregation Principle (ISP): A class should not be forced to implement behaviours it does not need.
- Strategy Pattern: A reusable algorithm should be separated from the classes that use it.

The instructor notes that this is also related to the Dependency Inversion Principle, which will be discussed later in the course.

## 8. Strategy Pattern and SOLID Principles

| Principle                             | Connection to the Strategy Pattern                                                                                      |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| SRP — Single Responsibility Principle | Separates the algorithm's responsibility from the class that uses it.                                                   |
| OCP — Open-Closed Principle           | New strategies can be introduced without repeatedly modifying the classes that use them.                                |
| LSP — Liskov Substitution Principle   | Different strategy implementations can be used through a common interface, provided they honour its contract.           |
| ISP — Interface Segregation Principle | Interfaces can represent specific capabilities instead of forcing every class to implement every operation.             |
| DIP — Dependency Inversion Principle  | Classes can depend on an abstraction such as `RefundStrategy` instead of being tightly coupled to a concrete algorithm. |

The Strategy Pattern does not automatically guarantee that every SOLID principle is satisfied. The design still needs to ensure that the abstractions and implementations are appropriate.

## 9. Constructor Injection and Frameworks

During the discussion, a question is raised about configuring strategy objects.

The instructor explains that strategies can be supplied through constructor injection or configured using a framework such as Spring Boot.

For example, the strategy can be passed into a class rather than instantiated inside it.

```
class VisaCreditCard {

    private final RefundStrategy refundStrategy;

    VisaCreditCard(RefundStrategy refundStrategy) {
        this.refundStrategy = refundStrategy;
    }

    public void refund() {
        refundStrategy.refund();
    }
}
```

Now the calling code determines which strategy to supply.

This makes the class more flexible and easier to test. It can use a different strategy without requiring changes to its internal implementation.

The instructor deliberately avoids going deeper into Spring Boot configuration because the focus is on understanding the design pattern rather than a specific framework.

## 10. Interview Preparation: Why Strategy Pattern Matters

The instructor shares an experience-based recommendation that Strategy Pattern is worth prioritising for machine-coding interviews.

The transcript mentions that, in the instructor's experience, roughly 30% of machine-coding rounds may involve some level of Strategy Pattern usage. This is an anecdotal estimate, not a general industry-wide statistic.

### When should you consider the Strategy Pattern?

Look for these signs in a problem statement:

- Multiple ways to perform the same operation.
- Several classes implementing the same algorithm.
- Repeated code across subclasses.
- A need to switch between algorithms.
- A requirement to add new algorithms without modifying existing classes extensively.

Examples include payment processing, sorting, searching, refunds, and chess movement.

## 11. Quick Revision

- Problem: Different classes contain duplicated implementations of the same algorithm.
- Solution: Extract each algorithm into its own strategy class.
- Abstraction: Define a common interface for the algorithm.
- Reuse: Multiple classes can use the same strategy implementation.
- Composition: A class holds a strategy object and delegates the operation to it.
- Flexibility: Different classes can use different strategies, and strategies can be supplied through constructor injection.
- SOLID connection: Strategy Pattern is especially relevant to OCP, SRP, ISP, and DIP.

One-line definition: The Strategy Pattern defines a family of algorithms, encapsulates each one separately, and makes them interchangeable through a common interface.
