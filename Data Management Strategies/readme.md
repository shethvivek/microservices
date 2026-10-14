<div class="entry-content clear" itemprop="text">

<h2 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif; font-size: 18pt;"><strong>Data Management Strategies in Microservices</strong></span></h2>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In a microservices architecture, applications are broken down into many small, independent services. Each service focuses on a single business function, such as orders, payments, or inventory. While this independence allows teams to scale and innovate faster, it creates new challenges in <strong>managing data across services</strong>.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Unlike monolithic systems, where a single database ensures consistency, microservices typically use separate databases to maintain autonomy. This shift introduces complexities around <strong>distributed transactions, consistency, performance, and reliability</strong>. To address these challenges, teams need proven strategies and patterns that keep services independent while ensuring <strong>business truth and user trust</strong>.</span></p>
<h4 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Why Data Gets Hard in Microservices</strong></span></h4>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In a monolith, all components share a <strong>single database</strong>. Complex operations (such as updating multiple tables) are easily handled through <strong>ACID transactions</strong>. </span><span style="font-family: arial, helvetica, sans-serif;">In microservices, however, each service owns its own database. A single business action now spans <strong>multiple services, processes, and databases</strong>, making it harder to guarantee correctness. </span><span style="font-family: arial, helvetica, sans-serif;">This introduces three unavoidable realities:</span></p>
<hr>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Reality 1: Network Fallibility</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Microservices communicate over the network (HTTP, gRPC, or messaging). But networks are unreliable.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">A call might <strong>fail</strong> due to a crash or outage.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">A response might <strong>time out</strong> because the other service was slow.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Sometimes&nbsp;the same message may be&nbsp;<strong>delivered twice</strong>, resulting in&nbsp;duplicate actions.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> When an order is placed, the Order Service calls the Payment Service to capture payment. If the network fails just after the payment succeeded, the Order Service won’t know the result. It might retry, causing the payment to be charged twice unless the Payment Service handles duplicates.</span></p>
<hr>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Reality 2: Independent Lifecycles</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">One goal of microservices is that each service can be <strong>deployed, scaled, and upgraded independently</strong>.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">That means services may be at different <strong>versions</strong> at the same time.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">A service could be <strong>temporarily offline</strong> while being updated.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">There’s no “one big transaction” spanning all services, as in a monolith.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> Suppose the Inventory Service is being redeployed while an order is placed. The Order Service and Payment Service may succeed, but the Inventory update could be missed. Without special handling, the system might accept more orders than available stock.</span></p>
<hr>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Reality 3: Business Truth vs. System Truth</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Business Truth: </strong>Business truth is about the <strong>real-world outcome that must always stay correct</strong> according to business rules, no matter what happens in the system.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">It represents what the business <em>intends</em> or <em>promises</em>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Even if parts of the system fail or are inconsistent, the business truth must be restored.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h6>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In e-commerce: <em>“An order is valid only if payment is successful and stock is available.”</em></span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">If payment fails after the order is created, the business truth is: <strong>there is no valid order</strong>. The system must cancel the order and issue a refund.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Business truth ensures that <strong>the system&nbsp;</strong><span style="box-sizing: border-box; text-align: left;"><strong>accurately reflects reality</strong>, even when</span>&nbsp;temporary inconsistencies occur.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>User Trust: </strong>User trust is about the <strong>confidence customers have in the system</strong>, that it works reliably, protects their data, and won’t cheat them.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">It’s the foundation of customer satisfaction and loyalty.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Even small inconsistencies can break trust if not handled well.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">In banking, if a customer transfers ₹5,000 and sees money deducted but not credited instantly, they lose trust, even if the system fixes it later.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">In e-commerce, if a user gets charged but doesn’t receive an order confirmation, they lose trust in the platform.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">User trust depends on how well the system <strong>manages failures, communicates clearly, and keeps its promises to customers</strong>.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>In Short</strong></span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Business Truth</strong> = The <strong>real-world correctness</strong> of operations (e.g., “No order without payment”).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>User Trust</strong> = The <strong>confidence users have</strong> that the system is reliable, fair, and consistent.</span></li>
</ul>
<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Challenges of Data Management in Microservices</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Beyond those realities, microservices introduce <strong>technical challenges</strong> when each service manages its own database.</span></p><h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Distributed Transactions</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In microservices, a single business operation often requires <strong>multiple services to update their databases</strong>.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Each service can ensure its local transactions are safe.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">But across services, there’s no natural “all or nothing” guarantee.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example (E-commerce Order):</strong></span></h6>
<ol style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Order Service</strong> → Create order record.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Inventory Service</strong> → Reduce stock count.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Notification Service</strong> → Send Notification.</span></li>
</ol>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">If step 2 fails after step 1 succeeded, you now have an order, but the Product is not in stock. Rolling this back across services is complex.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Note:</strong> This is why we need strategies like the <strong>Saga Pattern</strong> (chaining local transactions with compensating actions).</span></p>

