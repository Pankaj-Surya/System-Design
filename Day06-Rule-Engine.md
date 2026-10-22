# Java Summary Notes: Expense Validation Rule Engine

These notes summarize the lecture in an interview-friendly format, focusing on the problem-solving approach, Java class design, SOLID principles, and implementation flow.

## 1. The problem we are solving

Imagine we are building a travel expense management system. A user submits expenses for a trip, and our application must determine whether those expenses violate any business rules.

Examples:

- A restaurant expense must not exceed ₹75.
- Certain expense categories may be disallowed.
- The total expenses for a trip must not exceed a maximum budget.
- Some rules apply to every expense, regardless of category.

The challenge is to design the system so that new rules can be added without rewriting the entire validation engine.

## 2. Why not create a separate class for every expense type?

A straightforward approach might create classes such as:

- `RestaurantExpense`
- `AirfareExpense`
- `HotelExpense`

However, if these classes have the same fundamental behavior and differ only by their category, creating a class for each type can introduce unnecessary complexity.

Problems include:

- Repeated code and similar test cases.
- More classes to maintain.
- More changes when new expense categories are introduced.
- Difficulty supporting rules that apply across multiple expense categories.

Better approach: Represent an expense using a common `Expense` class and represent each validation rule as a separate implementation of a rule interface.

This separates the data being validated from the behavior used to validate it.

## 3. Overall class design

RuleManagerRunner

Coordinates the workflow and obtains configuration

RuleEngine (interface)

Defines the evaluation contract

SimpleRuleEngine

Applies expense-level, global, and trip-level rules

Supporting classes

Expense

Expense data and category

Violation

Validation failure details

RuleRegistry

Maps categories to rules

Rule interfaces

Define validation behavior

The important design decision is to separate rule configuration, rule behavior, evaluation, and orchestration.

## 4. Core Java classes and interfaces

### A. `Expense` — the data model

An expense contains the information required for validation.

```
public class Expense {
    private final String id;
    private final String tripId;
    private final String type;
    private final double amount;

    public Expense(String id, String tripId,
                   String type, double amount) {
        this.id = id;
        this.tripId = tripId;
        this.type = type;
        this.amount = amount;
    }

    public String getId() { return id; }
    public String getTripId() { return tripId; }
    public String getType() { return type; }
    public double getAmount() { return amount; }
}
```

For a production financial application, prefer `BigDecimal` over `double` for monetary amounts.

### B. `Violation` — validation failure details

A violation describes why an expense or trip failed validation.

```
public class Violation {
    private final String message;

    private Violation(String message) {
        this.message = message;
    }

    public static Violation of(String message) {
        return new Violation(message);
    }

    public String getMessage() {
        return message;
    }
}
```

The static `of()` method is a factory method. It provides a convenient way to create a violation without exposing the constructor.

### C. `ExpenseRule` — rules for individual expenses

Each expense rule checks one expense and returns an optional violation.

```
import java.util.Optional;

public interface ExpenseRule {
    Optional<Violation> check(Expense expense);
}
```

Why `Optional<Violation>`? One expense checked against one rule can either produce a violation or produce no violation.

For example, a restaurant expense may violate a maximum-amount rule.

### D. `TripRule` — rules for an entire trip

Trip-level rules receive a collection of expenses rather than one expense.

```
import java.util.List;
import java.util.Optional;

public interface TripRule {
    Optional<Violation> check(List<Expense> expenses);
}
```

For example, the trip budget rule checks the combined amount of all expenses.

The distinction is important:

| Rule type     | Input              | Result                |
| ------------- | ------------------ | --------------------- |
| `ExpenseRule` | One expense        | Zero or one violation |
| `TripRule`    | A list of expenses | Zero or one violation |

If a trip rule needs to report multiple distinct violations, its return type can instead be `List<Violation>`.

## 5. Implementing the rules

### A. Disallow rule

Some expense categories are not permitted at all.

```
import java.util.Optional;

public class DisallowRule implements ExpenseRule {

    @Override
    public Optional<Violation> check(Expense expense) {
        return Optional.of(
            Violation.of(
                "Expense type " + expense.getType()
                + " is not allowed. Expense ID: "
                + expense.getId()
            )
        );
    }
}
```

This rule always reports a violation. The registry determines which expense categories should use it.

### B. Maximum amount rule

```
import java.util.Optional;

public class MaxAmountRule implements ExpenseRule {

    private final double maximumAmount;

    public MaxAmountRule(double maximumAmount) {
        this.maximumAmount = maximumAmount;
    }

    @Override
    public Optional<Violation> check(Expense expense) {
        if (expense.getAmount() > maximumAmount) {
            return Optional.of(
                Violation.of(
                    "Expense exceeds maximum amount. Expense ID: "
                    + expense.getId()
                )
            );
        }

        return Optional.empty();
    }
}
```

For example, if restaurant expenses have a limit of ₹75:

