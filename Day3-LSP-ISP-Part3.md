# Summary Notes: SOLID Principles Through Chess and Payment System Examples

These notes follow the transcript’s language, terminology, and examples. The main focus is understanding Liskov Substitution Principle (LSP) and Interface Segregation Principle (ISP) by identifying design problems and improving the class structure.

## 1. Why solve a real-world LLD problem?

The instructor introduces a practical Low-Level Design (LLD) interview problem to show how SOLID principles apply to actual code.

- LLD interviews often involve designing games such as Snake, Connect Four, Tic-Tac-Toe, and Chess.
- Designing the complete chess game is not the objective here.
- The focus is on designing the chess piece class.
- The objective is to identify design problems, understand which SOLID principles are violated, and restructure the classes.

The key idea is that understanding design problems through practical examples makes the SOLID principles easier to retain.

## 2. Chess Piece Class: The Initial Design

Suppose we have to create a `ChessPiece` class.

### Possible properties

- Position: X-Y coordinates or row and column.
- Colour: An enum representing the colour of the piece.
- Character representation: A character representing the piece, such as `K`, `R`, or `B`.

### Possible behaviour

- `move()`
- `canMove()`
- Other piece-specific operations.

At first glance, keeping all these properties and behaviours inside one class might seem reasonable. However, the problem appears when we consider that different chess pieces have different movement rules.

For example:

- Bishop moves diagonally.
- Rook moves horizontally and vertically.
- Pawn has specific forward-movement and capturing rules.
- Knight moves in an L-shaped pattern.
- King and Queen have their own movement rules.

If every movement rule is handled by the same `ChessPiece` class, the class becomes responsible for the behaviour of every piece.

### Problem with the initial design

The movement function might look conceptually like this:

```
void move() {
    if (name.equals("R")) {
        // Rook movement logic
    } else if (name.equals("B")) {
        // Bishop movement logic
    } else if (name.equals("K")) {
        // Knight movement logic
    }
}
```

This design has two major problems.

- Single Responsibility Principle (SRP): The class handles the movement logic of multiple different pieces, along with other responsibilities.
- Open-Closed Principle (OCP): Adding a new piece or a modified chess variation requires changing the existing movement logic.

The instructor also points out that movement is not the only responsibility. Checking whether a piece can move and handling pawn promotion can introduce more logic into the same class.

This can eventually create a Monster Class or God Class.

## 3. Improving the Design Through Inheritance

A natural improvement is to create a parent class and separate child classes for individual chess pieces.

For example:

- `ChessPiece` — parent class
- `Pawn`
- `Rook`
- `Knight`
- `Bishop`
- `Queen`
- `King`

The parent class can be abstract, with an abstract `move()` method. Each child class implements its own movement logic.

Conceptually:

```
abstract class ChessPiece {
    abstract void move();
}

class Rook extends ChessPiece {
    @Override
    void move() {
        // Rook movement logic
    }
}

class Bishop extends ChessPiece {
    @Override
    void move() {
        // Bishop movement logic
    }
}
```

### Advantages

- Movement behaviour is separated into individual classes.
- Adding a new piece can be done by creating a new subclass.
- The design is more extensible and better aligned with SRP and OCP.

For example, if a chess variation introduces a modified bishop that can also move like a knight, we can create a new class instead of adding more conditions to the original `ChessPiece` class.

However, inheritance alone does not guarantee a good design. The next problem introduces Liskov Substitution Principle.

## 4. Liskov Substitution Principle (LSP)

### Definition

The instructor explains LSP using the following idea:

Objects of a child class should be substitutable for objects of the parent class without breaking the expected behaviour.

For example, the design should allow code to work with a `ChessPiece` reference that holds a `Rook`, `Knight`, or `Pawn`, provided the child class follows the behaviour promised by the parent class.

```
ChessPiece piece = new Rook();
piece.move();
```

The important point is not merely that a child class extends a parent class. The child class must also honour the parent's behavioural expectations.

### 4.1 Chess castling example

The instructor introduces a `castle()` function.

Castling is a chess move involving the King and a Rook. It is subject to conditions, including:

- The King and the relevant Rook have not previously moved.
- The King is not currently in check.
- The squares between them are unoccupied.
- The King does not pass through or land on an attacked square.

The transcript emphasises that castling is initiated by the King, while the Rook participates in the move.

Now suppose `castle()` is declared as an abstract method in the parent `ChessPiece` class.

Every child class must implement it:

- `King` implements castling.
- `Pawn` must also implement `castle()`.
- `Knight` must also implement `castle()`.
- `Bishop` must also implement `castle()`.

But most pieces cannot castle.

