We've evolved a naive test that validates too much into a very specific one
<br/>
Let's now improve a test that does not validate enough.

---

Let's test upserting some data into a table
| name | color |
|------|-------|
| apple| green |
| banana| yellow |

After upsert:
| name | color |
|------|-------|
| apple| green |
| banana| green |
| cherry| red |

---
Naive test that validates too little

```kotlin
// insert test data:
// Fruit("apple", "green"),
// Fruit("banana", "yellow"),

dao.upsert(listOf(
    Fruit("banana", "green"),
    Fruit("cherry", "red"),
))

dao.getAll().size shouldBe 3
```
---

Complete test that validates everything, hard to understand

```kotlin
// insert test data:
// Fruit("apple", "green"),
// Fruit("banana", "yellow"),

dao.upsert(listOf(
    Fruit("banana", "green"),
    Fruit("cherry", "red"),
))

dao.getAll() shouldContainExactlyInAnyOrder listOf(
    Fruit("banana", "green"),
    Fruit("cherry", "red"),
    Fruit("apple", "green"),
)
```
It might be good enough already
---
If, and only if, we need more readable test


```kotlin
// insert test data:
// Fruit("apple", "green"),
// Fruit("banana", "yellow"),

val rowsToUpsert = listOf(
    Fruit("banana", "green"), 
    Fruit("cherry", "red"),
    )
val rowsToKeepUnchanged = dao.getAll().filter { 
    it.name !in rowsToUpsert.map{ it.name}
}

dao.upsert()

dao.getAll() shouldContainExactlyInAnyOrder 
        rowsToKeepUnchanged + rowsToUpsert
```