- ₹10 → accepted by this rule.
- ₹50 → accepted by this rule.
- ₹100 → violation.

Notice that the rule's logic is reusable. Different categories can use different configured maximum amounts without duplicating the validation algorithm.

### C. Trip total maximum rule

```
import java.util.List;
import java.util.Optional;

public class TripTotalMaxRule implements TripRule {

    private final double maximumAmount;

    public TripTotalMaxRule(double maximumAmount) {
        this.maximumAmount = maximumAmount;
    }

    @Override
    public Optional<Violation> check(List<Expense> expenses) {
        double total = 0;

        for (Expense expense : expenses) {
            total += expense.getAmount();
        }

        if (total > maximumAmount) {
            return Optional.of(
                Violation.of(
                    "Total trip expenses exceed the maximum budget."
                )
            );
        }

        return Optional.empty();
    }
}
```

This rule calculates the total and compares it with the trip budget.

One important assumption: all expenses passed to this rule belong to the same trip. A robust implementation should validate that assumption or retrieve expenses for a specific trip before evaluation.

## 6. `RuleRegistry` — centralized rule configuration

The registry stores the mapping between expense categories and their rules.

For example:

- Restaurant → maximum amount ₹75.
- Airfare → disallow rule.
- Entertainment → disallow rule.

```
import java.util.List;
import java.util.Map;

public class RuleRegistry {

    public static Map<String, List<ExpenseRule>>
    getExpenseRuleRegistry() {

        return Map.of(
            "RESTAURANT",
            List.of(new MaxAmountRule(75)),

            "AIRFARE",
            List.of(new DisallowRule()),

            "ENTERTAINMENT",
            List.of(new DisallowRule())
        );
    }

    public static List<ExpenseRule> getGlobalExpenseRules() {
        return List.of();
    }

    public static List<TripRule> getTripRules() {
        return List.of(new TripTotalMaxRule(500));
    }
}
```

Here, the registry uses static methods for simplicity, as in the lecture. A more configurable production system could load rules from application configuration or a database.

Registry pattern takeaway: A registry is a centralized lookup mechanism for configuration or objects. In this example, it is primarily a way to keep rule mappings separate from the evaluation algorithm. The name itself is less important than the separation of responsibilities.

## 7. `RuleEngine` — the abstraction

The lecture emphasizes that the rule engine should be an interface rather than a concrete class that callers depend on directly.

```
import java.util.List;
import java.util.Map;

public interface RuleEngine {

    List<Violation> evaluate(
        List<Expense> expenses,
        Map<String, List<ExpenseRule>> expenseRuleRegistry,
        List<ExpenseRule> globalExpenseRules,
        List<TripRule> tripRules
    );
}
```

Why use an interface?

Suppose we initially have a `SimpleRuleEngine`. Later, we develop a faster implementation.

```
public class SimpleRuleEngine implements RuleEngine {
    // Straightforward validation algorithm
}
```

A future `OptimizedRuleEngine` could implement the same interface. The caller would not need to know which implementation it uses.

This demonstrates Dependency Inversion Principle (DIP): high-level code depends on the `RuleEngine` abstraction rather than directly on a particular engine implementation.

## 8. `SimpleRuleEngine` — the evaluation algorithm

The lecture's evaluation flow has three stages:

## 1

Expense-level validation

For every expense, run the rules mapped to its category.

## 2

Global expense validation

Apply rules that apply to every expense, regardless of category.

## 3

Trip-level validation

Run each trip rule against the complete collection of expenses.

Here is a complete implementation of the evaluation logic:

```
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class SimpleRuleEngine implements RuleEngine {

    @Override
    public List<Violation> evaluate(
            List<Expense> expenses,
            Map<String, List<ExpenseRule>> expenseRuleRegistry,
            List<ExpenseRule> globalExpenseRules,
            List<TripRule> tripRules) {

        List<Violation> violations = new ArrayList<>();

        // 1. Expense-level rules
        for (Expense expense : expenses) {
            List<ExpenseRule> rules =
                expenseRuleRegistry.getOrDefault(
                    expense.getType(), List.of()
                );

            checkExpenseAgainstRules(
                expense, rules, violations
            );

            // 2. Global rules
            checkExpenseAgainstRules(
                expense, globalExpenseRules, violations
            );
        }

        // 3. Trip-level rules
        for (TripRule rule : tripRules) {
            Optional<Violation> violation =
                rule.check(expenses);

            violation.ifPresent(violations::add);
        }

        return violations;
    }

    private void checkExpenseAgainstRules(
            Expense expense,
            List<ExpenseRule> rules,
            List<Violation> violations) {

        for (ExpenseRule rule : rules) {
            rule.check(expense).ifPresent(violations::add);
        }
    }
}
```

A few key details:

- `getOrDefault()` returns an empty rule list when no category-specific rules are configured.
- `Optional.ifPresent()` adds a violation only when the rule finds one.
- The engine collects violations rather than stopping at the first failure.
- The rule engine does not decide where configuration comes from. It receives the configured rules as inputs.

