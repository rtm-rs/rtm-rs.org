+++
title = "Glossary"
description = "This document explains the basic terms used in RTM."
date = 2022-05-01T19:30:00+00:00
updated = 2022-05-01T19:30:00+00:00
draft = false
weight = 30
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "This document explains the basic terms used in RTM."
toc = true
top = false
+++

#### Relation

An object that represents data in your system, implements `each` and yields
[tuples](#tuple).

#### Repository

An object that uses [relations](#relation) to interact with a datastore.  A repository implements a trait that is used to define a clear API between your datastore and your application.

#### Mapper

An object that receives a relation and maps it to another representation. Anything
that responds to `call` can be a mapper in RTM.

#### Command

An object that executes a datastore-specific operation to create, update,
or delete tuples in a relation. Commands execute operations using the relation
interface, which is datastore-specific; however, on the surface, they simply respond
to `call`.

Every [adapter](#adapter) *can provide custom command types*.

#### Tuple

An element in a relation. It's typically represented by a hash object.

#### Datastore

A persistence backend, typically a database but also a flat file with data like
CSV or YAML.

#### Gateway

An object that encapsulates access to a specific persistence backend. For example
the SQL gateway provides access to database tables via its [datasets](#dataset)

#### Dataset

A raw source of data with an interface specific for a given datastore. [Relations](#relation)
use datasets to fetch data from persistence backends, like databases. The dataset
interface is not directly exposed to the application layer; however, it is
available as a private interface of [relations](#relation).

#### Adapter

An adapter is a library providing infrastructure for RTM to support specific
persistence backends, e.g. a JSON file, SQLite database, etc.
An adapter ships with its own [`Gateway`](#gateway), [`Dataset`](#dataset), [`Relation`](#relation) and [`Command`](#command) traits.
To implement a new adapter, say `rtm-yaml`, one would implement these four traits.

### Patterns

RTM is implemented using a couple of fundamental patterns that make it flexible
and extendible.

#### Callable Functional Objects

Relation, mapper, and command types are callable, they
receive input and return output with no run-time side-effects. Furthermore, all
types don't have a mutable state, and it's safe to memoize them and rely on
consistent behavior.

#### Data Pipeline

[Relations](#relation), [mappers](#mapper) and [commands](#command) can be composed into a data pipeline, which is a
simple idea that one type returns a relation and passes it to another. All types
respond to `call` and accept a relation and implement the `>>` operator
([`core::ops::Shr`](https://doc.rust-lang.org/stable/core/ops/trait.Shr.html)), which is used to construct the pipeline.

#### Graph

A nested datum with a root and its nodes. RTM allows you to build `relation`
and `command` graphs for working with nested structures and associations.

#### CQRS

***RTM is not a CQRS framework.***

Command and Query Responsibility Segregation (CQRS), is a way of organizing your application so that reading
data is separated from changing data - the responsibility of handling commands
(mutations) from the responsibility of handling side-effect-free query/read access.
Martin Fowler has a succinct synopsis of [CQRS and its downsides](https://martinfowler.com/bliki/CQRS.html).

Using RTM, part of this pattern is easily achievable with [relations](#relation) and [commands](#command).  Because RTM may be useful when implementing a CQRS pattern, it does not follow that a project adopting RTM must commit to the full complexity of CQRS or Event-Sourcing.

See the [FAQ entry for CQRS](./faq) for additional detail.

#### ES

***RTM is not a ES framework, but it does have parts useful to such a pattern.***

Event Sourcing (ES) uses events generated by CQRS commands, reads and writes as the source of truth for the state of the application.
Martin Fowler has a synopsis of [ES](https://martinfowler.com/eaaDev/EventSourcing.html).

See the [FAQ entry for ES](./faq/) for additional detail.