---
layout: post
title: "Clean Tests: Builder and Mother Object Patterns for Test Data That Reads Like a Spec"
date: 2026-02-05 10:00:00 +0000
categories: testing software-engineering code-quality
---

Here's a test. Tell me what it verifies:

```java
sut.save(new Product(null, "productA", null, null, "xk82a", false, null));
sut.save(new Product(null, "productA", null, null, "m9f3z", false, null));
sut.save(new Product(null, "productB", null, null, "qw71p", false, null));

var results = sut.findByName("productA");

assertThat(results).hasSize(2);
```

You probably figured it out. But you had to work for it. You had to scan past seven constructor arguments per line, mentally discard the nulls, wonder whether `"xk82a"` matters, and piece together what actually differs between the three products. For a test this simple, that effort is unnecessary. For tests with real-world complexity, it becomes a serious problem.

Now look at the same test written differently:

```java
given(aProduct().name("productA"));
given(aProduct().name("productA"));
given(aProduct().name("productB"));

var results = sut.findByName("productA");

assertThat(results).hasSize(2);
```

This version reads like a specification: *given a product named A, another named A, and another named B -- find by name A, expect 2 results.* No noise. No distractions. Just intent.

The difference comes down to two patterns: the **Builder pattern** and the **Mother Object pattern**. They're not new, but in my experience they remain underused in test code. And they make a remarkable difference in how maintainable a test suite becomes.

## The Problem With Raw Constructors in Tests

When you use constructors or factory methods with positional arguments directly in tests, you run into several issues.

**Mandatory fields that don't matter.** A product needs a serial number. It's required by the domain model. But in a test for `findByName`, the serial number is completely irrelevant. You're forced to invent values like `"xk82a"` and `"m9f3z"` that have nothing to do with the behavior under test. A reader encountering these values has to ask: do these matter? Are they part of the test scenario?

**Signal drowns in noise.** The raw constructor version has seven concepts per line, six of which are irrelevant. The thing you actually care about is buried among nulls and meaningless strings. You have to scan each constructor call to spot what differs between the three products.

**Maintenance becomes painful.** When a mandatory field is added to `Product`, every single test that constructs a product breaks. You end up with a tedious, error-prone search-and-replace across hundreds of test files.

## The Builder Pattern: Flexibility and Clarity

A builder lets each test express only the data that matters for that particular scenario:

```java
public class ProductBuilder {
    private String name = randomString();
    private String serialNumber = randomString();
    private String category;
    private Instant bestBefore;
    // ... other fields with sensible defaults

    public ProductBuilder name(String name) {
        this.name = name;
        return this;
    }

    public ProductBuilder category(String category) {
        this.category = category;
        return this;
    }

    public ProductBuilder bestBefore(Instant bestBefore) {
        this.bestBefore = bestBefore;
        return this;
    }

    public Product build() {
        return new Product(name, serialNumber, category, bestBefore, /* ... */);
    }
}
```

Mandatory fields like `name` and `serialNumber` get random default values. If a test doesn't mention a field, it's immediately clear that field is irrelevant to the scenario. And when the `Product` class changes, there's one place to update: the builder.

I'd recommend writing these builders manually rather than relying on generated ones. Hand-written builders give you more flexibility. For example, you can accept other builders as parameters, compose them, or add domain-specific convenience methods.

## The Mother Object Pattern: Reusable Test Scenarios

Where the builder provides flexibility, the Mother Object pattern provides common starting points. A mother object is a class with factory methods that return pre-configured builders for frequently used scenarios:

```java
public class TestFixtures {

    /**
     * Factory method for an arbitrary product with mandatory fields
     * initialized with random values. Tests should initialize all fields
     * that are relevant for the test explicitly.
     */
    public static ProductBuilder aProduct() {
        return new ProductBuilder()
                .name(randomString())
                .serialNumber(randomString());
    }

    public static ProductBuilder aFoodProduct() {
        return aProduct()
                .category("Food")
                .bestBefore(Instant.now().plus(14, DAYS));
    }

    private static String randomString() {
        return RandomStringUtils.insecure().nextAlphabetic(10);
    }
}
```

Notice how `aFoodProduct()` builds on `aProduct()`. You create a vocabulary of domain-specific factories -- `aProduct()`, `aFoodProduct()`, maybe `anExpiredProduct()` -- that capture common configurations. Tests read naturally because they use the language of the domain.

The key insight is that these factory methods return **builders, not finished objects**. This means tests can start with a convenient base and customize only what they need:

```java
given(aFoodProduct().name("Organic Milk"));
```

## Putting It All Together

Here's the full test class using both patterns:

```java
@DataJdbcTest
@Testcontainers
@AutoConfigureTestDatabase(replace = NONE)
class ProductRepositoryTest {

    @Autowired
    private ProductRepository sut;

    @Test
    public void findByName() {
        // Given
        given(aProduct().name("productA"));
        given(aProduct().name("productA"));
        given(aProduct().name("productB"));

        // When
        var results = sut.findByName("productA");

        // Then
        assertThat(results).hasSize(2);
    }

    private void given(ProductBuilder product) {
        sut.save(product.build());
    }
}
```

