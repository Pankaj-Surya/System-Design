# Summary Notes: SOLID Principles — SRP and OCP

These notes continue from the previous transcript, using the same language and terminology as the lecture. The main focus is on practical examples, problematic code, and how we improve the code by applying SOLID principles.

## 1. Single Responsibility Principle (SRP) — More Examples

SRP states that a class or a piece of code should have one and only one reason to change.

The idea is not that every function or every method needs a separate class. The idea is to identify whether a class is handling unrelated responsibilities that may change for different reasons.

### Example 1: User Manager and Activity Logging

Suppose we have a `UserManager` class with the following functions:

- `addUser()`
- `deleteUser()`
- `updateUser()`
- `getUser()`
- `logUserActivity()`

Initially, the `UserManager` class looks like a class that handles CRUD operations on users.

However, when we introduce `logUserActivity()`, the class starts handling another responsibility: activity logging.

Activity logging can become complicated over time:

- Initially, we log when a user clicks somewhere.
- Later, we may also log scrolling, mouse hovers, and how much time a user spends on the platform.
- Initially, logs may be stored in a log file.
- Later, we may want to store them in a database or another data store.

Every time the activity logging logic changes, we have to modify the same `UserManager` class.

This indicates a possible violation of SRP.

### Better code structure

Separate activity logging into its own class.

UserManager

Add, delete, update, and get users

ActivityLogger

Records user activity and handles logging logic

Example:

```
public class UserManager {
    public void addUser(User user) {
        // Add user
    }

    public void deleteUser(User user) {
        // Delete user
    }

    public void updateUser(User user) {
        // Update user
    }

    public User getUser(int id) {
        // Fetch user
        return null;
    }
}
```

```
public class ActivityLogger {
    public void logUserActivity(User user, String activity) {
        // Record user activity
        // Write logs to the configured storage
    }
}
```

Now, if the activity logging logic changes, we can modify `ActivityLogger` without unnecessarily modifying `UserManager`.

Key takeaway: Separate responsibilities based on their reasons to change, not simply because there are multiple functions in a class.

## 2. SRP Example: HTML Converter

Suppose we have an `HTMLConverter` class that performs three main tasks:

1. Reads text from a file.
2. Converts the text into HTML.
3. Writes the converted content to a file.

The conversion logic works like this:

- Read text from a file.
- Split the text into paragraphs.
- Wrap each paragraph inside HTML paragraph tags (`<p>`).
- Add a break tag after each paragraph.
- Write the resulting HTML into an output file.

At first, this looks like a simple implementation. However, the class handles both file I/O and text conversion.

### What is the problem?

Suppose the input file changes from plain text to Markdown or XML.

The conversion logic may need to change.

Similarly, if the way we read or write files changes, the same class may need to change again.

The issue is not necessarily that reading and writing must be separate classes. Reading and writing are related operations and can reasonably stay together.

The bigger problem is that file I/O and text conversion are separate responsibilities.

### Better code structure

Separate the code into three parts:

Main / Driver Code

Creates objects and starts the operation

FileProcessor

Reads and writes file data

TextProcessor

Converts text into HTML

Example:

FileProcessor.java

```
public class FileProcessor {
    private String filePath;

    public FileProcessor(String filePath) {
        this.filePath = filePath;
    }

    public String readAllText() {
        // Read content from filePath
        return "";
    }

    public void writeToFile(String text) {
        // Write text to filePath
    }
}
```

TextProcessor.java

```
public class TextProcessor {
    private FileProcessor fileProcessor;

    public TextProcessor(FileProcessor fileProcessor) {
        this.fileProcessor = fileProcessor;
    }

    public void convertText() {
        String text = fileProcessor.readAllText();

        // Split text into paragraphs
        // Wrap each paragraph in HTML tags

        String html = "<html><body></body></html>";

        fileProcessor.writeToFile(html);
    }
}
```

Main.java

