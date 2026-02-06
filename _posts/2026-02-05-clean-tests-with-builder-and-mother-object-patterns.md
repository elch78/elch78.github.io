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

For a long time I thought of tests primarily as a safety net to prevent regressions. Only recently did I learn that tests have another equally important benefit: **documentation**. A good test tells you something about the expectations of the system. It captures what the code *should* do in a way that stays in sync with the actual behavior, unlike comments or wikis that rot over time. But tests can only serve as documentation if they're readable. And that's where test suites often fall short.

Two patterns that are very helpful for tests are the **Builder pattern** and the **Mother Object pattern**. They're not new, but in my experience they remain underused in test code. And they make a remarkable difference in how maintainable a test suite becomes.

## The Problem With Raw Constructors in Tests

When you use constructors or factory methods with positional arguments directly in tests, you run into several issues.

**Mandatory fields that don't matter.** A product needs a serial number. It's required by the domain model. But in a test for `findByName`, the serial number is completely irrelevant. You're forced to invent values like `"xk82a"` and `"m9f3z"` that have nothing to do with the behavior under test. A reader encountering these values has to ask: do these matter? Are they part of the test scenario?

**Signal drowns in noise.** The raw constructor version has seven concepts per line, six of which are irrelevant. The thing you actually care about is buried among nulls and meaningless strings. You have to scan each constructor call to spot what differs between the three products.

**Maintenance becomes painful.** When a mandatory field is added to `Product`, every single test that constructs a product breaks. You end up with a tedious, error-prone search-and-replace across hundreds of test files.

## The Builder Pattern: Flexibility and Clarity

The builder pattern provides a fluent API to construct objects in a very flexible way. Instead of positional arguments the arguments are named. This pattern is
well adopted.

```java
Product product = Product.builder()
        .name(randomString())
        .serialNumber(randomString())
        .build();
```

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
        return Product.builder()
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

The basic factory method `aProduct()` provides a pre-initialized product. The mandatory fields like `name` and
`serialNumber` get random default values. The purpose is to provide an empty object that can be used by a test out of
the box. The random values help to avoid accidental matching. Tests are expected to use this builder and initialize any
value that matters for the tests. If a test doesn't
mention a field, it's immediately clear that field is irrelevant to the scenario. And when the `Product` class changes,
there's one place to update: the builder.

Notice how `aFoodProduct()` builds on `aProduct()`. You create a vocabulary of domain-specific factories --
`aProduct()`, `aFoodProduct()`, maybe `anExpiredProduct()` -- that capture common configurations. Tests read naturally
because they use the language of the domain.

The key insight is that these factory methods return **builders, not finished objects**. This means tests can start with
a convenient base and customize only what they need:

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

The `given()` helper is a small but meaningful touch. `sut.save(product.build())` is infrastructure plumbing.
`given(aProduct().name("productA"))` is intent. The test reads as a clear Given-When-Then specification without any
mechanical noise.

## Nested Builders: Where It Really Shines

The `findByName` example is deliberately simple. Where this approach really pays off is when your domain model has
nested objects. Consider a food product with nutrition information -- calories, allergens, and so on. With nested
builders, even these more complex scenarios stay readable:

```java
@Test
void findByNutritionInfoCaloriesLessThan() {
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
void findByNutritionInfoAllergensContaining() {
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

Notice how `.nutritionInfo(aNutritionInfo().calories(200))` reads naturally. The builder for `NutritionInfo` is passed
directly into the product builder. This is one of the reasons I recommend hand-writing your builders: you can design
them to accept other builders as parameters, making this kind of composition feel effortless.

And this is a very simple example. Real-world code has entities with 15, 20, or more fields, nested objects, and complex
initialization logic. The difference between the two approaches scales dramatically with complexity.

## All that matters at one glance

Writing and maintaining test must be easy. If it is a chore developers avoid it. Another benefit of easy setup is that
all that matters for a test is visible at one glance. What happens if the setup is a lot of work is that repetitive
stuff is outsourced into @BeforeAll methods which makes a test much harder to understand especially if class hierarchies
are involved. This approach only hides the complexity. The builder and mother object pattern also hides the complexity
but keeps the relevant signals visible.

And finally if the setup is cohesivea test that is not needed anymore there is a higher chance that the test can just be
deleted without touching other code.

## From Unit Tests to End-to-End: The Scenario Pattern

These patterns aren't limited to repository tests. They scale all the way up to end-to-end tests. The idea is to
introduce a `Scenario` class -- a test-scoped Spring bean that provides convenience methods for setting up preconditions
and executing actions through real services. No mocking, no shortcuts. Everything runs against a real database via
Testcontainers.

Note: I don't advocate to test everything end to end. I've written about my stance on Unit- vs. Integrationtests in a
previous post [Rethinking Testing: Speed and Behavior matter more than nomenclature]({% post_url
2026-01-09-Rethinking-Testing-Speed-and-Behavior-matter-more-than-nomenclature %})

```java
@Component
public class Scenario {
    private final ProductService productService;
    private final CartService cartService;

