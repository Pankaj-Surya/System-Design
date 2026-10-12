# Summary Notes: Low-Level Design (LLD) and SOLID Principles

These notes follow the same language and terminology used in the transcript, with the main concepts, examples, and interview points organized for easy revision.

## 1. Why Low-Level Design is Important

If you have a good grasp of SOLID principles and can think analytically, you have a good chance of doing well in a Low-Level Design interview, even if you don't know much about design patterns.

The main focus is on problem-solving skills, code structure, maintainability, and extensibility.

## 2. High-Level Design vs Low-Level Design

Whenever we build a feature or a product, there are two main design aspects.

| High-Level Design (HLD)                                           | Low-Level Design (LLD)                                             |
| ----------------------------------------------------------------- | ------------------------------------------------------------------ |
| Gives a bird's-eye view of the architecture.                      | Focuses on code structure and code separation.                     |
| Explains the components involved in a system.                     | Explains how classes, functions, and modules should be structured. |
| Focuses on what each component does.                              | Focuses on how the code is organized and maintained.               |
| Coding implementation details are generally not covered.          | Class and function definitions are important.                      |
| Focuses on the overall architecture of an application or feature. | Focuses on making code maintainable and extendible.                |

### Why do we need Low-Level Design?

In big companies and midsize startups, one significant problem is code maintenance.

- As we keep adding more code, the codebase can become messy and difficult to manage.
- After a year or a year and a half, an early-stage startup's codebase might become difficult to maintain.
- Big companies don't want their codebase to become unmanageable after working on a product for three or four years.
- LLD helps us structure and separate code so that it remains maintainable and extendible.

### What do we write in an LLD design document?

A design document may contain:

- The high-level architecture of the feature.
- The modules in the codebase or project that need to be changed.
- Probable class and function definitions.
- Code snippets representing how the implementation will be structured.
- Details about how the code will remain maintainable and extendible.

Important: You generally don't need to write the final, exact, end-to-end implementation logic in the design document. The focus is on how you are going to structure the code.

## 3. Low-Level Design Interviews

In an SD2 interview, you might get a round called:

- Low-Level Design round
- Low-Level DSA round
- Machine Coding round

The main focus of these rounds is how you structure your code so that it is maintainable and extendible.

### What if the interviewer expects running code?

- If the company expects final running code within a limited time, the problem statement may be relatively easier.
- If the company doesn't expect a final runnable solution, the problem may be trickier.
- In both situations, the majority of the scoring focuses on how you structure the code and why you made particular design decisions.

You should have good reasoning for moving from a bad implementation to a better implementation. Otherwise, it can look like you have simply learned something and copy-pasted it without understanding it.

### Key takeaway

LLD is more of a problem-solving round where you identify bad code and improve it into good code.

The interviewer wants to understand whether you naturally think about design philosophies and code maintenance when you write code.

## 4. What are Design Patterns?

The problems related to code maintenance have already been studied, and people have developed solutions for them.

Design patterns are coding solutions or code-maintenance solutions that solve specific types of code-maintenance problems.

Examples of design patterns mentioned in the transcript:

- Strategy Pattern
- Factory Pattern
- Adapter Pattern

### Connection with algorithms

Think about algorithmic problems:

- A problem statement doesn't tell you whether it is a Dynamic Programming (DP), Greedy, or Divide and Conquer problem.
- You have to identify the problem and determine which approach is suitable.
- Similarly, in LLD, you have to identify a code-maintenance problem and decide which design pattern can solve it.

Just like binary search is an approach for solving certain types of problems, design patterns are approaches for solving certain types of code-maintenance problems.

The important part is not just knowing the design patterns. It is identifying when a particular pattern fits your situation.

## 5. What are SOLID Principles?

A good majority of design patterns are directly based on SOLID principles.

SOLID principles are important because:

- They help us make better coding decisions.
- They help us understand why a particular class or function should be implemented in a certain way.
- They provide reasoning for improving code quality and maintainability.
- They are useful both in interviews and in regular day-to-day work.

