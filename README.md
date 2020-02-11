# hotpot-db

<img width="500px" src="https://66.media.tumblr.com/dc1e0c3d4372dd7a763cb3abba5c07b4/tumblr_ogk0t7i51o1vj3zbeo1_500.gifv"/>

### The 🌶🌶🌶 hottest way to store data

hotpot-db is a spicy, incredibly easy to use, and delcious database system.

```bash
# COMING SOON!
# hotpot_db = "0.0.0"
```

## What in the pot?

1. schemaless
2. reliable (uses SQLite3)
3. embeddable
4. fast
5. JSON store
6. queryable JSON schemas 


```rust
use hotpot_db::*;
use serde::{Deserialize, Serialize};
use serde_json::json;

#[derive(Debug, Serialize, Deserialize)]
struct Person {
    name: String,
    age: u8,
}

#[derive(Serialize, Deserialize)]
struct Grade {
    assignment: String,
    score: f32,
    person: Person,
}

fn main() -> Result<(), hotpot_db::Error> {
    let mut pot = HotPot::new();

    // lets make a new collection
    pot.create_collection("address_book")?;

    // let add people
    for name in vec!["david", "ness", "sakura"] {
        let person = Person {
            name: String::from(name),
            age: 26,
        };
        let json_to_store = serde_json::to_string(&person).unwrap();
        pot.add_object_to_collection("address_book", json_to_store)?;
    }

    // we can even more complex structs that are nested
    let grade = Grade {
        assignment: String::from("First Test"),
        score: 90.25,
        person: Person {
            name: String::from("drbh"),
            age: 101,
        },
    };
    let json_to_store = serde_json::to_string(&grade).unwrap();
    pot.add_object_to_collection("address_book", json_to_store)?;

    // here we can add arbitrary structs as long as they can be JSON serialized
    let random_data = vec![String::from("cheese"), String::from("art")];
    let new_json_to_store = json!(random_data).to_string();
    pot.add_object_to_collection("address_book", new_json_to_store)?;

    // lets query our hotpot!

    // this queries for ages that are a specific int value
    let query = QueryBuilder::new()
        .collection("address_book")
        .kind(QueryKind::Object)
        .key("age")
        .int(26)
        .finish();

    let results = pot.execute(query);
    println!("{:#?}", results);

    // this queries for scores that are a specific float value
    let query = QueryBuilder::new()
        .collection("address_book")
        .kind(QueryKind::Object)
        .key("score")
        .float(90.25)
        .finish();

    let results = pot.execute(query);
    println!("{:#?}", results);

    // this queries arrays that contain a string value
    let query = QueryBuilder::new()
        .collection("address_book")
        .kind(QueryKind::Contains)
        .string("cheese")
        .finish();

    let results = pot.execute(query);
    println!("{:#?}", results);

    // we can also do nested queries
    let query = QueryBuilder::new()
        .collection("address_book")
        .kind(QueryKind::Object)
        .key("person.age")
        .int(101)
        .finish();

    let results = pot.execute(query);
    println!("{:#?}", results);

    Ok(())
}
```