    public Scenario(ProductService productService, CartService cartService) {
        this.productService = productService;
        this.cartService = cartService;
    }

    public Product given(ProductBuilder productBuilder) {
        return productService.createProduct(productBuilder.build());
    }

    public Cart givenCart() {
        return cartService.createCart();
    }

    public Cart addProductToCart(UUID cartId, UUID productId) {
        return cartService.addProduct(cartId, productId);
    }

    public Cart getCart(UUID cartId) {
        return cartService.getCart(cartId);
    }
}
```

```java
@SpringBootTest
class CartFeatureTest {

    @Autowired
    private Scenario scenario;

    @Test
    void addProductToCart() {
        // Given
        Product product = scenario.given(aProduct());
        Cart cart = scenario.givenCart();

        // When
        scenario.addProductToCart(cart.getId(), product.getId());

        // Then
        Cart result = scenario.getCart(cart.getId());
        assertThat(result.getItems())
                .extracting(CartItem::productId)
                .containsExactly(product.getId());
    }
}
```

Read that test aloud: *given a product and a cart, when the product is added to the cart, then the cart contains that
product.* It's practically a specification. The `Scenario` class hides how products are persisted, how carts are
created, and how the "add to cart" operation works internally. The test only expresses what matters: the business
behavior.

The `Scenario` bean delegates to real services -- the same ones your application uses in production. This means the test
exercises the full stack: controllers, services, repositories, database. But the test itself reads like a high-level
description of the feature.

This is only one small step away from BDD. The `scenario.given()`, `scenario.addProductToCart()`, and
`scenario.getCart()` methods are essentially step definitions. Converting these tests into Cucumber or JBehave
specifications would be almost mechanical. The patterns give you that option without forcing you into a BDD framework
upfront.

## Generate test data using the business logic vs. database dumps

There's an alternative approach I see teams reach for: loading test data from SQL dumps or JSON fixtures. This seems
convenient at first -- just dump your test database and load it before each test.

The problem is that these dumps are opaque. When a field is added to the domain model, the dump silently becomes stale.
When you refactor the domain, the dumps don't follow. And when a test fails, you're debugging data that was created
outside of your test code, making it harder to understand the scenario.

With builders and mother objects, the test data is created through the same business logic your application uses. Even
with very complex test data, the setup remains readable and the tests follow refactorings naturally.

## JSON test data

What I also often encounter is json test data that is loaded from files not seldomly dozens. A heap of copies of the same
json structure with tons of noise and important bits scattered all over the place but not where it matters: in the test.
A nightmare to maintain if the structure changes. The builder and mother object pattern works perfectly for these cases as well.

A stupid simple example of a builder for json that I've actually used like this. Writing a builder like this is a matter
of minutes - seconds with LLMs today.

This gives you all the expressive power described above. KISS

```java
public class ProductJsonBuilder {
    private String name;
    private String category;
    private String manufacturer;

    public ProductJsonBuilder name(String name) {
        this.name = name;
        return this;
    }

    public ProductJsonBuilder category(String category) {
        this.category = category;
        return this;
    }

    public ProductJsonBuilder manufacturer(String manufacturer) {
        this.manufacturer = manufacturer;
        return this;
    }

    public String build() {
        return """
                {
                    "name": "%s",
                    "category": "%s",
                    "manufacturer": "%s",
                    "serialNumber": "SN-12345",
                    "inStock": true
                }
                """.formatted(name, category, manufacturer);
    }
}
```

## Practical Takeaways

- **Write a builder for every entity** Initialize mandatory fields with random or sensible defaults so that the objects are usable out of the box and the tests can focus on what matters.
- **Create a `TestFixtures` class** (or split into multiple classes by domain area) with factory methods for common scenarios. Name them using domain language: `aProduct()`, `aFoodProduct()`, `anExpiredProduct()`.
- **Return builders from factory methods**, not finished objects. This gives tests the flexibility to customize without sacrificing the convenience of pre-configured defaults.
- **Write builders by hand.** The small upfront effort pays off in flexibility and maintainability. You can add convenience methods, accept other builders as parameters, and evolve them with your domain.
- **Add `given()` helpers** to your test classes to hide infrastructure details like `save()` and `build()`. Let the test body express intent, not mechanics.
- **Use random values for irrelevant fields.** This communicates to the reader that those values don't matter for the test scenario. If a test breaks because of a random value, that's a signal the test has an implicit dependency you should make explicit.

Clean tests aren't just about aesthetics. A test suite where every test reads like a specification is one that teams
actually trust, maintain, and use as documentation. The Builder and Mother Object patterns are simple
tools that get you there.
I've written more about why I prefer testing behaviors end-to-end rather than obsessing over unit vs. integration test
labels in [Rethinking Testing: Speed and Behavior matter more than nomenclature]({% post_url
2026-01-09-Rethinking-Testing-Speed-and-Behavior-matter-more-than-nomenclature %}). I mentioned that I consider tests as
first class citizens that deserve the same code quality as the business logic. With this post I want to show how far you
can get even without
Gherkin and Cucumber.