If those classes throw an exception whenever `castle()` is called, the parent class promises a behaviour that its children cannot meaningfully provide.

This is the design problem: the class hierarchy forces child classes to implement behaviour that does not belong to them.

### 4.2 Movement and invalid arguments

The instructor also distinguishes between an invalid move and an illegal argument.

For example, suppose a Pawn is asked to move backwards. According to the transcript, this can be treated as an illegal argument because moving backwards is outside the Pawn's allowed movement direction.

In other cases, a piece may be given a valid destination coordinate, but the move cannot currently be performed because of the board's state. That can be treated as an invalid move.

The distinction matters because a common parent method may not fit every piece's specific rules equally well.

### 4.3 Why this violates LSP

The design becomes problematic when code cannot safely treat every child as a `ChessPiece` and rely on the parent's methods.

For example, calling `castle()` on a `Pawn` results in an exception, while calling it on a `King` may perform the operation.

The parent class therefore exposes behaviour that is not appropriate for all its subclasses.

Main takeaway: Do not put a method in a parent class merely because some of its child classes need it. The parent-child relationship must preserve meaningful behavioural expectations.

## 5. HTML Converter Example: A Better Use of Inheritance

The instructor revisits the earlier HTML converter example.

Initially, a `TextProcessor` converts text into HTML. Later, the requirement expands to support multiple input formats:

- Text to HTML
- Markdown to HTML
- JSX to HTML
- LaTeX to HTML

One proposed design is to define a common abstract class and create separate converters.

```
abstract class HTMLConverter {
    abstract String convert();
}

class TextToHTMLConverter extends HTMLConverter {
    @Override
    String convert() {
        // Convert text to HTML
        return "<html>Text</html>";
    }
}

class MarkdownToHTMLConverter extends HTMLConverter {
    @Override
    String convert() {
        // Convert Markdown to HTML
        return "<html>Markdown</html>";
    }
}
```

The implementations above are illustrative; the transcript focuses on the class structure rather than the conversion algorithms.

### Why this design is better

Every converter is expected to convert its input into HTML. Therefore, the child classes fulfil the common behaviour promised by the parent class.

The code can work with an `HTMLConverter` reference without needing special checks for each converter type.

Important distinction: Inheritance works well when child classes can honour the contract defined by their parent. It becomes problematic when the parent forces a child to support behaviour that does not apply to it.

## 6. Custom Chess Variations: A Limitation of the Inheritance Design

The instructor returns to chess and introduces another requirement: custom chess variations.

Suppose a modified chess game allows:

- A Rook to move like a Rook and a Knight.
- A Knight to move like a Bishop and a King.
- A piece to support several movement behaviours.
- Additional custom movement rules.

With the inheritance-based design, each piece has its movement behaviour tied to its specific class.

As the number of combinations increases, the design becomes harder to extend. Creating a separate subclass for every possible combination can lead to many classes and duplicated logic.

The transcript uses this problem to introduce a broader design concern: behaviour should not always be tightly coupled to the class that represents an object.

A similar problem appears in payment systems.

## 7. Payment System Example: Credit Card Design

Suppose we are designing a payment system with an abstract `CreditCard` class.

The instructor lists common card properties, such as:

- Card number
- Cardholder
- Expiry date
- CVV

The initial design also places several payment operations inside the abstract class:

- `swipeAndPay()`
- `onlinePayment()`
- `tapAndPay()`
- `upiPayment()`
- `refund()`
- `internationalPayment()`

Concrete classes extend `CreditCard`, including:

- `VisaCreditCard`
- `MasterCardCreditCard`
- `RupayCreditCard`
- `DinersCreditCard`
- `AMEXCreditCard`

Each class is expected to implement or override the methods inherited from `CreditCard`.

### 7.1 Problem: Not every card supports every operation

The transcript uses hypothetical payment scenarios to illustrate the problem:

- UPI payment is described as supported only by RuPay.
- Refunds are described as unavailable for Diners in the example.
- International payments are described as unavailable for RuPay.
- The international-payment algorithm differs between card types; for example, the hypothetical MasterCard implementation requires a PIN, while other implementations do not.

These scenarios are used as design examples in the transcript, not as verified statements about current payment-network capabilities.

### 7.2 Problems with the initial design

Problem 1: Forced implementation

If `CreditCard` declares every operation as abstract, each subclass must implement operations it may not support.

For example, a card that does not support international payments may need to throw an exception from `internationalPayment()`.

This creates the same type of design problem seen with `castle()` in the chess example.

Problem 2: Code duplication

Some card types may share the same payment algorithm, while others need small variations.

If every subclass implements its own version, the same logic may be repeated across several classes.

Problem 3: Poor extensibility

Suppose a new payment feature becomes mandatory for every card. The design may require changes across multiple subclasses.

