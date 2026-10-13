
<h2>What Is Domain-Driven Design (DDD) ?</h2>

Domain-Driven Design (DDD) Architecture is a method of structuring software so that the domain, meaning the business problem being solved, is placed at the center of the design. In DDD, we first build a conceptual model of our business, capturing all the essential objects, rules, and behaviours.

This model consists of several building blocks:

* <b>Entities:</b> Domain objects that have a distinct identity, remaining the same even as their attributes change.
* <b>Value Objects:</b> Immutable objects defined solely by their properties and values, not by identity.
* <b>Aggregates:</b> A cluster of related entities and value objects treated as a single unit for data changes and consistency.
* <b>Domain Events:</b> Records of something important that has happened within the domain, used to trigger side effects or communication.
* <b>Domain Services:</b> Operations or business logic that don’t naturally fit within entities or value objects but are essential to the domain.

For a better understanding, please refer to the following image.

By focusing on the domain, DDD helps ensure that your code accurately reflects the way your business works, making it easier to understand, maintain, and evolve.

<h3>Layman’s Examples to Understand Domain-Driven Design (DDD)</h3>

Let’s understand the need for Domain-Driven Design with a few examples that are easy to understand.

<h4>Example 1: Online Store</h4>

Here:

* <b>Domain Expert (Bookstore Owner):</b> Talks about managing orders, keeping track of inventory, customer information, and handling payments.
* <b>Developer:</b> Creates objects like Order, Inventory, Customer, and Payment in code, each encapsulating real-world business rules, for instance, reducing stock when an order is placed or validating payment details.
* <b>Common Language:</b> Both domain experts and developers use the same terms and concepts like “order,” “inventory,” and “payment” to avoid miscommunication and ensure clarity.


<h4>Example 2: Bank Account Management</h4>

Here:

* <b>Domain Expert (Bank Manager):</b> Discusses accounts, transactions, overdraft limits, and how interest is calculated.
* <b>Developer:</b> Models these as Account, Transaction, and InterestCalculator objects. All the rules (such as not allowing overdrafts beyond ₹10,000 or calculating monthly interest) reside within these objects, not scattered throughout controllers or database scripts.
* <b>Common Language:</b> Everyone calls it an “Account” and a “Transaction,” so requirements and code are always in sync, and mistakes are minimized.

<h3>Domain-Driven Design (DDD) Architecture in ASP.NE Core</h3>

Domain-Driven Design (DDD) structures an application into distinct layers. This separation of concerns leads to cleaner, more maintainable code and helps ensure that business rules remain at the heart of the system. For a better understanding, please refer to the following diagram:

The following is a detailed explanation of each layer and its core components:

<h3>Domain Layer (Entities, Repository Interfaces, Domain Services)</h3>

The Domain Layer is the core of DDD, containing pure business logic and the domain model. It models real-world business concepts, rules, and behaviors, independent of technical concerns. The following are the Key Components:

* <b>Entities:</b> Business objects with a unique identity that persists over time (identity never changes). They contain state and business behavior (e.g., Product, Order, Customer).
* <b>Repository Interfaces:</b> Define contracts for accessing and storing entities. They define methods for operations like finding orders or saving products, with implementations provided in the Infrastructure Layer.
* <b>Domain Services:</b> Services that encapsulate domain logic that doesn’t naturally fit within a single entity (e.g., complex business calculations or operations spanning multiple entities).

<b>Example:</b> The Order entity enforces rules like “an order must have at least one item.” The IOrderRepository interface defines how orders are persisted; however, the actual database code is located elsewhere.

<h3>Infrastructure Layer (EF Core DbContext, Repository Implementations, External Integrations)</h3>
The Infrastructure Layer contains all technical implementation details related to external systems and frameworks (databases, message brokers, files, third-party services). It keeps infrastructure concerns isolated so the Domain Layer remains clean and independent. The following are the Key Components:

* <b>EF Core DbContext:</b> Acts as a bridge between domain models and the database. Manages querying, saving, and updating entities (e.g., Product, Order) within the database.
* <b>Repository Implementations:</b> Actual code for storing and retrieving entities, as defined by the repository interfaces in the Domain Layer (e.g., OrderRepository using EF Core).
* <b>External Integrations:</b> Services communicating outside the core system, such as email services, payment gateways, messaging queues, or third-party APIs. Placed here to avoid polluting the domain and application logic.

<b>Example:</b> OrderRepository implements IOrderRepository to interact with SQL Server using EF Core. The EmailService integration sends confirmation emails when an order is placed.

<h3>Presentation Layer (Controllers)</h3>

The Presentation Layer is responsible for handling all incoming requests from users or external systems and managing how the application interacts with its clients. It serves as the interface between the outside world and the internal system. The following are the Key Components of the Presentation Layer:

<h4>Controllers:</h4>

* Act as the entry points for external clients such as browsers, mobile apps, or other services.
* Handle receiving HTTP requests, performing input validation (checking format and correctness), and returning appropriate HTTP responses (e.g., JSON data, status codes).
* Designed to be thin and lightweight, delegating business workflows and logic to the Application Layer.

<b>Example:</b> An OrderController receives a POST request, validates it, and calls the Application Layer’s OrderService to process the order. The result is returned as a response.

<h3>Application Layer (DTOs and Services)</h3>

The Application Layer acts as a coordinator that connects the Presentation Layer with the Domain Layer. It manages the flow of data, business workflows, and enforces application-specific policies. This layer does not contain core business rules; those reside in the Domain Layer. The following are the Key Components:

<h4>DTOs (Data Transfer Objects):</h4>

* Simple objects are used to carry data between the Presentation Layer and the Application Layer.
* They exclude business behavior and contain only the necessary data for interaction.
* DTOs help decouple the domain model from external data formats, improving security and flexibility.
* Examples include OrderCreateDTO for incoming order data and OrderDTO for responses.

<h4>Services (Application Services):</h4>

* Represent the system’s use cases and business workflows, such as placing orders, processing payments, or managing product lifecycles.
* Handle tasks like transaction management, authorization, and calling multiple domain entities or services to complete a use case.
* Delegate core domain logic to domain entities and domain services to maintain separation of concerns.

<b>Example:</b> The OrderService method PlaceOrderAsync() takes validated input (DTOs) from the Presentation Layer, calls domain methods to create and validate an order, and manages the overall order processing flow.

<h3>Why Clear Layering or Benefits of DDD?</h3>

* <b>Maintainability:</b> Each layer focuses on a single concern, making the code easier to understand and maintain.
* <b>Testability:</b> Layers can be tested independently (e.g., domain logic without database).
* <b>Flexibility:</b> You can replace UI or infrastructure details without affecting the core business logic.
* <b>Collaboration:</b> Domain experts and developers can focus on the domain model without worrying about technical details.
