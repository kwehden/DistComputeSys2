# Source Material

## Pillar 1: Pike's 5 Rules of Programming

Source: https://www.cs.unc.edu/~stotts/COMP590-059-f24/robsrules.html
Author: Rob Pike (Notes on Programming in C, 1989)

### The Rules

**Rule 1:** Bottlenecks occur in surprising places, so don't try to second guess and put in a speed hack until you've proven the location.

**Rule 2:** Measure. Don't tune for speed until you've measured, and even then don't unless one part of the code overwhelms the rest.

**Rule 3:** Fancy algorithms are slow when n is small, and n is usually small. Fancy algorithms have big constants. Until you know that n is frequently going to be big, don't get fancy. (Even if n does get big, use Rule 2 first.)

**Rule 4:** Fancy algorithms are buggier than simple ones, and they're much harder to implement. Use simple algorithms as well as simple data structures.

**Rule 5:** Data dominates. If you've chosen the right data structures and organized things well, the algorithms will almost always be self-evident. Data structures, not algorithms, are central to programming.

### Related Maxims
- Rules 1 and 2 restate Tony Hoare's: "Premature optimization is the root of all evil."
- Ken Thompson rephrased Rules 3 and 4 as: "When in doubt, use brute force."
- Rule 5 was articulated by Fred Brooks and is often shortened to: "Write stupid code that uses smart objects."

### Application in This Overlay
Pike's Rules serve as a **brake** on complexity. They prevent both transactional bias (Rule 5: get the data structures right) AND distributed-enthusiasm bias (Rules 3-4: don't get fancy until you've proven n is big).

---

## Pillar 2: The Reactive Manifesto (v2.0)

Source: https://www.reactivemanifesto.org/
Published: September 16, 2014
Authors: Jonas Bonér, Dave Farley, Roland Kuhn, Martin Thompson

### The Four Traits

**Responsive:** The system responds in a timely manner. Responsiveness is the cornerstone of usability. The system provides rapid and consistent response times, establishing reliable upper bounds.

**Resilient:** The system stays responsive in the face of failure. Resilience is achieved through replication, containment, isolation, and delegation. Failures are contained within each component, isolating components from each other. Recovery is delegated to external components.

**Elastic:** The system stays responsive under varying workload. It reacts to changes in input rate by increasing or decreasing resources. The design has no contention points or central bottlenecks.

**Message Driven:** Reactive Systems rely on asynchronous message-passing to establish boundaries between components that ensure loose coupling, isolation, and location transparency. Message-passing enables load management, elasticity, and flow control through back-pressure.

### Application in This Overlay
Reactive properties define **what justified boundaries must do** — but only at boundaries. Within a single small bounded context, these properties are optional overhead.

---

## Pillar 3: Domain-Driven Design (DDD)

Sources:
- Eric Evans, "Domain-Driven Design: Tackling Complexity in the Heart of Software" (2003)
- Martin Fowler, "Bounded Context" (bliki): https://martinfowler.com/bliki/BoundedContext.html
- Martin Fowler, "DDD Aggregate" (bliki): https://martinfowler.com/bliki/DDD_Aggregate.html
- Vaughn Vernon, "Implementing Domain-Driven Design" (2013)

### Key Concepts

**Bounded Context:** A boundary within which a domain model is defined and applicable. Each bounded context has its own ubiquitous language and maintains internal consistency. Different contexts may model the same concept differently.

**Ubiquitous Language:** The shared vocabulary between developers and domain experts within a bounded context. When the language changes, you've crossed a context boundary.

**Aggregate:** A cluster of domain objects that can be treated as a single unit. The aggregate defines a consistency boundary — transactions should not cross aggregate boundaries. External references go only to the aggregate root.

**Context Map:** The visual and structural representation of how bounded contexts relate to each other. Relationship patterns include:
- Shared Kernel: two contexts share a subset of the model
- Customer/Supplier: upstream supplies, downstream consumes
- Conformist: downstream conforms to upstream's model
- Anti-Corruption Layer: downstream translates from upstream's alien model
- Open Host Service: upstream provides a protocol for integration
- Published Language: shared language for integration (e.g., event schemas)
- Separate Ways: contexts don't integrate

**Anti-Corruption Layer (ACL):** A translation layer that prevents an external model from corrupting the internal domain model. The ACL belongs to the consuming context.

### Application in This Overlay
DDD provides the **structural rationale** for boundaries. It answers "where should boundaries go?" based on domain model coherence, not infrastructure topology. Without DDD, distributed-systems guidance draws boundaries where infrastructure naturally splits — which often produces the wrong topology.

---

## How the Three Compose

| Question | DDD answers | Pike answers | Reactive answers |
|----------|-------------|--------------|------------------|
| Where are the boundaries? | Where the ubiquitous language changes | — | — |
| How many boundaries? | As many as the domain demands | As few as complexity allows | — |
| What's the consistency unit? | The aggregate | The simplest correct data structure | — |
| Is this boundary justified? | Is there a language/model difference? | Is there measured scale need? | — |
| What does the boundary do? | — | — | Message-driven, resilient, elastic, responsive |
| What integration pattern? | Context map relationship type | — | — |
| What data structure at boundaries? | — | Whatever makes algorithms self-evident | Events (messages) |
