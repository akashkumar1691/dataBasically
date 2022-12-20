# moocBase2
a relational database management system that includes B+ tree indices, efficient join algorithms, query optimization, multigranularity locking to support concurrent execution of transactions, and database recovery

## Usage of Database

The `Database` class represents the entire database. It is the public interface
of our database - we do not parse SQL statements in our database, and instead,
users of our database use it like a Java library.

All work is done in transactions, so to use the database, a user would start
a transaction with `Database#beginTransaction`, then call some of
`Transaction`'s numerous methods to perform selects, inserts, and updates.

For example:
```java
Database db = new Database("database-dir");

try (Transaction t1 = db.beginTransaction()) {
    Schema s = new Schema(
        Arrays.asList("id", "firstName", "lastName"),
        Arrays.asList(Type.intType(), Type.stringType(10), Type.stringType(10))
    );
    t1.createTable(s, "table1");
    t1.insert("table1", Arrays.asList(
        new IntDataBox(1),
        new StringDataBox("John", 10),
        new StringDataBox("Doe", 10)
    ));
    t1.insert("table1", Arrays.asList(
        new IntDataBox(2),
        new StringDataBox("Jane", 10),
        new StringDataBox("Doe", 10)
    ));
    t1.commit();
}

try (Transaction t2 = db.beginTransaction()) {
    // .query("table1") is how you run "SELECT * FROM table1"
    Iterator<Record> iter = t2.query("table1").execute();

    System.out.println(iter.next()); // prints [1, John, Doe]
    System.out.println(iter.next()); // prints [2, Jane, Doe]

    t2.commit();
}

db.close();
```

More complex queries can be found in
[`src/test/java/edu/berkeley/cs186/database/TestDatabase.java`](src/test/java/edu/berkeley/cs186/database/TestDatabase.java).
