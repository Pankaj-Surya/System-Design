# SOLID Principles in Java — One Consistent Credit Card Example

To make SOLID easier to understand and explain in interviews, we’ll use the same credit card payment system for all five principles. We’ll start with problematic code, understand why it becomes difficult to maintain, and then improve it step by step.

Imagine we’re building a payment system that supports different cards:

- Visa — swipe, tap, online payment, refunds.
- RuPay — swipe, tap, online payment, UPI, refunds.
- Diners Club — swipe, tap, online payment, refunds.

These are illustrative capabilities for our example, not a claim that every real card of each brand has exactly these features.

The interviewer-friendly story is: “Initially, our code works. But as the business adds more cards and payment features, the design starts creating problems. Each SOLID principle helps us solve one specific problem.”

## Our common starting point

```
class CreditCard {
    public void swipeAndPay() {
        System.out.println("Payment using swipe");
    }

    public void tapAndPay() {
        System.out.println("Payment using tap");
    }

    public void onlinePayment() {
        System.out.println("Payment online");
    }

    public void upiPayment() {
        System.out.println("Payment using UPI");
    }

    public void refund() {
        System.out.println("Refund processed");
    }
}
```

At first, this looks convenient. But every card is treated as if it supports every operation. As requirements grow, this design creates the problems that the five SOLID principles help us address.

## 1. S — Single Responsibility Principle (SRP)

One class should have one reason to change.

### The story: Our credit card class is doing too much

Suppose the business asks us to add three features:

1. Process credit card payments.
2. Calculate transaction fees.
3. Generate payment receipts.

A developer adds all three responsibilities to `CreditCard`.

```
class CreditCard {
    public void makePayment(double amount) {
        System.out.println("Payment of " + amount + " processed");
    }

    public double calculateFee(double amount) {
        return amount * 0.02;
    }

    public void generateReceipt(double amount) {
        System.out.println("Receipt generated for " + amount);
    }
}
```

What is the problem?

The class now has multiple reasons to change:

- Payment processing changes when payment rules change.
- Fee calculation changes when the fee policy changes.
- Receipt generation changes when the receipt format changes.

A change in one business area can affect a class responsible for another area.

### Refactoring: Separate responsibilities

```
class CreditCard {
    public void makePayment(double amount) {
        System.out.println("Payment of " + amount + " processed");
    }
}

class FeeCalculator {
    public double calculateFee(double amount) {
        return amount * 0.02;
    }
}

class ReceiptGenerator {
    public void generateReceipt(double amount) {
        System.out.println("Receipt generated for " + amount);
    }
}
```

Now each class has a focused responsibility.

How to explain it in an interview:

> “Initially, my CreditCard class handled payment processing, fee calculation, and receipt generation. I realized that these responsibilities change for different reasons. So, I separated them into focused classes. Now a change in receipt formatting does not require modifying the payment-processing class.”

Remember: SRP is about reasons to change, not simply having one method per class.

## 2. O — Open/Closed Principle (OCP)

Software should be open for extension but closed for modification.

### The story: A new card brand arrives

Our payment system initially supports Visa and RuPay.

```
class PaymentService {
    public void pay(String cardType, double amount) {
        if (cardType.equals("VISA")) {
            System.out.println("Visa payment: " + amount);
        } else if (cardType.equals("RUPAY")) {
            System.out.println("RuPay payment: " + amount);
        }
    }
}
```

Now the business asks us to support Diners Club. We must modify `PaymentService` and add another condition.

Every new card brand may force changes to existing, working code. Over time, the conditional logic grows and becomes harder to maintain.

### Refactoring: Introduce a common abstraction

```
interface CreditCardPayment {
    void pay(double amount);
}

class VisaCard implements CreditCardPayment {
    public void pay(double amount) {
        System.out.println("Visa payment: " + amount);
    }
}

class RuPayCard implements CreditCardPayment {
    public void pay(double amount) {
        System.out.println("RuPay payment: " + amount);
    }
}

class DinersCard implements CreditCardPayment {
    public void pay(double amount) {
        System.out.println("Diners Club payment: " + amount);
    }
}
```

The service no longer needs to know the brand.

```
class PaymentService {
    public void processPayment(
            CreditCardPayment card, double amount) {
        card.pay(amount);
    }
}
```

Usage:

```
PaymentService service = new PaymentService();

service.processPayment(new VisaCard(), 1000);
service.processPayment(new RuPayCard(), 2000);
service.processPayment(new DinersCard(), 3000);
```

When a new card is introduced, we can add a new implementation of `CreditCardPayment` without changing `PaymentService`.

