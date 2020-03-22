# Integrating Bounded Contexts

## Integration Basics
To integrate two Bounded Contexts,
   - API, RPCs
   - messaging (publisher-subscriber)
   - RESTful HTTP

Exchanging Information across System Boundaries
- programming language facillities to serialize objects into a binary format and deserialize on the consumer's side
- use some standard intermediate format (XML, JSON, Protocol Buffers..)

Define such a reliable contract using a standards-based approach, which actually forms a Published Language.
> Using the fully qualifed class name (pacakge name inclueded) for the typeName allows subscribers to precisely differentiate various types.

The consumer's Port Adapters should shield its domain model from any such dependencies and must insead pass needed Event data as appropriate parameters with types as defined only in its own Bounded Context

## Integration Using RESTful Resources
It is a kind of Open Host Service

Publishing a Shared Kernel or accepting a Conformist relationship puts consumers into a tightly coupled integration with the consumed domain model. Should be avoided.

In the Hexagonal or Ports and Adapters architecture, a consumer makes a request in the form

```
GET /tenants/{tenantId}/users/{username}/inRole/{role}
```

### Implementing the REST client using an anticorruption layer
Collaboration Context will interact with the Identity and Access Context and translate the user-in-role representation into a Value Object for a specific kind of Collaborator. In this particular case we do use a Separated Interface and an implementation class because the implementation is technical and should not reside in the Domain Layer.

## Integration Using Messaging
When method assignUser() of the Role completes, its last responsibility is to publish an Event: UserAssignedToRole

### Long Runnign Processes, and Avoiding Responsibility
> Does it make sense that an upstream Bounded Context listens for Events published from a downstream Context? Or, in an Event-Driven Architecture, are systems really upstream and downstream to each other? Need they be cas in that mold? 

Whatever the case, we need not be concerned because the idempotent operation allows for any number of infrastructure and architectural influences to be harnlessly ignored when they should be.

How would we ensure that the process is run completely to its finish?

### Process State Machines and Time-out Trackers
To clarify, the tracker is not part of the Core Domain.

The ProductService now provides the finishing behavor for the process, informing the tracker that it is completed(). From this point forward the tracker will no longer be selected as a retry or time-out notifier. The process is done.

### When Messaging or Your System is Unavailable
Back off attempts to send notifications until the messaging system is available once again.