Apart from SOLID, other principles also exist, such as DRY (Don't Repeat Yourself).

### Full form of SOLID

| Letter | Principle                       | Abbreviation |
| ------ | ------------------------------- | ------------ |
| S      | Single Responsibility Principle | SRP          |
| O      | Open-Closed Principle           | OCP          |
| L      | Liskov Substitution Principle   | LSP          |
| I      | Interface Segregation Principle | ISP          |
| D      | Dependency Inversion Principle  | DIP          |

Knowing the full forms is useful, but simply memorizing them is not enough. You need to understand why these principles are important and how they help improve problematic code.

### Approach used in the transcript

Instead of only learning definitions, the approach is to:

1. Write problematic code.
2. Analyze the problems in that code.
3. Understand why those problems exist.
4. Improve the code.
5. Connect the improvement with a SOLID principle.

This approach makes it easier to understand the principles through practical coding situations.

## 6. Single Responsibility Principle (SRP)

The first principle of SOLID is the Single Responsibility Principle (SRP).

### Definition

> A class or a piece of code should have one and only one reason to change.

The same theory can also be applied to a function, including in functional programming.

The main idea is that a class should not try to do too many different things.

### 6.1 Problematic Code: Employee Class

Suppose we are building an employee management system, similar to products like Workday or Rippling.

We start with an `Employee` class that contains:

- Employee ID
- Employee name
- Employee address
- `printPerformanceReport()`
- `computeSalary()`
- `updateEmployeeData()`
- `printEmployeeDetails()`

At first, this looks like a pretty regular employee class. However, it has a problem: the class is trying to do too many things.

### 6.2 Why is this a problem?

Because the class is doing too many things, there are too many reasons to change the same piece of code.

Consider these scenarios:

| Function                   | Reason for change                                                 |
| -------------------------- | ----------------------------------------------------------------- |
| `computeSalary()`          | Tax slabs or the salary calculation logic changes.                |
| `updateEmployeeData()`     | Data storage changes from an Excel sheet to a dedicated database. |
| `printPerformanceReport()` | The report format changes from a Word file to Excel or PDF.       |
| Employee data properties   | Employee details or the way employee data is managed changes.     |

For example, suppose the government changes the tax slabs from 5%, 10%, 15%, and 30% to a different structure. We may need to update the salary calculation function.

Similarly, if the company moves from storing employee data in Excel to a database, the employee data update function may need to change.

If the performance report format changes, we may need to update the report printing function.

Because all these responsibilities are inside the same class, changes in unrelated requirements can affect the same class.

This violates the Single Responsibility Principle.

### 6.3 Better Code: Separate Responsibilities

Instead of putting everything inside one `Employee` class, we can divide the responsibilities into separate classes.

Employee

Basic employee data and CRUD operations

EmployeeSalaryCalculator

Responsible for computing salary.

EmployeePerformanceReportGenerator

Responsible for generating performance reports.

The responsibilities are now separated:

- `Employee` handles basic employee data creation and retrieval, updating, and other employee data operations.
- `EmployeeSalaryCalculator` handles salary calculation.
- `EmployeePerformanceReportGenerator` handles performance report generation.

### Example code

The following example uses Java, as in the transcript. The principles are language-agnostic and can also be applied in TypeScript.

Employee.java

```
public class Employee {
    private int id;
    private String name;
    private String address;

    public Employee(int id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    public int getEmployeeId() {
        return id;
    }

    public String getEmployeeName() {
        return name;
    }

    public String getEmployeeAddress() {
        return address;
    }

    public void setEmployeeName(String name) {
        this.name = name;
    }

    public void setEmployeeAddress(String address) {
        this.address = address;
    }
}
```

EmployeeSalaryCalculator.java

```
public class EmployeeSalaryCalculator {

    public double computeSalary(Employee employee) {
        // Salary calculation logic
        // Based on employee level, years of service,
        // taxes, bonuses, etc.
        return 0.0;
    }
}
```

EmployeePerformanceReportGenerator.java

```
public class EmployeePerformanceReportGenerator {

    public void printPerformanceReport(Employee employee) {
        // Fetch performance feedback and ratings
        // Calculate average ratings
        // Generate and print the report
    }
}
```

### 6.4 How does the better code follow SRP?

Suppose the performance rating system changes from a five-star rating system to categories such as “Meets Expectations” and “Exceeds Expectations.”

- We update `EmployeePerformanceReportGenerator`.
- The `Employee` class does not need to change.
- The salary calculator does not need to change.

Similarly, if salary calculation logic changes, we update the salary calculator without modifying the performance report generator.

This is how we achieve better separation of responsibilities.

Remember: The point is not the algorithm itself. The point is how we maintain and structure the code.

## 7. SRP Depends on the Reference Frame

Whenever we discuss principles related to code maintenance and code quality, everything depends on our reference frame.

For example, someone might argue that `EmployeeSalaryCalculator` can still have multiple reasons to change:

- Taxation changes.
- Employee level distribution changes.
- A new bonus structure is introduced.

All these changes affect salary calculation.

So, how much should we separate the code?

The important point is that there is no need to keep splitting a class endlessly. You should have logical reasoning for your design decisions.

The transcript compares code cleanliness with keeping a desk clean: what feels clean to one person might not feel clean to another person.

### Important guidelines

- There can always be more segregation.
- Don't keep splitting classes without a logical reason.
- Focus on whether the overall code is maintainable and understandable.
- If someone suggests further segregation, consider whether it solves a real problem.
- In an interview, explain your reasoning instead of blindly following a rule.

Having multiple files is not a problem. Large codebases can contain thousands or even lakhs of files.

Having code in multiple files is not an issue; having code that is not maintainable is an issue.

## 8. Monster Class (God Class)

A monster class is a single class that takes too many responsibilities, essentially acting like a God Class.

It has a large amount of functionality crammed into one place, violating the Single Responsibility Principle.

### Why is a monster class problematic?

- It becomes difficult to manage.
- It becomes difficult to maintain.
- Changes in one responsibility can affect other functionality.
- It becomes harder to understand the class and make modifications safely.

For example, an `Employee` class that manages employee data, salary calculations, performance reports, database updates, and report formatting can grow into a monster class.

### How do we handle it?

Separate the responsibilities into classes with clear purposes, such as:

- `Employee`
- `EmployeeSalaryCalculator`
- `EmployeePerformanceReportGenerator`

The goal is to keep the code manageable, maintainable, and extendible.

## 9. Important Interview Takeaways

Understand the reasoning behind the code

Don't just memorize SOLID definitions or copy-paste implementations. Be able to explain why a particular implementation is better.

Focus on maintainability and extensibility

The main objective of LLD is to structure code so that future changes can be handled more easily.

Identify the problem before applying a principle

Just as you identify whether an algorithmic problem needs DP or binary search, identify the code-maintenance problem before deciding how to improve the implementation.

Don't over-engineer

More classes and more files do not automatically mean better code. Every design decision should have a logical reason.

## 10. Quick Revision

| Concept         | What to remember                                              |
| --------------- | ------------------------------------------------------------- |
| HLD             | Bird's-eye view of the architecture.                          |
| LLD             | Code structure, classes, functions, and maintainability.      |
| Design patterns | Coding solutions for specific code-maintenance problems.      |
| SOLID           | Five principles that help guide code design decisions.        |
| SRP             | One and only one reason to change a class or piece of code.   |
| Monster class   | A class with too many responsibilities.                       |
| Good code       | Maintainable, extendible, readable, and logically structured. |

### Final conclusion

The central idea of this lecture is that Low-Level Design is about identifying problematic code and improving it into maintainable and extendible code.

SOLID principles provide a way to reason about these improvements. The first principle, SRP, teaches us that when a class is trying to do too many things, we should consider separating its responsibilities so that changes in one area do not unnecessarily affect other areas.

Note: The provided transcript ends while the next example is being introduced, so these notes cover the material available in the transcript, including SRP, and do not assume the missing continuation.
