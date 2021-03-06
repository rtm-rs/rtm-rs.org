+++
title = "Attributes"
description = "Working with attributes via the relations API."
date = 2022-05-01T15:00:00+00:00
updated = 2022-05-01T15:00:00+00:00
draft = false
weight = 50
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Working with attributes via the relations API.'
toc = true
top = false
+++

Relations provide a powerful API for working with their attributes. They can be used to rename
canonical columns, qualify them, compose more complex `WHERE` conditions, or create SQL functions.

## Qualifying and Renaming Attributes

Joining relations introduces a problem of having conflicting attribute names. To
solve this you often need to qualify and rename columns.

To qualify all attributes in a relation:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true) do
    associations do
      has_many :tasks
      has_many :posts
    end
  end

  def with_tasks
    join(:tasks, user_id: :id)

    # the same will be done when you use a shortcut:
    join(tasks)
  end
  # produces "SELECT users.id, users.name ..."
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true) do
    associations do
      has_many :tasks
      has_many :posts
    end
  end

  def with_tasks
    join(:tasks, user_id: :id)

    # the same will be done when you use a shortcut:
    join(tasks)
  end
  # produces "SELECT users.id, users.name ..."
end
```

{% end %}

To rename all attributes in a relation:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true) do
    associations do
      has_many :tasks
      has_many :posts
    end
  end

  def with_tasks
    prefix(:user)
  end
  # produces "SELECT users.id AS user_id, users.name AS user_name ..."
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true) do
    associations do
      has_many :tasks
      has_many :posts
    end
  end

  def with_tasks
    prefix(:user)
  end
  # produces "SELECT users.id AS user_id, users.name AS user_name ..."
end
```

{% end %}

To rename attributes in block-based DSLs you can use `as` method:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { [id.as(:user_id), name.as(:user_name) }
  end
  # produces "SELECT users.id AS user_id, users.name AS user_name ..."
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { [id.as(:user_id), name.as(:user_name) }
  end
  # produces "SELECT users.id AS user_id, users.name AS user_name ..."
end
```

{% end %}`

## Creating functions from attributes

You can use an attribute to create a function. This is useful in cases where
you would like to append a function that's created from an attribute coming
from another relation. For example:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select(:id, :name, tasks[:id].func { int::count(id).as(:task_count) }).
      left_join(tasks).
      group(:id)
  end
  # SELECT "users"."id", "users"."name", COUNT("tasks"."id") AS "task_count" FROM "users" LEFT JOIN "tasks" ON ("users"."id" = "tasks"."user_id") GROUP BY "users"."id" ORDER BY "users"."id"
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select(:id, :name, tasks[:id].func { int::count(id).as(:task_count) }).
      left_join(tasks).
      group(:id)
  end
  # SELECT "users"."id", "users"."name", COUNT("tasks"."id") AS "task_count" FROM "users" LEFT JOIN "tasks" ON ("users"."id" = "tasks"."user_id") GROUP BY "users"."id" ORDER BY "users"."id"
end
```

{% end %}

## Using attributes for restrictions

You can use Attribute API in restriction methods such as `#where`:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    where { name.is('Jane') | name.is('Joe') }
  end
  # SELECT "id", "name" FROM "users" WHERE (("name" = 'Jane') OR ("name" = 'Joe')) ORDER BY "users"."id""
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    where { name.is('Jane') | name.is('Joe') }
  end
  # SELECT "id", "name" FROM "users" WHERE (("name" = 'Jane') OR ("name" = 'Joe')) ORDER BY "users"."id""
end
```

{% end %}

## Learn more

Check out API documentation:

* [api::rom-sql::SQL](Attribute)
* [api::rom-sql::SQL](Schema)