```
public class Main {
    public static void main(String[] args) {
        FileProcessor fileProcessor =
            new FileProcessor("input.txt");

        TextProcessor textProcessor =
            new TextProcessor(fileProcessor);

        textProcessor.convertText();
    }
}
```

The HTML conversion logic above is illustrative; the actual paragraph processing and HTML generation would need to be implemented.

### What did we achieve?

- `FileProcessor` handles file I/O.
- `TextProcessor` handles text conversion.
- `Main` acts as the driver code.

Now, anybody who wants to read or write a file can use `FileProcessor`, while anybody who wants to convert file content into HTML can use `TextProcessor`.

This is another application of SRP.

## 3. Composition: TextProcessor Has a FileProcessor

In the HTML converter example, we introduced a `FileProcessor` instance inside `TextProcessor`.

```
public class TextProcessor {
    private FileProcessor fileProcessor;

    public TextProcessor(FileProcessor fileProcessor) {
        this.fileProcessor = fileProcessor;
    }
}
```

This is an example of composition through constructor injection in the lecture's terminology.

The `TextProcessor` class has a `FileProcessor` as one of its properties and uses it to perform file I/O.

Instead of implementing file reading and writing directly inside `TextProcessor`, we pass a `FileProcessor` instance through the constructor.

### Why is this useful?

- Responsibilities are separated.
- Code becomes easier to maintain.
- The same `FileProcessor` can be reused by other classes.
- The text conversion logic doesn't need to know the internal details of file I/O.

The lecture also distinguishes composition from dependency injection (DI). Constructor injection is a way of supplying a dependency, while composition describes a relationship between objects. They are related concepts, but they are not the same thing.

### Why multiple files are useful

The lecture highlights several benefits:

- Better code organization: Each file has a clear responsibility.
- Less chance of merge conflicts: Multiple developers can work on different files.
- Easier searching: Smaller files can be easier for editors and IDEs to index and search.
- Better maintainability: Developers can locate the relevant functionality more easily.

The important point is that having too many files is not automatically a problem. The real problem is code that is difficult to maintain.

## 4. Open-Closed Principle (OCP)

The second SOLID principle discussed in the transcript is the Open-Closed Principle.

### Definition

> A class should be open for extensions, but closed for modifications.

This means we should try to structure code so that we can introduce new functionality without repeatedly modifying existing classes.

Let's understand this through the notification system example.

## 5. Problematic Code: Notification Centre

Suppose an application supports several types of notifications:

- SMS
- Email
- Push notification
- WhatsApp

We have a `NotificationCentre` class that sends notifications based on their type.

Conceptually, the code looks like this:

```
public class NotificationCentre {

    public void sendNotification(
        String type,
        String message
    ) {
        switch (type) {
            case "SMS":
                // Send SMS
                break;

            case "EMAIL":
                // Send email
                break;

            case "PUSH":
                // Send push notification
                break;

            case "WHATSAPP":
                // Send WhatsApp notification
                break;
        }
    }
}
```

### What is the problem?

Whenever we introduce a new notification type, we have to modify `NotificationCentre`.

For example:

- We introduce a new web push notification.
- We add another case to the switch statement.
- We introduce another notification type later.
- We modify the same class again.

Similarly, if we remove a notification type, we have to modify this class again.

This class is tightly coupled to the different types of notifications.

### Why is this an OCP violation?

The `NotificationCentre` class should be responsible for triggering notifications, not for knowing the implementation details of every notification type.

Every time we introduce or remove a notification type, the class needs modification.

This is the problem that OCP helps us address.

## 6. Why Separating Classes Is Not Enough

One possible improvement is to create separate classes:

- `SMSNotification`
- `EmailNotification`
- `PushNotification`

Each class can contain its own notification logic.

However, suppose `NotificationCentre` still uses a switch statement to decide which class to instantiate.

```
if (type.equals("SMS")) {
    // Create SMSNotification
} else if (type.equals("EMAIL")) {
    // Create EmailNotification
} else if (type.equals("PUSH")) {
    // Create PushNotification
}
```