<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Data Consistency</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Because services update at <strong>different times</strong>, data across the system may not always match.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Some services may reflect the <strong>latest state</strong>, while others still show <strong>stale data</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Microservices often rely on <strong>eventual consistency</strong>: the system will be correct, but not instantly.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">After an order is placed, the Order Service immediately shows <strong>Order Confirmed</strong><em>.</em></span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The Inventory Service may still show the old stock count for a few seconds due to message delays.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">During that gap, another customer could also order the last item, creating a temporary inconsistency.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Note:</strong> The key is deciding where <strong>strong consistency</strong> (immediate correctness) is required (e.g., banking transactions) vs. where <strong>eventual consistency</strong> is acceptable (e.g., updating a profile picture).</span></p>

<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Performance Overhead</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In a monolith, you could run a <strong>JOIN query</strong> across multiple tables in one database. In microservices, data is split across multiple databases.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Getting related information often requires calling <strong>several services</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">This increases <strong>latency</strong> (delay) and <strong>network overhead</strong>.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h6>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">To display a user’s order history page:</span></p><div class="code-block code-block-5" style="margin: 8px 0; clear: both;">
</div><ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Fetch user details from the <strong>User Service</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Fetch orders from the <strong>Order Service</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Fetch product details from the <strong>Product Service</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Fetch payment status from the <strong>Payment Service</strong>.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Instead of&nbsp;<span style="box-sizing: border-box; text-align: left;">a single SQL query, this may involve&nbsp;<strong>4–5 cross-service calls</strong>, which are</span>&nbsp;slower and more complex to maintain.</span></p>

<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Anti-Pattern: One Shared Database for All Services</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">At first, many beginners try to make life easier by saying<span style="box-sizing: border-box; text-align: left;">,&nbsp;<strong>“Why don’t we just let all microservices share the same database?</strong>“</span>&nbsp;It sounds simple because:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">There’s only one database to manage.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Data joins are easy, just like in a monolith.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Developers don’t need to think about distributed consistency.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">But in reality, this is an <strong>anti-pattern</strong> that violates core microservice principles of <strong>independence and autonomy</strong>.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Why It’s Problematic</strong></span></h5>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Tight Coupling</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Microservices are supposed to be <strong>loosely coupled</strong>; each service can evolve independently.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">With a shared database, a <strong>schema change</strong> in one service could accidentally break another service.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> If the <strong>Product Service</strong> adds a new column in the Product table, the <strong>Order Service</strong> might fail if it wasn’t expecting that column. Suddenly, both services must be updated together, which removes their independence.</span></p>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Scaling Issues</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Microservices allow each service to scale separately depending on demand.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">With&nbsp;<strong>a single shared database, all services compete for the&nbsp;<strong>same </strong>resources</strong> (CPU, memory, I/O).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">One heavy service can slow down the entire system.cg</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> During a holiday sale, the <strong>Inventory Service</strong> might need to process millions of stock updates. If it shares the same database as the <strong>Billing Service</strong>, payment processing could slow down because of database overload.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Security Risks</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">If one service is compromised, the attacker potentially gains access to <strong>all tables</strong>, not just the intended ones.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">One “bad actor” service compromises the integrity of the whole system.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> If a bug in the Notification Service accidentally deletes records, it could wipe out Order or Payment data, resulting in a catastrophic outcome.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Deployment Bottlenecks</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">In microservices, services should deploy independently.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">With a shared database, even a small schema change in one service might require retesting and redeploying other services that share it.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">This kills the whole purpose of independent deployments in microservices.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> If the <strong>Inventory Team</strong> wants to add a new stock-tracking field, the <strong>Billing Team</strong> must update their queries as well, so both teams have to schedule a joint release. This kills the agility of microservices.</span></p><div class="code-block code-block-6" style="margin: 8px 0; clear: both;">
</div><h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Ownership Confusion</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">In microservices, each team should <strong>own its data</strong> and make decisions about schema, performance, and scaling.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">With a shared database, no one knows who really controls what. Multiple teams touching the same tables often lead to conflicts, delays, and finger-pointing when something goes wrong.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> The Inventory team and Billing team both depend on the Product table. Who decides when to add or remove a column? Who optimizes indexes? This shared responsibility often leads to conflicts and mistakes.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A shared database may feel easy, but it creates a <strong>tightly coupled monolith disguised as microservices.</strong> You lose independence, scalability, and agility, which are the very reasons for adopting microservices in the first place.</span></p>