Likewise, if a card stops supporting a feature, the class hierarchy may need further restructuring.

The transcript highlights that separating behaviour by card class does not solve every problem when behaviours overlap across multiple combinations of classes.

## 8. Interface Segregation Principle (ISP)

### Definition

The instructor explains ISP as follows:

No class should be forced to depend on or implement methods it does not need.

In the credit card example, not every card supports every payment operation. Therefore, putting every operation into the common parent class forces some subclasses to provide meaningless implementations.

The solution is to separate optional or specialised behaviours into smaller interfaces.

### 8.1 Keep only common behaviour in `CreditCard`

The abstract `CreditCard` class should contain the properties and behaviours that are genuinely common and mandatory for all supported card types.

Operations that are not common to every card should be separated into interfaces.

### 8.2 Create feature-specific interfaces

The instructor proposes interfaces such as:

- `UPICompatibleCreditCard`
- `RefundCompatibleCreditCard`
- An interface for PIN-based international payments
- An interface for non-PIN international payments

For example:

```
interface UPICompatibleCreditCard {
    void upiPayment();
}

interface RefundCompatibleCreditCard {
    void refund();
}
```

Now, only card classes that support a particular feature need to implement its corresponding interface.

```
class RupayCreditCard extends CreditCard
        implements UPICompatibleCreditCard,
                   RefundCompatibleCreditCard {

    @Override
    public void upiPayment() {
        // UPI payment logic
    }

    @Override
    public void refund() {
        // Refund logic
    }
}
```

This code is an illustrative example of the proposed design. The exact features supported by each card depend on the requirements being modelled.

### 8.3 Why interfaces are used instead of another abstract class

The instructor explains that a card can be both:

- A `CreditCard`
- A `UPICompatibleCreditCard`

Java does not support extending multiple classes. However, a class can implement multiple interfaces.

This allows a card class to combine several independent capabilities without requiring every capability to be part of its parent class.

### Benefits of this design

- Classes implement only the behaviours relevant to them.
- Optional capabilities are separated from common card functionality.
- New capabilities can be introduced through new interfaces.
- The design is more flexible than placing every operation in one large abstract class.

## 9. Remaining Problems After Applying ISP

The instructor points out that introducing interfaces improves the design, but does not automatically resolve every issue.

### Problem 1: A feature can be discontinued

Suppose AMEX stops supporting `tapAndPay()` in the hypothetical example.

If tap-and-pay functionality is embedded in the common card design, several classes and interfaces may need to be updated.

The lesson is to consider that a feature supported today may not be supported by a particular class in the future.

### Problem 2: Code duplication still exists

Suppose Visa and MasterCard use the same refund algorithm, while RuPay uses a different one.

Even after segregating the interfaces, the same refund logic may still be duplicated in Visa and MasterCard.

A similar issue can occur with online payment, swipe-and-pay, and tap-and-pay operations when several card types share the same algorithm.

This leaves two design concerns:

1. Feature changes: How easily can support for a capability be added or removed?
2. Shared algorithms: How can common logic be reused without duplicating it across multiple classes?

The transcript ends at this point, after identifying these remaining problems. It has not yet explained the subsequent design solution.

## 10. Quick Revision: SOLID Principles Covered

| Principle                             | Main idea                                                                                                       | Example from the transcript                                                                  |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| SRP — Single Responsibility Principle | A class should not take on too many unrelated responsibilities.                                                 | One `ChessPiece` class managing every piece's movement.                                      |
| OCP — Open-Closed Principle           | The design should support extensions without repeatedly modifying existing code.                                | Adding modified chess pieces without adding more conditions to the original movement method. |
| LSP — Liskov Substitution Principle   | Child classes should be usable wherever the parent class is expected without breaking its behavioural contract. | A `Pawn` being forced to implement `castle()`.                                               |
| ISP — Interface Segregation Principle | Classes should not be forced to implement or depend on methods they do not need.                                | Separating UPI and refund operations into feature-specific interfaces.                       |

## 11. Key Interview Takeaways

- Start by identifying the responsibilities of a class before writing code.
- Avoid large classes containing many unrelated behaviours.
- Inheritance helps separate behaviour, but it must preserve the parent's behavioural contract.
- Do not force subclasses to implement operations that do not apply to them.
- Use interfaces to represent optional or specialised capabilities.
- Separate common behaviour from class-specific behaviour.
- Applying SOLID principles improves structure, but code duplication and changing feature requirements may still need further design improvements.

Where this transcript leaves off: The design has improved through inheritance and interface segregation, but shared algorithms are still duplicated. The next step would be to explore how common behaviour can be reused while keeping the classes flexible.
