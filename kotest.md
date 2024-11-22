## Build Precise And Maintainable Tests With Kotest

#### Alex Kuznetsov

---

## First example: match lists

```kotlin
actual shouldBe expected
```
not easy to see what exactly is different

<img src="match-lists0.png" />

---

* `shouldBe` Is A Swiss Army Knife
* Does Lots Of Things
* Specialized Tools Are Better

<img src="swiss-army-knife.png" />

---

## It's a recurring problem
## Let's do something about it

I am not a visionary. I'm an engineer. I'm happy with the people who are wandering around looking at the stars but I am looking at the ground and I want to fix the pothole before I fall in.

Linus Torvalds

---

## We've built a better Matcher

<img src="match-ordered-lists0.png" />

---

## Match Two Slices

<img src="match-two-slices.png" />

---

* Kotest is designed and built by a committee
* Not very consistent, many redundancies
* But it's a committee including practitioners like you and I, not only by visionaries
* 377 contributors, 4.5K stars on GitHub
* If you have a problem, maybe kotest already has a solution

---


## When Order Does Not Matter - be specific

<img src="match-unordered-collections.png"  height="75%" width="75%"/>

---

## Non-Deterministic Order - be specific

<img src="nonDeterministicOrder.png" />

---

## `BigDecimal` and scale - be specific

test end result, not how it was computed

<img src="BigDecimalScale.png" />

---

## Double Numbers Are Not Precise - be specific

<img src="doubleNotPrecise.png" />

---

Let's see how a test evolves into a more specific and less fragile one

---

## Matching JSON - Naive Test

```kotlin
toPayload(myInstance) shouldBe 
        """{"destination":"01234","send_to":"Jane Doe"}"""
```

---

the order of fields in JSON should not matter, but this fails:

```kotlin
"""{"destination":"01234","send_to":"Jane Doe"}""" shouldBe 
        """{"send_to":"Jane Doe","destination":"01234"}"""
```

---

## `shouldEqualJson` is Specialized, More Robust

```kotlin
"""{
    "destination":"01234",
    "send_to":"Jane Doe"
    }
    """ shouldEqualJson 
        """{"send_to":"Jane Doe","destination":"01234"}"""
```

---


## `shouldEqualJson` is Specialized, More Robust, but

if we add a new field to the object being serialized, the test will fail:

```kotlin
val actual = """{
    "destination":"01234",
    "send_to":"Jane Doe",
    "weight":2.34}
    """
actual shouldEqualJson 
        """{"send_to":"Jane Doe","destination":"01234"}"""
```

---

## why do we even need this test? what exactly are we verifying?

We use a very common library to serialize objects to JSON. 
<br/>
<br/>
We should not be testing that library.

---

## Real Life Example 
Passing Around Zip Codes

```kotlin
data class ZipCode(
    val zip: String
) {
    public constructor(...) {
        //custom initialization logic
    }
    init {
        //validation
    }
}
// val destination: ZipCode should serialize to JSON 
// as "destination":"01234"
// not as "destination":{"val":"1234"}
```

---

- What we are testing: Is our custom serializer properly plugged in?
- be as specific as possible: `shouldContainJsonKeyValue`

```kotlin
val package = Package(
    destination = ZipCode(1234), 
    sendTo = "Jane Doe",
    weight = 2.34,
)
val actual = createPayload(package)
        
actual.shouldContainJsonKeyValue("destination", "01234") 
```
---

## Test One Specific Thing, Not The Whole Context

* tests will be precise and maintainable.
* but is can be more effort to write them.
* so it might or might not be worth it.

---

How to be more specific when matching data classes:

* Ignore timestamps, identities, UUIDs, etc.
* get good description of what exactly is different

---

## Match Data Classes

<img src="compareByFields.png" height="75%" width="75%"/>

---

## Match Only Some Fields

<img src="excludeFields.png" />

---

## Use `assertSoftly`

```kotlin
assertSoftly {
    actual.color shouldBe "red"
    actual.taste shouldBe "sweet"
}
```

---

## Add Field, Need To Update Test

Suppose we need to add `Fruit.weight` 

<img src="fragileTest.png" />

---

## Use Sample Instance 

```kotlin
val sampleFruit = Fruit("apple", "green", "sweet")

"use sampleFruit" {
 val tartRedFruit = sampleFruit.copy(
     color = "red", 
     taste = "tart"
 )
 canUseInSmoothie(tartRedFruit) shouldBe true
}
```

---

## use interface to reduce coupling

```kotlin
interface HasColorAndTaste {
  val color: String
  val taste: String
}

fun canUseInSmoothie(fruit: HasColorAndTaste): Boolean =
  fruit.color.length + fruit.taste.length > 5

val redAndSweet = object : HasColorAndTaste {
    override val color: String = "red"
    override val taste: String = "sweet"
}
canUseInSmoothie(redAndSweet) shouldBe true
```