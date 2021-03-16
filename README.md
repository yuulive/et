# et

[crates.io](https://crates.io/crates/et)

### Et - the 🌶🌶🌶 hottest way to store data

et is a spicy, incredibly easy to use, and delicious database system.

```bash
et = "0.0.1"
```

## Flavor Palette

1. schemaless
2. reliable (uses SQLite3)
3. embeddable
4. fast (<200ms to search through +500K objects)
5. JSON store
6. queryable JSON schemas 


## Example
```rust
use et::*;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct Person {
    name: String,
    age: u8,
}

fn main() -> Result<(), et::Error> {
    // must pass a path to init
    let mut pot = Et::new(".");

    // lets make a new collection
    pot.create_collection("address_book")?;

    // well make a new item we want to store
    let person = Person {
        name: String::from("david holtz"),
        age: 26,
    };

    // we insert the object into the collection!
    pot.insert::<Person>("address_book", &person)?;

    // before we query we can add an index to speed things up
    pot.add_index_to_collection("address_book", "name", "naming_index")?;

    // finally we can query
    let query = QueryBuilder::new()
        .collection("address_book")
        .kind(QueryKind::Object)
        .key("name")
        .comparison("=")
        .string("david holtz")
        .finish();

    let results = pot.execute(query);
    println!("{:#?}", results);

    Ok(())
}
```

## Recipe

et is made from few, but time tasted ingredients. It is a new approach to an old dish. 

**Ingredients**
1. 1 cup, `SQLite 3.30.1`
2. 2 tablespoons, `Rust`
3. A pinch, `JSON serde`

## Concepts

#### Collection  
In a technical sense, a collection is just a table in SQLite, that stores data in a specific format. Each row is an `Entry` which consists of three columns: id, time_created, data. The data column holds each JSON object and the other columns are used as et metadata.  

In theory, a collection should house similar data to make it easier to manage, but et doesn't care about schema so you can store any kind of object in a single collection.

#### Objects

Each entry contains an object and the are the heart of et. Objects are special because you can query their contents efficiently. 

This is an advantage over storing JSON in other datastores since you don't have to read the full object to query the contents. et wraps SQLite's json1 extension into an easy to use API. 


## Speed Estimates

Objects allow us to store schemaless data and still search through it efficiently. Query's on small dbs ~10MB run in <5ms and tested queries on larger DB's ~100MB run <500ms.

## Query Kinds

In a hot pot you can only query in two different ways. You can check the contents of an array or the attribute/values of an object.

et offers the developer a simple QueryBuilder that allows you to conveniently write and read your queries. 

#### Querying Arrays
```rust
let query = QueryBuilder::new()
    .collection("transaction_records")
    .kind(QueryKind::Contains)
    .comparison(">")
    .int(100)
    .finish();
```

#### Querying Objects
```rust
let query = QueryBuilder::new()
    .collection("address_book")
    .kind(QueryKind::Object)
    .key("name")
    .comparison("=")
    .string("david holtz")
    .finish();
```

### Changelogs

- March 24th, PR to allow user to specify location of the database, thanks @juzi5201314 for the clean code https://github.com/drbh/et/pull/3

- March 25th allow user to overwrite entries at specific index with new API `upsert_at_index`

- March 26th change Option to enum. Avoid the possibility of unexpected results from overloaded input. thanks @HiruNya for pointing this bug out https://github.com/drbh/et/issues/2