The `given()` helper is a small but meaningful touch. `sut.save(product.build())` is infrastructure plumbing. `given(aProduct().name("productA"))` is intent. The test reads as a clear Given-When-Then specification without any mechanical noise.

## Nested Builders: Where It Really Shines

The `findByName` example is deliberately simple. Where this approach really pays off is when your domain model has nested objects. Consider a food product with nutrition information -- calories, allergens, and so on. With nested builders, even these more complex scenarios stay readable:

```java
@Test
void findLowCalorieProducts() {
    // Given
    given(aFoodProduct().nutritionInfo(aNutritionInfo().calories(200)));
    given(aFoodProduct().nutritionInfo(aNutritionInfo().calories(800)));
    given(aFoodProduct().nutritionInfo(aNutritionInfo().calories(350)));

    // Then
    assertThat(sut.findByNutritionInfoCaloriesLessThan(200)).hasSize(0);
    assertThat(sut.findByNutritionInfoCaloriesLessThan(201)).hasSize(1);
    assertThat(sut.findByNutritionInfoCaloriesLessThan(351)).hasSize(2);
}

@Test
void findByAllergen() {
    // Given
    given(aFoodProduct().nutritionInfo(aNutritionInfo().allergens("nuts, dairy")));
    given(aFoodProduct().nutritionInfo(aNutritionInfo().allergens("gluten")));
    given(aFoodProduct().nutritionInfo(aNutritionInfo().allergens("nuts, soy")));

    // Then
    assertThat(sut.findByNutritionInfoAllergensContaining("nuts")).hasSize(2);
    assertThat(sut.findByNutritionInfoAllergensContaining("gluten")).hasSize(1);
    assertThat(sut.findByNutritionInfoAllergensContaining("egg")).hasSize(0);
}
```

Notice how `.nutritionInfo(aNutritionInfo().calories(200))` reads naturally -- a food product with nutrition info that has 200 calories. The builder for `NutritionInfo` is passed directly into the product builder. This is one of the reasons I recommend hand-writing your builders: you can design them to accept other builders as parameters, making this kind of composition feel effortless.

Now imagine writing these tests with raw constructors. Each `NutritionInfo` would need its own constructor call with every mandatory field spelled out, nested inside the `Product` constructor with all of *its* fields. The tests would become a wall of text where the actual scenario -- "find products under 200 calories" -- disappears behind constructor noise.

## Signal vs. Noise: A Side-by-Side Comparison

Let's be explicit about what changes:

| Aspect | Raw Constructor | Builder + Mother Object |
|--------|----------------|------------------------|
| Concepts per line | 7 (6 irrelevant) | 1 (the one that matters) |
| Mandatory but irrelevant fields | Must be invented manually | Randomized automatically |
| Reader's first question | "Do these values matter?" | "A product named X -- got it" |
| Adding a mandatory field | Fix every test file | Fix one builder |
| Reads like | A constructor call | A specification |

And this is a very simple example. Real-world code has entities with 15, 20, or more fields, nested objects, and complex initialization logic. The difference between the two approaches scales dramatically with complexity.

## Why Not Database Dumps?

There's an alternative approach I see teams reach for: loading test data from SQL dumps or JSON fixtures. This seems convenient at first -- just dump your test database and load it before each test.

The problem is that these dumps are opaque. When a field is added to the domain model, the dump silently becomes stale. When you refactor the domain, the dumps don't follow. And when a test fails, you're debugging data that was created outside of your test code, making it harder to understand the scenario.

With builders and mother objects, the test data is created through the same business logic your application uses. Even with very complex test data, the setup remains readable and the tests follow refactorings naturally.

## Practical Takeaways

- **Write a builder for every entity** that appears in more than a couple of tests. Initialize mandatory fields with random or sensible defaults.
- **Create a `TestFixtures` class** (or split into multiple classes by domain area) with factory methods for common scenarios. Name them using domain language: `aProduct()`, `aFoodProduct()`, `anExpiredProduct()`.
- **Return builders from factory methods**, not finished objects. This gives tests the flexibility to customize without sacrificing the convenience of pre-configured defaults.
- **Write builders by hand.** The small upfront effort pays off in flexibility and maintainability. You can add convenience methods, accept other builders as parameters, and evolve them with your domain.
- **Add `given()` helpers** to your test classes to hide infrastructure details like `save()` and `build()`. Let the test body express intent, not mechanics.
- **Use random values for irrelevant fields.** This communicates to the reader that those values don't matter for the test scenario. If a test breaks because of a random value, that's a signal the test has an implicit dependency you should make explicit.

Clean tests aren't just about aesthetics. A test suite where every test reads like a specification is one that teams actually trust, actually maintain, and actually use as documentation. The Builder and Mother Object patterns are simple tools that get you there.