<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Database Per Microservice Approach</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">The industry-recommended approach is the <b>“Database per Microservice” </b>model. Here:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Each microservice <strong>owns its own database</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">No service can access another’s tables directly.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Services interact only through <strong>APIs, events, or contracts</strong>.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This respects the principle that <em>“a microservice is responsible for both its logic and its data.”</em></span></p>
<h4 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Benefits of This Approach</strong></span></h4>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Autonomy</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Each team can evolve its database <strong>independently</strong>, without breaking others.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Database schema changes affect only that service.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> The <strong>Product Team</strong> can add new attributes, such as “Color” or “Material,” to their database without asking the <strong>Order Team</strong> for permission.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Independent Scaling</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Each database scales based on the load of its service.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">A high-traffic service can get a bigger or more optimized database without affecting others.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Order Service</strong> database may require heavy transaction support and indexing.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Notification Service</strong> database may just need lightweight storage for templates.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Each can scale independently, avoiding overloading a single shared system.</span></li>
</ul>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Technology Flexibility</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Each microservice can use the <strong>best database technology</strong> for its needs, rather than being forced into a single type.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Examples:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Order Service</strong> → SQL Server or PostgreSQL (great for relational, transactional data).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Product Service</strong> → MongoDB (perfect for flexible product catalogs).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Notification Service</strong> → Redis (ideal for fast key-value lookups and caching).</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This “polyglot persistence” gives every service the freedom to choose what works best.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Clear Ownership</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The team owning a service also owns its data.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">They decide on schema design, indexing, performance optimization, and security.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> The <strong>Order Team</strong> manages only the Orders database. If something breaks, there’s no confusion about who is responsible.</span></p><div class="code-block code-block-7" style="margin: 8px 0; clear: both;">
</div><h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Real-Time Example of Database per Service</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Order Service</strong>: Uses a <strong>relational database</strong> (like SQL Server) to ensure strict transaction guarantees for customer orders.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Product Service</strong>: Uses a <strong>NoSQL database</strong> (like MongoDB) to handle flexible product attributes and support fast search.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Notification Service</strong>: Uses <strong>Redis</strong> to retrieve templates and send SMS/email notifications quickly.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Each service uses a different database type, optimized for its workload. This is impossible with a single shared database.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">The <strong>Database per Microservice</strong> pattern ensures independence, scalability, and agility. It allows each service to choose its own technology, scale on demand, and evolve without affecting others. This is the foundation for building truly autonomous microservices.</span></p>
<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Data Consistency Models: Strong vs. Eventual</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">When data is distributed across multiple microservices, <strong>consistency becomes one of the biggest challenges</strong>. Unlike monolithic systems, where a single ACID transaction ensures everything is updated together, microservices must choose between <strong>accuracy at all times</strong>, <strong>scalability, and flexibility</strong>. The two most common models are <strong>Strong Consistency</strong> and <strong>Eventual Consistency</strong>.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Strong Consistency</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In this model, <strong>all services see the same data immediately</strong> after a transaction. That means, in <strong>strong consistency</strong>, once a transaction is committed, all services see the <strong>same data immediately</strong>.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">This usually requires <strong>distributed transactions</strong> (e.g., Two-Phase Commit) where multiple databases agree before the operation is finalized.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">It mimics the behaviour of a monolith, but across distributed services.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Guarantees <strong>correctness at all times</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Prevents anomalies (e.g., no chance of seeing old balances or duplicate records).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">No risk of one service showing stale or outdated data.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Very <strong>complex</strong> to implement across microservices.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Increases <strong>latency</strong> because all services must wait for each other.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Introduces <strong>bottlenecks</strong> when scaling.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Real-World Example</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Banking Transactions.</strong> If you transfer ₹5000 from Account A to Account B, both accounts must reflect the correct balances <strong>instantly</strong>. You cannot risk one account being debited while the other is not credited. Even a small delay could cause serious errors (like withdrawing money that isn’t really available).</span></li>
</ul>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Eventual Consistency</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In this model, <strong>services may temporarily show different data</strong>, but they “catch up” over time to reach the same state and become consistent.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Updates are propagated asynchronously (e.g., via events or messages). It may take seconds or minutes for all services to be in sync.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The system prioritizes <strong>performance and availability</strong> over instant accuracy.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Works well for <strong>most business cases</strong> where a small delay is acceptable.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Supports <strong>high scalability</strong> and <strong>better fault tolerance</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Works well in high-traffic systems where strict accuracy is not critical at all times.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Users may see <strong>outdated data</strong> for a short time.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Developers must design carefully to handle these short-lived inconsistencies.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Real-World Example</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>E-commerce shipping address</strong>: If you update your address, it’s okay if the <strong>Order Service</strong> reflects the change a few seconds later. A short lag doesn’t harm the core business.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Social media likes</strong>: When you “like” a post, you may not see the updated like count immediately, but it will eventually appear.</span></li>
</ul>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Rule of Thumb</strong></span></h5>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Use <strong>Strong Consistency</strong> only where correctness is non-negotiable (e.g., financial transactions, medical records).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Use <strong>Eventual Consistency</strong> for most business cases where speed, scalability, and availability matter more, and where slight delays are acceptable (e.g., profile and catalog updates).</span></li>
</ul>
<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Patterns for Data Consistency</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">To handle consistency in microservices, several well-known patterns exist. Now let’s look at the <strong>main patterns</strong> used to keep data consistent in microservices.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Saga Pattern</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A <strong>Saga</strong> is the most popular way to manage distributed transactions in microservices. Instead of one big global transaction, a <strong>Saga</strong> breaks a large business transaction into a sequence of <strong>smaller local transactions</strong>, each executed by a different service, with <strong>compensating actions</strong> to undo work if something fails.</span></p><div class="code-block code-block-8" style="margin: 8px 0; clear: both;">
</div><ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">After completing its local transaction, a service <strong>publishes an event</strong> to trigger the next step.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">If something fails, the system triggers <strong>compensating actions</strong> to undo previous work.</span></li>
</ul>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Styles of Saga</strong></span></h5>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Choreography (Decentralized)</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Services communicate via <strong>events</strong>. Each service listens for an event, performs its own local transaction, and publishes the next event.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros:</strong> Simple to start, no central controller.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons:</strong> Harder to manage as workflows grow; flow is scattered across services. Debugging becomes difficult.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Order Service → publishes <strong>OrderCreated</strong></span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Inventory Service → reserves stock, publishes <strong>StockReserved</strong></span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Payment Service → processes payment, publishes <strong>PaymentCompleted</strong></span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Analogy</strong>: Think of a wedding dance where each dancer knows their steps and reacts to others. Works well for small groups, but becomes chaotic (disorganized, confusing, and lacking order) as more people join.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Orchestration (Centralized)</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A <strong>Saga Orchestrator</strong> (central <strong>orchestrator service</strong>) manages the workflow and tells each service what to do next. It keeps track of the flow, retries, and compensations.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros:</strong> Clear visibility, easier debugging, retries, and timeouts managed in one place.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons:</strong> Adds a central coordinating component (orchestrator).</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Analogy</strong>: Think of an orchestra with a conductor who directs musicians in sequence. More organized and predictable.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example: Order Saga (Orchestrated)</strong></span></h5>
<ol style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>CreateOrder (Pending)</strong> → Order Service saves order.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>ReserveInventory</strong> → Inventory Service reserves stock.</span>
<ul>
<li style="list-style-type: none;">
<ul>
<li><span style="font-family: arial, helvetica, sans-serif;">If fail → <strong>CancelOrder</strong>.</span></li>
</ul>
</li>
</ul>
</li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>ProcessPayment</strong> → Payment Service charges the customer.</span>
<ul>
<li style="list-style-type: none;">
<ul>
<li><span style="font-family: arial, helvetica, sans-serif;">If fail → <strong>ReleaseInventory, CancelOrder</strong>.</span></li>
</ul>
</li>
</ul>
</li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>ConfirmOrder</strong> → Finalize the order.</span></li>
</ol>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">If step 3 fails, compensating actions <strong>restore balance</strong>: release stock and cancel the order. This ensures the business truth: <strong>No Order Without Successful Payment.</strong></span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">So,<strong> think of Saga as “Business-Level Rollback”</strong> for microservices. Instead of ACID rollback across a single DB, each service undoes its own work when something fails.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Two-Phase Commit (2PC)</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A <strong>classic distributed transaction protocol</strong>. It is a traditional way to ensure <strong>strong consistency</strong> across distributed systems. A central coordinator asks all participants if they can commit. If everyone agrees, it commits. Otherwise, it rolls back.</span></p><div class="code-block code-block-9" style="margin: 8px 0; clear: both;">
</div><ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Phase 1: Prepare</strong> → All participants confirm they’re ready.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Phase 2: Commit</strong> → If all agree, the transaction is committed; otherwise, it’s rolled back.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Provides <strong>strong consistency</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Guarantees <strong>atomicity</strong> across multiple databases.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Very slow in distributed environments.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Locks resources while waiting for all participants.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">It can create bottlenecks and single points of failure.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Rarely used in microservices because availability is usually more important.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="box-sizing: border-box; text-align: left; font-family: arial, helvetica, sans-serif;"><strong>Banking systems</strong> still use 2PC when transferring money across different banks/databases because correctness cannot be compromised, but it is avoided in large-scale microservices because it doesn’t scale well.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Rule of Thumb:</strong> Avoid 2PC in microservices unless absolutely necessary. Prefer <strong>Sagas + compensations</strong>.</span></p>

