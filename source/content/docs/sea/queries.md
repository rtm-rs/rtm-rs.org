+++
title = "Queries"
description = "Queries: Composite or singular."
date = 2022-05-01T15:00:00+00:00
updated = 2022-05-01T15:00:00+00:00
draft = false
weight = 40
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Queries: Composite or singular.'
toc = true
top = false
+++

## Default `by_pk` method

All relations come with the default `#by_pk` method. It supports composite PKs too.

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
# with a single PK
users.by_pk(1)

# with a composite [post_id, tag_id] PK
posts_tags.by_pk(1, 2)
```

---

```rust
# with a single PK
users.by_pk(1)

# with a composite [post_id, tag_id] PK
posts_tags.by_pk(1, 2)
```

{% end %}

## Selecting columns

To explicitly select columns you can either use a list of symbols or relation schema:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select(:id, :name)

    # or

    select(*schema.project(:id, :name))

    # or

    select(self[:id], self[:name])
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select(:id, :name)

    # or

    select(*schema.project(:id, :name))

    # or

    select(self[:id], self[:name])
  end
end
```

{% end %}

In a basic case, which is selecting unqualified columns using their canonical names,
a list of symbols is all you need. Schemas and their attributes are useful in more
complex cases, so it's beneficial to know that you can use them in `select` method.


## Appending more columns

If you have a relation with some columns already selected and you want to add more,
you can use `select_append` method:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select(:id, :name)
  end

  def details
    index.select_append(:email, :created_at, :updated_at)
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select(:id, :name)
  end

  def details
    index.select_append(:email, :created_at, :updated_at)
  end
end
```

{% end %}

## Projection DSL

Both `select` and `select_append` accept a block which exposes projection DSL.
You can use it for simple selection of columns, or results of SQL functions.

### Projecting attributes

Within the block you can refer to relation attributes by their names and use
[api::rom-sql::SQL](Attribute) API for projections:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { [id, name] }
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { [id, name] }
  end
end
```

{% end %}

### Projecting function results

Apart from returning column values, you can also project function results:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { [name, integer::count(id).as(:count)] }.group(:id)
    # SELECT "name", COUNT("id") AS "count" ...
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { [name, integer::count(id).as(:count)] }.group(:id)
    # SELECT "name", COUNT("id") AS "count" ...
  end
end
```

{% end %}

Functions can accept any number of arguments:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { string::concat(id, '-', name).as(:uid) }
    # SELECT CONCAT("id", '-', "name") AS "uid" ...
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def index
    select { string::concat(id, '-', name).as(:uid) }
    # SELECT CONCAT("id", '-', "name") AS "uid" ...
  end
end
```

{% end %}

## Restricting relations

To restrict a relation you can use `where` method which accepts a hash with
conditions or a block for more advanced usage.

### Simple conditions

If you pass a hash to `where` all conditions will be translated into SQL and ANDed together:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def by_name(name)
    where(name: name)
    # ... WHERE ("name" = 'Jane') ...
  end

  def admin_by_name(name)
    where(name: name, admin: true)
    # ... WHERE ("name" = 'Jane') AND ("admin" IS TRUE) ...
  end

  def by_ids(ids)
    where(ids: ids)
    # ... WHERE ("id" IN (1, 2)) ...
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def by_name(name)
    where(name: name)
    # ... WHERE ("name" = 'Jane') ...
  end

  def admin_by_name(name)
    where(name: name, admin: true)
    # ... WHERE ("name" = 'Jane') AND ("admin" IS TRUE) ...
  end

  def by_ids(ids)
    where(ids: ids)
    # ... WHERE ("id" IN (1, 2)) ...
  end
end
```

{% end %}

### Complex conditions

If you pass a block to `where` you can use restriction DSL to compose more complex conditions:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def query
    where { (id < 10) | (id > 20) }
    # (("id" < 10) OR ("id" > 20))

    where { id.not(10..20) }
    # (("id" < 10) OR ("id" > 20))

    where { id.in(1..10) & id.in(20..100) }
    # (("id" >= 1) AND ("id" <= 10) AND ("id" >= 20) AND ("id" <= 100))

    where { name.ilike('%an%') }
    # ("name" ILIKE '%an%' ESCAPE '\\')
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def query
    where { (id < 10) | (id > 20) }
    # (("id" < 10) OR ("id" > 20))

    where { id.not(10..20) }
    # (("id" < 10) OR ("id" > 20))

    where { id.in(1..10) & id.in(20..100) }
    # (("id" >= 1) AND ("id" <= 10) AND ("id" >= 20) AND ("id" <= 100))

    where { name.ilike('%an%') }
    # ("name" ILIKE '%an%' ESCAPE '\\')
  end
end
```

{% end %}

## Aggregations and HAVING

To create `HAVING` clause simply use `having` method, which works in a similar way as `where`
and supports creating aggregate functions for your conditions:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def email_duplicates
    select { [email, integer::count(id).as(:count)] }.
      group(:email).
      having { count(id) >= 2 }
      # ... HAVING (count("id") >= 2) ...
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def email_duplicates
    select { [email, integer::count(id).as(:count)] }.
      group(:email).
      having { count(id) >= 2 }
      # ... HAVING (count("id") >= 2) ...
  end
end
```

{% end %}

## Order

`order` method with block will order your query:

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def query
    order { created_at.desc }
    # ... ORDER BY "created_at" DESC

    order { created_at.asc }
    # ... ORDER BY "created_at" ASC
  end
end
```

---

```rust
class Users < ROM::Relation[:sql]
  schema(infer: true)

  def query
    order { created_at.desc }
    # ... ORDER BY "created_at" DESC

    order { created_at.asc }
    # ... ORDER BY "created_at" ASC
  end
end
```

{% end %}

## Nullify

`nullify` makes it trivial to implement the Null Object pattern on rom-sql
relations.  After calling `nullify`, there will never issue a query to the database.

[Check out the Sequel docs for more information.](http://sequel.jeremyevans.net/rdoc-plugins/files/lib/sequel/extensions/null_dataset_rb.html)

{% fenced_code_tab(tabs=["ruby", "rust"]) %}

```ruby
class Tasks < ROM::Relation[:sql]
  schema(infer: true)

  def for_working_status(status)
    return nullify if status == :on_vacation
    self
  end
end

tasks = ROM.env.relations[:tasks]
tasks.for_working_status(:on_vacation).count # => 0
```

---

```rust
class Tasks < ROM::Relation[:sql]
  schema(infer: true)

  def for_working_status(status)
    return nullify if status == :on_vacation
    self
  end
end

tasks = ROM.env.relations[:tasks]
tasks.for_working_status(:on_vacation).count # => 0
```

{% end %}

## Learn more

Check out API documentation:

* [api::rom-sql::SQL/Relation](Reading)