How to explain it in an interview:

> “Initially, I used if-else conditions for every card brand. When new card brands were introduced, I had to keep modifying the same service. I introduced a `CreditCardPayment` interface and moved brand-specific behavior into separate implementations. Now the system is open for adding new card types but closed to changes in the existing payment service.”

Remember: OCP is about adding new behavior without repeatedly modifying stable code. It does not mean existing code can never change.

## 3. L — Liskov Substitution Principle (LSP)

A subtype should be usable wherever its parent type is expected, without breaking the expected behavior.

### The story: Not every card supports UPI

Imagine we create one parent class that claims every credit card supports every payment operation.

```
class CreditCard {
    public void pay(double amount) {
        System.out.println("Card payment: " + amount);
    }

    public void upiPayment(double amount) {
        System.out.println("UPI payment: " + amount);
    }

    public void refund(double amount) {
        System.out.println("Refund: " + amount);
    }
}

class VisaCard extends CreditCard {
}

class RuPayCard extends CreditCard {
}

class DinersCard extends CreditCard {
    @Override
    public void upiPayment(double amount) {
        throw new UnsupportedOperationException(
            "UPI is not supported by this card"
        );
    }
}
```

Consider this code:

```
CreditCard card = new DinersCard();
card.upiPayment(500);
```

The code compiles, but fails at runtime.

The problem is that `DinersCard` cannot honor the behavior promised by the parent class. The parent abstraction promises too much.

### Refactoring: Model only common behavior in the parent

```
interface CreditCardPayment {
    void pay(double amount);
}

interface UpiCompatible {
    void upiPayment(double amount);
}

interface RefundCompatible {
    void refund(double amount);
}
```

Now we model each card according to the capabilities it supports.

```
class VisaCard implements CreditCardPayment, RefundCompatible {
    public void pay(double amount) {
        System.out.println("Visa payment: " + amount);
    }

    public void refund(double amount) {
        System.out.println("Visa refund: " + amount);
    }
}

class RuPayCard implements CreditCardPayment,
        UpiCompatible, RefundCompatible {
    public void pay(double amount) {
        System.out.println("RuPay payment: " + amount);
    }

    public void upiPayment(double amount) {
        System.out.println("RuPay UPI payment: " + amount);
    }

    public void refund(double amount) {
        System.out.println("RuPay refund: " + amount);
    }
}

class DinersCard implements CreditCardPayment, RefundCompatible {
    public void pay(double amount) {
        System.out.println("Diners Club payment: " + amount);
    }

    public void refund(double amount) {
        System.out.println("Diners Club refund: " + amount);
    }
}
```

A UPI operation now accepts only a card that supports UPI.

```
class UpiPaymentService {
    public void payUsingUpi(UpiCompatible card, double amount) {
        card.upiPayment(amount);
    }
}
```

```
UpiPaymentService service = new UpiPaymentService();

service.payUsingUpi(new RuPayCard(), 500);

// Does not compile:
// service.payUsingUpi(new DinersCard(), 500);
```

The invalid operation is prevented by the type system instead of failing unexpectedly at runtime.

How to explain it in an interview:

> “Initially, every card inherited a UPI operation, but some cards could not support it and had to throw exceptions. That violated LSP because the subtype could not honor the parent’s contract. I redesigned the abstractions so that all cards share common payment behavior, while UPI and refund capabilities are represented separately. Now code can safely use a card through the capability it actually supports.”

Remember: LSP is about behavioral correctness when substituting one implementation for another. Capability-specific interfaces help us achieve it.

## 4. I — Interface Segregation Principle (ISP)

Clients should not be forced to depend on methods they do not use.

### The story: One enormous card interface

Suppose we try to solve our card problem by creating a single interface with every possible operation.

```
interface CreditCardOperations {
    void pay(double amount);
    void upiPayment(double amount);
    void refund(double amount);
    void internationalPinPayment(double amount);
}
```

Every card must implement all four methods.

```
class DinersCard implements CreditCardOperations {
    public void pay(double amount) {
        System.out.println("Diners payment");
    }

    public void upiPayment(double amount) {
        throw new UnsupportedOperationException();
    }

    public void refund(double amount) {
        System.out.println("Diners refund");
    }

    public void internationalPinPayment(double amount) {
        throw new UnsupportedOperationException();
    }
}
```

Diners Club is forced to implement operations it does not support. Those methods become misleading, unnecessary implementations.

### Refactoring: Split the interface by capability