<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>CQRS (Command Query Responsibility Segregation)</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">CQRS separates <strong>Writes</strong> (Commands that change state) from <strong>Reads</strong> (Queries that return data). In this case:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The <strong>Command side</strong> updates the system of record (primary database).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The <strong>Query side</strong> uses a different database or projection optimized for fast reads.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Read models can be designed for speed and scalability.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Write models can focus on correctness and validation.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">More complex architecture.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Requires syncing between write and read stores (often eventual consistency).</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong> Ticket booking system</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Command Side:</strong> BookSeat service updates the core booking DB.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Query Side:</strong> A separate read database serves “Available Seats” to users very quickly.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Even if updates take a second to sync, users can still browse quickly while ensuring seat reservations are reliable.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This avoids overloading the main booking database with heavy queries while ensuring users see near real-time results.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Query Patterns Across Services:</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In microservices, each service owns its <strong>own database</strong>. This ensures autonomy and independence but introduces a challenge:</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Sometimes you need data that is <strong>spread across multiple services</strong>. For example, to build a <strong>Customer Profile Page</strong>, you may need:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">User Details (from <strong>User Service</strong>)</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Order History (from <strong>Order Service</strong>)</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Last Payment Status (from <strong>Payment Service</strong>)</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Since there is no single “database” to query across all services, we need <strong>query patterns</strong> to combine data in valuable ways. Two popular strategies are <strong>API Composition</strong> and <strong>CQRS Read Models (Materialized Views).</strong></span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>API Composition</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In this approach, a <strong>client (or&nbsp;</strong><span style="box-sizing: border-box; text-align: left;"><strong>an API Gateway)</strong>&nbsp;makes calls to multiple services and&nbsp;<strong>combines their responses</strong> into a single</span>&nbsp;result.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The client or gateway acts like a <strong>data aggregator</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Each service still owns its data, and no extra database is needed.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Aggregation occurs at <strong>runtime</strong> when the request is made.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The responses are merged and presented as a single view.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Simple</strong> to implement. No extra data storage or sync logic is required.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Data is always <strong>fresh</strong> because it always fetches the latest data directly from services.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Latency Increases</strong>: Can cause <strong>high latency</strong> (longer response times) when many services are involved.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Failure risk</strong>: If one service is down or slow, the whole composition may fail or return incomplete results, or the entire response may suffer.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Complex logic at client/gateway</strong>: Someone needs to handle retries, errors, and combine data.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">More network traffic due to multiple calls.</span></li>
</ul>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Real-World Example:</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A <strong>Customer Profile Page</strong> in an e-commerce app:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Call <strong>User Service</strong> for customer details (name, email, phone).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Call <strong>Order Service</strong> for past orders.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Call the <strong>Payment Service</strong> for the latest payment status.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The API Gateway merges all of these into a single response for the UI.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This approach is fine when you need a <strong>few services</strong> and can tolerate multiple calls.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Analogy:</strong> Think of a <strong>travel booking website</strong>. To display your trip details, it fetches flights, hotel bookings, and car rentals from various services and combines them into your itinerary.</span></p>

