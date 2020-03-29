# Application

the finest set of components that are assembled to interact with and support a Core Domain model.

## User Interface

### Use a Mediator to publish Aggregate internal state
To work around the problem of tight coupling between the model and its clitns, choose to design Mediator interfaces to which the Aggregate publishes its internal state.

### State representations of Aggregate instances
It is very important to create representations that are based on use case, not on Aggregate instances. Resist the temptation to produce representations that are a one-to-one- reflection of your domain model Aggregate states.

### Dealing with multiple, disparate clients
Design Application Services to accept Data Transformer, where each client specifies the Data Transformer type.

## Application Services
Keep Application Services thin, using them only to coordinate tasks on the model.