```
interface BasicCardPayment {
    void pay(double amount);
}

interface UpiPayment {
    void upiPayment(double amount);
}

interface Refund {
    void refund(double amount);
}

interface InternationalPinPayment {
    void payWithInternationalPin(double amount);
}
```

Each card implements only the interfaces it needs.

```
class VisaCard implements BasicCardPayment, Refund {
    public void pay(double amount) {
        System.out.println("Visa payment");
    }

    public void refund(double amount) {
        System.out.println("Visa refund");
    }
}

class RuPayCard implements BasicCardPayment, UpiPayment, Refund {
    public void pay(double amount) {
        System.out.println("RuPay payment");
    }

    public void upiPayment(double amount) {
        System.out.println("RuPay UPI payment");
    }

    public void refund(double amount) {
        System.out.println("RuPay refund");
    }
}

class DinersCard implements BasicCardPayment, Refund {
    public void pay(double amount) {
        System.out.println("Diners payment");
    }

    public void refund(double amount) {
        System.out.println("Diners refund");
    }
}
```

Now a client that needs only refunds can depend on the `Refund` interface, without depending on UPI or international PIN payment.

How to explain it in an interview:

> “The original interface forced every card to implement all payment capabilities, even when some were not supported. I split the large interface into smaller, capability-based interfaces. Each card now implements only the operations relevant to it, and clients depend only on the capabilities they need.”

Remember the difference between LSP and ISP:

- LSP: Can an implementation safely honor the behavior expected by its abstraction?
- ISP: Is the abstraction forcing implementations or clients to depend on operations they do not need?

They often lead to similar refactoring, but solve different problems.

## 5. D — Dependency Inversion Principle (DIP)

High-level modules should depend on abstractions, not concrete implementations.

### The story: Our payment service is tied to Visa

Suppose our high-level business service directly creates a Visa card.

```
class VisaCard {
    public void pay(double amount) {
        System.out.println("Visa payment: " + amount);
    }
}

class PaymentService {
    private final VisaCard card = new VisaCard();

    public void processPayment(double amount) {
        card.pay(amount);
    }
}
```

Now `PaymentService` is tightly coupled to `VisaCard`.

If we want to use RuPay, we might need to modify `PaymentService`. Testing the service independently is also harder because it directly constructs its dependency.

### Refactoring: Depend on an abstraction

We already introduced the common `CreditCardPayment` interface. We can reuse it here.

```
interface CreditCardPayment {
    void pay(double amount);
}

class VisaCard implements CreditCardPayment {
    public void pay(double amount) {
        System.out.println("Visa payment: " + amount);
    }
}

class RuPayCard implements CreditCardPayment {
    public void pay(double amount) {
        System.out.println("RuPay payment: " + amount);
    }
}

class PaymentService {
    private final CreditCardPayment card;

    public PaymentService(CreditCardPayment card) {
        this.card = card;
    }

    public void processPayment(double amount) {
        card.pay(amount);
    }
}
```

Usage:

```
PaymentService visaService =
        new PaymentService(new VisaCard());

visaService.processPayment(1000);

PaymentService rupayService =
        new PaymentService(new RuPayCard());

rupayService.processPayment(2000);
```

The service depends on the interface rather than a specific card brand. The implementation is supplied from outside.

How to explain it in an interview:

> “Initially, my high-level `PaymentService` directly created a `VisaCard`, making it tightly coupled to one implementation. I introduced a `CreditCardPayment` abstraction and passed the required card through the constructor. Now the service works with any compatible implementation, and I can substitute a test implementation when testing.”

Remember: Dependency Inversion is the design principle. Constructor injection is one technique used to implement it; they are not the same thing.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.baeldung.com\&sz=32)

Baeldung

+1



## Final interview revision

| Principle | Credit card problem                                            | What we do                                             |
| --------- | -------------------------------------------------------------- | ------------------------------------------------------ |
| S — SRP   | One class handles payments, fees and receipts                  | Separate responsibilities                              |
| O — OCP   | Every new card requires another `if-else`                      | Add implementations behind an abstraction              |
| L — LSP   | A card cannot honor an inherited operation                     | Ensure implementations honor their contracts           |
| I — ISP   | One interface forces cards to implement unsupported operations | Split interfaces by capability                         |
| D — DIP   | Payment service directly creates a specific card               | Depend on an abstraction and inject the implementation |

A useful interview technique is to explain each principle in this order: business requirement → problematic code → why it is a problem → refactoring → benefit.

The examples deliberately reuse the same card domain, but each principle focuses on a different design concern. SOLID principles are guidelines rather than a requirement to create an interface or class for every small operation.
