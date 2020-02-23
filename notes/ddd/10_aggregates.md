# Aggregates

Is an Aggregate just a way to cluster a graph of closely related objects under a common parent? If so, is there some practical limit to the number of objects that should be allowed to reside in the graph? Can the associations be navigated deeply, modifying various objects along the way? What is this concept of invariants and a consistency boundary all about?

Answer to the last question that greatly influences the answers to the others

## Design Attempts
### First attempt: large cluster Aggregate
The large-cluster Aggregate was designed with false invariants in mind, not real business rules
- transactional issues(db locks, concurrency): performance, scalability drawbacks

### Second attempt: multiple Aggregates
- solves the transaction failure issues by modeling it away
- less convinient
- can grow out of control

## Design Rules
### 1. Must understand the models' true invariants
> invariant is a business rule that must always be consistent(transactional).

```
Aggregate == transactional consistency boundary
```
A properly designed Aggregate is one that can be modified in any way requried by the business with its invariants completely consistent within __a single transaction__

### 2. Design small Aggregates
Limit the Aggregate to tjust the root entity and a minimal number of attributes and/or value-typed properties

### 3. Reference other Aggregates by identity
- Making Aggregates work together through identity references
- Model navigation through Disconnected Domain Model technique
  - Use a repository or Domain Service to look up dependent objects ahead of invoking the Aggregate behvior
  - If query overhead causes performance issues, consider the use of __theta joins__ or __CQRS__
  - If CQRS and theta joins are not an option, you may need to strike a balance between inferred and direct object reference
  - If all this advice seems to lead to a less convenient model, consider the additional benefits it affords: better perforance models plus easy to add scalability and distribution
- Scalability and Distribution
  - Almost infinite scalability is achieved by allowing for continuous repartitioning of Aggregate data storage as explained in [Life Beyond Distributed Transactions: An Apostate's Opinion](https://cs.brown.edu/courses/cs227/archives/2012/papers/weaker/cidr07p15.pdf) by Amazon.com's Pat Helland 

### 4. Use eventual consistency outside the boundary
- Ask whose job it is

## Reasons to break the rules
1. User interface convenience
2. Lack of technical methanisms
3. Global transactions
4. Query performance

## Implementation
### Create a root entity with unique identity
### Favor value object parts
> p.382 There are good reasons why ProductBacklogItem is modeled as an Entity rather than a Value. Since the backing database is used via Hibernate, it must model collections of Values as database entities. Reordering any one of the elements could cause a significant number of the ProductBacklogItem instances to be deleted and replaced. That would tend to cause significant overhead in the infrastructure. ... However, if we were to switch from using Hibernate with MySQL to a key-value store, we could easily change ProductBacklogItem to be a Value type instead. When using a key-value or document store, Aggregate instances are typically serialized as one value representation for storage.

### Using [Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter) and [Tell, Don't Ask](https://martinfowler.com/bliki/TellDontAsk.html)

### Optimistic concurrency
The Root's version would be incremented every time a state-altering command is executed _anywhere inside_ the Aggregate boundary

### Avoid dependency injection
the dependent object could be another Aggregate or a number of them. think of the potential overhead of injecting repository and domain service instances into Aggregates.