Even though we have separated the notification classes, `NotificationCentre` still needs to change whenever a new notification type is introduced.

So, the main problem has not been completely resolved.

The important point: Simply creating multiple classes does not automatically solve an OCP violation. We must also consider how those classes interact.

## 7. Better Code: Apply the Open-Closed Principle

The improved approach is to introduce a common `Notification` interface.

Think about it:

- SMS is a notification.
- Email is a notification.
- Push is a notification.
- WhatsApp is a notification.

All these notification types share a common abstraction.

### Step 1: Create a Notification interface

```
public interface Notification {
    void sendMessage(String message);
}
```

This interface defines the common method that every notification implementation must provide.

### Step 2: Implement the interface

EmailNotification.java

```
public class EmailNotification
        implements Notification {

    @Override
    public void sendMessage(String message) {
        // Send email notification
    }
}
```

SMSNotification.java

```
public class SMSNotification
        implements Notification {

    @Override
    public void sendMessage(String message) {
        // Send SMS notification
    }
}
```

PushNotification.java

```
public class PushNotification
        implements Notification {

    @Override
    public void sendMessage(String message) {
        // Send push notification
    }
}
```

Each class implements the same interface but provides its own implementation of `sendMessage()`.

### Step 3: Improve NotificationCentre

Now `NotificationCentre` works with a list of `Notification` objects instead of checking individual notification types.

```
import java.util.List;

public class NotificationCentre {

    public void sendNotifications(
        List<Notification> notifications,
        String message
    ) {
        for (Notification notification : notifications) {
            notification.sendMessage(message);
        }
    }
}
```

Notice that there is no switch statement here.

The class doesn't need to know whether the notification is SMS, email, or push.

### Step 4: Create and pass notification objects

```
import java.util.List;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {

        List<Notification> notifications =
            Arrays.asList(
                new EmailNotification(),
                new SMSNotification(),
                new PushNotification()
            );

        NotificationCentre centre =
            new NotificationCentre();

        centre.sendNotifications(
            notifications,
            "Your order has been confirmed."
        );
    }
}
```

### What happens when we introduce a new notification?

Suppose we introduce `WhatsAppNotification`.

We create a new class that implements `Notification`.

```
public class WhatsAppNotification
        implements Notification {

    @Override
    public void sendMessage(String message) {
        // Send WhatsApp notification
    }
}
```

We can then add its instance to the list of notifications.

The `NotificationCentre` class does not need to change.

Before

Every new notification type requires modifications to the notification centre's switch statement.

After

Every new notification type implements the `Notification` interface, and the notification centre continues working with the common interface.

This is how the improved design follows OCP: the system can be extended with new notification implementations without modifying the existing notification centre.

## 8. Real-World Example: Scheduled Notifications

The lecture gives the example of applications that send notifications at a particular time.

Suppose a cron job runs every day at 10 a.m.

The process might be:

1. Fetch notification data from the database.
2. Create the required notification objects for each user.
3. Put those objects into a list.
4. Pass the list to `NotificationCentre`.
5. Trigger each notification.

The notification centre does not need to care about the type of notification. The corresponding notification object already contains the implementation for sending that notification.

This keeps the code organized and makes it easier to introduce new notification types in the future.

## 9. Another Example: Payment Methods

The same principle can be applied to a payment system.

Suppose the platform initially supports credit card payments.

Later, the platform introduces:

- Debit card payments
- UPI payments
- Net banking
- PayPal

We don't want every module in the application to change whenever a new payment method is introduced.

For example, an order confirmation class should not need to know whether the user paid through UPI, PayPal, net banking, or a debit card.

Eventually, the order still needs to be confirmed after the payment has been successfully processed.

The design should separate payment-specific logic from functionality that works with the completed order.

Key takeaway: When requirements change, avoid unnecessarily impacting every class in the application.

## 10. Depend on Abstractions Rather Than Concrete Classes

The lecture makes an important observation:

> It is generally a good idea to depend on abstractions rather than on concrete classes.

This is a useful design guideline that appears in several SOLID principles and design patterns.

### What are concrete classes?

Concrete classes are the actual classes containing the implementation logic.

Examples:

- `EmailNotification`
- `SMSNotification`
- `PushNotification`

### What are abstractions?

Abstractions represent the common contract or general idea shared by different implementations.

In our example:

```
public interface Notification {
    void sendMessage(String message);
}
```

The `Notification` interface is the abstraction.

### Comparison

| Depending on a concrete class                                      | Depending on an abstraction                                                      |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| The code knows about a specific implementation.                    | The code works with a common contract.                                           |
| Changing the implementation may require changes to dependent code. | New implementations can often be introduced without changing the dependent code. |
| Example: depending directly on `EmailNotification`.                | Example: depending on `Notification`.                                            |

For example, a business logic method can accept a `Notification` parameter instead of accepting an `EmailNotification` parameter.

```
public void notifyUser(Notification notification) {
    notification.sendMessage("Hello!");
}
```

Now the method can work with any notification implementation that follows the interface.

This makes the code more extendible.

## 11. What About User Input and Conditional Logic?

An important question raised in the lecture is: what if the user selects the notification or payment method?

Suppose the user enters `SMS`, `EMAIL`, or `PUSH`.

Some code still needs to interpret the user's input and decide which object to create.

For example:

```
String type = scanner.nextLine();

Notification notification;

if (type.equalsIgnoreCase("SMS")) {
    notification = new SMSNotification();
} else if (type.equalsIgnoreCase("EMAIL")) {
    notification = new EmailNotification();
} else {
    throw new IllegalArgumentException(
        "Unsupported notification type"
    );
}
```

This is not automatically a violation of OCP. The conditional logic may be necessary at the point where external input is translated into an object.

The important distinction is where the conditional logic lives and which class is responsible for object creation.

The lecture introduces the Factory Design Pattern as the next step in addressing this kind of object creation logic.

### Factory Design Pattern — Preview

The basic idea of the Factory Pattern is to have a dedicated class responsible for object creation.

Instead of scattering `new` object creation and selection logic throughout the application, we can centralize that responsibility in a factory.

For example:

- `NotificationFactory` creates the appropriate notification object.
- `NotificationCentre` sends notifications.
- Individual notification classes contain their own sending logic.

The `new` keyword itself is not a problem. The concern is how and where object creation is handled.

## 12. Interview Revision: SRP vs OCP

| Principle | Main question to ask                                                             | Example from the lecture                                                                                                      |
| --------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| SRP       | Does this class have multiple unrelated reasons to change?                       | Separating employee salary calculation and performance reporting from employee data management.                               |
| OCP       | Will adding a new feature require unnecessary modifications to existing classes? | Introducing a `Notification` interface so the notification centre doesn't need a new switch case for every notification type. |

## 13. Final Quick Revision

- SRP: A class should have one and only one reason to change.
- SRP example: Separate `UserManager` and `ActivityLogger`.
- SRP example: Separate `FileProcessor` and `TextProcessor`.
- Composition: `TextProcessor` has a `FileProcessor` instance and uses it for file I/O.
- Multiple files: Can improve organization, collaboration, and maintainability.
- OCP: A class should be open for extensions but closed for modifications.
- OCP example: Replace notification-type switch logic with a common `Notification` interface.
- Abstraction: Allows code to work with a common contract instead of depending on specific implementations.
- Factory Pattern: A dedicated object-creation mechanism that can help centralize selection and instantiation logic.

### Main lesson

The central idea of both principles is to make code easier to maintain when requirements change.

With SRP, we separate responsibilities that change for different reasons. With OCP, we structure the code so that introducing new functionality does not require unnecessary changes to existing classes.

The transcript ends with the instructor wrapping up the OCP discussion and introducing the Factory Design Pattern as a topic for further learning. It does not yet cover the remaining SOLID principles in detail.