The lecture discusses a straightforward algorithm rather than optimizing prematurely. With \\(E\\) expenses and an average of \\(R\\) applicable rules per expense, the expense checks take approximately \\(O(E \times R)\\), plus the cost of trip-level rules.

## 9. `RuleManagerRunner` — coordinating the workflow

The manager acts as the caller of the rule engine. It obtains configuration, invokes the engine, and handles the result.

```
import java.util.List;
import java.util.Map;

public class RuleManagerRunner {

    private final RuleEngine ruleEngine;

    public RuleManagerRunner(RuleEngine ruleEngine) {
        this.ruleEngine = ruleEngine;
    }

    public List<Violation> run(List<Expense> expenses) {

        Map<String, List<ExpenseRule>> expenseRules =
            RuleRegistry.getExpenseRuleRegistry();

        List<ExpenseRule> globalRules =
            RuleRegistry.getGlobalExpenseRules();

        List<TripRule> tripRules =
            RuleRegistry.getTripRules();

        return ruleEngine.evaluate(
            expenses,
            expenseRules,
            globalRules,
            tripRules
        );
    }
}
```

The constructor injects the engine dependency. The manager does not need to instantiate `SimpleRuleEngine` internally.

A simple `main()` method can demonstrate this:

```
import java.util.List;

public class Main {

    public static void main(String[] args) {

        List<Expense> expenses = List.of(
            new Expense("E1", "T1", "RESTAURANT", 10),
            new Expense("E2", "T1", "RESTAURANT", 50),
            new Expense("E3", "T1", "RESTAURANT", 100)
        );

        RuleEngine engine = new SimpleRuleEngine();
        RuleManagerRunner manager =
            new RuleManagerRunner(engine);

        List<Violation> violations = manager.run(expenses);

        violations.forEach(
            violation -> System.out.println(
                violation.getMessage()
            )
        );
    }
}
```

With the example configuration, the ₹100 restaurant expense violates the ₹75 category limit. The total of ₹160 does not violate the ₹500 trip budget.

## 10. SOLID principles demonstrated

| Principle                 | Application in this design                                                                                |
| ------------------------- | --------------------------------------------------------------------------------------------------------- |
| S — Single Responsibility | The registry stores mappings, rules validate, the engine evaluates, and the manager coordinates.          |
| O — Open/Closed           | New rule implementations can be introduced and registered without rewriting the engine's evaluation flow. |
| L — Liskov Substitution   | Different `RuleEngine` implementations should honor the same evaluation contract.                         |
| I — Interface Segregation | `ExpenseRule` and `TripRule` expose different contracts for different validation responsibilities.        |
| D — Dependency Inversion  | The manager depends on `RuleEngine`, not directly on `SimpleRuleEngine`.                                  |

A useful interview distinction: SRP does not mean a class can never change. It means a class should have one primary responsibility. If the evaluation algorithm changes, changing the rule engine can be appropriate.

## 11. Common interview questions from the lecture

Q1. Why do we need a registry if we already have rule classes?

Rule classes define how to validate. The registry defines which rules apply to which expense categories.

Q2. Why are global rules separate from category-specific rules?

Global rules apply to every expense, so they should not need to be repeated in every category mapping.

Q3. Why is the rule engine an interface?

To allow different implementations to be substituted without tightly coupling the manager to one concrete algorithm.

Q4. Why does the trip rule receive a list of expenses?

Because trip-level validation may require aggregate information, such as the sum of all expenses.

Q5. Should we optimize the algorithm immediately?

Not necessarily. First implement a correct, understandable design. Discuss input size, complexity, and optimization opportunities with the interviewer before adding complexity.

Q6. Should the engine fetch configuration itself?

In this design, no. Configuration retrieval belongs to the coordinating layer, leaving the engine focused on evaluation.

## 12. Important lessons from the lecture

- Start with the business requirements before designing classes.
- Avoid creating classes solely because categories have different names.
- Separate data models from validation behavior.
- Use interfaces to represent interchangeable behaviors.
- Keep category-specific, global, and trip-level rules distinct.
- Use a registry to centralize rule mappings.
- Use constructor injection to make dependencies replaceable and testing easier.
- Add utility checks, such as verifying that all expenses belong to the same trip, when the requirements call for them.
- Test the system with normal cases, boundary values, disallowed categories, empty lists, and multiple simultaneous violations.
- Practice implementing the solution yourself rather than memorizing the code. In an interview, you should be able to explain why each abstraction exists and what would change when a new rule is introduced.

Interview-ready conclusion: “I modeled expenses as data, validation policies as separate rule implementations, and the evaluation algorithm behind a `RuleEngine` interface. A registry maps expense categories to their rules, while a manager coordinates configuration and evaluation. This separation makes the solution extensible, testable, and aligned with the SOLID principles.”