<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>CQRS Read Models (Materialized Views)</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Sometimes, frequent queries need data from many services; calling all of them every time would be too <strong>slow</strong>. In this case, we use <strong>CQRS Read Models</strong> or <strong>Materialized Views</strong>. Here, instead of calling multiple services every time, we maintain a <strong>separate read-optimized database</strong> (also called a <strong>materialized view</strong>) that <strong>pre-aggregates data</strong> from multiple services.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">A <strong>separate read database</strong> is maintained.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">This read database stores a <strong>pre-combined view of data</strong> from multiple services.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Services publish <strong>events</strong> (like “OrderPlaced” or “PaymentCompleted”) that update the read model.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In this scenario:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The <strong>write models</strong> (original service databases) remain independent.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The <strong>read model</strong> is optimized for queries, not for updates.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Pros:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Very fast queries</strong>, all the needed data is already in one place.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Great for dashboards, reports, and analytics.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Reduces repeated service calls.</span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Cons:</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Complexity</strong>, keeping the read model in sync with changes from multiple services, is tricky.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Data may be slightly <strong>out of date</strong> (eventual consistency).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Data is stored in multiple places (services + read model).</span></li>
</ul>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example:</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A <strong>Reporting Service</strong> in e-commerce:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Aggregates data from Orders, Payments, and Deliveries into a single <strong>read model</strong>.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">When you run a sales report, the system queries only the <strong>reporting database</strong> instead of calling all services separately.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">This allows instant queries, even with millions of records.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This approach is best when you need <strong>frequent</strong><span style="box-sizing: border-box; text-align: left;"><strong>, complex queries</strong> (e.g., analytics and&nbsp;</span>dashboards) and performance is critical.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Analogy:</strong> Think of a <strong>news aggregator app</strong>. Instead of checking dozens of websites every time you open it, the app <strong>already stores and organizes articles</strong> in one place for fast browsing.</span></p>
<hr>
<h3 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Handling Distributed Transactions and Compensating Actions</strong></span></h3>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In a <strong>monolithic system</strong>, multiple operations (such as creating an order, updating stock, and charging a payment) can be wrapped in a <strong>single ACID transaction</strong>. This ensures an all-or-nothing outcome: either everything succeeds, or everything rolls back automatically.</span></p>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In <strong>microservices</strong>, each service owns its own database, and operations span across <strong>different services and networks</strong>. This makes global ACID transactions <strong>impractical</strong>, so instead, we rely on <strong>distributed workflows</strong> and <strong>compensating actions</strong> to maintain consistency.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Compensating Transactions</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">A <strong>compensating transaction</strong> is an action that <strong>undoes</strong> the effect of a previous step if something goes wrong later in the process.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">Think of it as a “manual rollback” at the business level.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Instead of undoing everything in a single significant DB transaction, each service <strong>reverses its own work</strong> when needed.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example: Airline Booking System</strong></span></p>
<ol style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Seat Reservation Service</strong> reserves a seat.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Payment Service</strong> charges the customer.</span></li>
</ol>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">If payment fails:</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The system doesn’t just leave the seat blocked.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">A <strong>compensating action</strong> in the Seat Reservation Service restores the seat to availability.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This way, the system still aligns with the user’s intention: no payment → no booking.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Idempotency</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">In distributed systems, failures and retries are normal. Without <strong>idempotency</strong>, retries can cause data corruption.</span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Idempotent operation</strong>: An operation that can be repeated multiple times without changing the final result.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Example: Cancel Order Request</strong></span></p>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">If the <strong>CancelOrder</strong> request is sent once, the order is canceled, and the user gets a refund.</span></li>
<li><span style="box-sizing: border-box; text-align: left;"><span style="color: #000000;">If the same request is sent again (due to a retry or a duplicate message), the system should&nbsp;</span><strong><span style="color: #000000;">not issue a second refund</span></strong><span style="color: #000000;">.</span></span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Idempotency ensures that repeated operations lead to the same outcome.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">This prevents overcharging, over-refunding, or double deductions.</span></p>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Real-Time Example: Order and Payment Workflow</strong></span></h5>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Let’s look at an e-commerce scenario where multiple services participate in a single workflow.</span></p>
<ol style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>User places an order</strong></span>
<ul>
<li style="list-style-type: none;">
<ul>
<li><span style="font-family: arial, helvetica, sans-serif;">The <strong>Order Service</strong> creates an entry in <em>Pending</em> status.</span></li>
</ul>
</li>
</ul>
</li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Payment Service charges the user</strong></span>
<ul>
<li style="list-style-type: none;">
<ul>
<li><span style="font-family: arial, helvetica, sans-serif;">If successful → move to the next step.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">If it fails → compensating action <strong>cancels the order</strong>.</span></li>
</ul>
</li>
</ul>
</li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Inventory Service deducts stock</strong></span>
<ul>
<li style="list-style-type: none;">
<ul>
<li><span style="font-family: arial, helvetica, sans-serif;">If successful → continue.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">If stock is unavailable → compensating action <strong>refunds the payment</strong> and <strong>cancels the order</strong>.</span></li>
</ul>
</li>
</ul>
</li>
<li><span style="font-family: arial, helvetica, sans-serif;"><strong>Notification Service sends confirmation</strong></span>
<ul>
<li style="list-style-type: none;">
<ul>
<li><span style="font-family: arial, helvetica, sans-serif;">User receives an email or SMS confirming their order.</span></li>
</ul>
</li>
</ul>
</li>
</ol>
<h5 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Failure Scenarios &amp; Compensations</strong></span></h5>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Payment Fails</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The Order Service cancels the pending order.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Any reserved stock is restored (compensating action).</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The user gets a notification: <em>“Your order could not be processed.”</em></span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Inventory Out of Stock</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The Inventory Service reports failure.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The Saga triggers a rollback: Payment Service refunds the money, and the Order Service cancels the order.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">The customer sees: <em>“Item out of stock, your payment has been refunded.”</em></span></li>
</ul>
<h6 style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;"><strong>Notification Fails</strong></span></h6>
<ul style="text-align: justify;">
<li><span style="font-family: arial, helvetica, sans-serif;">The order, payment, and inventory updates are already successful.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Instead of rolling back, the Notification Service retries sending.</span></li>
<li><span style="font-family: arial, helvetica, sans-serif;">Because of <strong>idempotency</strong>, multiple retry attempts don’t send duplicate emails.</span></li>
</ul>
<p style="text-align: justify;"><span style="font-family: arial, helvetica, sans-serif;">Managing data in microservices is not about forcing monolithic database practices into a distributed system, but about embracing new approaches. By following patterns like Saga, CQRS, and compensating transactions, and by choosing the right consistency model, teams can balance accuracy with scalability. The key is to design for independence while ensuring a consistent user experience. With the right strategies, microservices can deliver both developer flexibility and business reliability.</span></p>
