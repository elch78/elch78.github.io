How to write clean tests

- Builder and mother object pattern
- test fixtures

# Motherobject and Builder for easy test data creation.
- test can focus on the information that is relevant with zero noise (i.e. data that is not relevant)
- motherobjects for different commonly used objects e.g. aFoodProduct
- flexibility
- manually written builders allow more flexibility than generated one. E.g. can accept builders as parameters
- easy to maintain. If a mandatory field is added there is one place to fix all tests

  public class TestFixtures {


    public static ProductBuilder aFoodProduct() {
        return aProduct()
                .category("Food")
                .bestBefore(Instant.now().plus(2, WEEKS));
    }

  /**
    * Factory method for an arbitrary product with the mandatory fields initialized with random
    * values. Tests should initialize all fields that are relevant for the tests explicitly.
      */
      public static ProductBuilder aProduct() {
      return new ProductBuilder()
      .name(randomString())
      .serialNumber(randomString());
      }

  private static String randomString() {
  return RandomStringUtils.insecure().nextAlphabetic(10);
  }
  }

package de.elchworks.clean_tests.product;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.data.jdbc.test.autoconfigure.DataJdbcTest;
import org.springframework.boot.jdbc.test.autoconfigure.AutoConfigureTestDatabase;
import org.testcontainers.junit.jupiter.Testcontainers;

import static de.elchworks.clean_tests.TestFixtures.aProduct;
import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.boot.jdbc.test.autoconfigure.AutoConfigureTestDatabase.Replace.NONE;

@DataJdbcTest
@Testcontainers
// tell autoconfigure to not replace the database with an embedded one
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

    private void given(Product.ProductBuilder product) {
        sut.save(product.build());
    }
}

in contrast to test without builder and mother object

@Test           
public void findByName() {                                                                                                                                                                                                                                                      
// Given    
sut.save(new Product(null, "productA", null, null, "xk82a", false, null));                                                                                                                                                                                                  
sut.save(new Product(null, "productA", null, null, "m9f3z", false, null));
sut.save(new Product(null, "productB", null, null, "qw71p", false, null));

      // When
      var results = sut.findByName("productA");

      // Then
      assertThat(results).hasSize(2);
}

- Signal vs noise -- The clean version reads like a spec: "given a product named A, another named A, another named B — find by name A → 2 results." The raw version buries "productA" among 6 other arguments per line. You have to scan each constructor call to spot what
  differs.
- Serial numbers are a distraction -- "xk82a", "m9f3z", "qw71p" must be unique (it's a mandatory field), so you're forced to invent values that have nothing to do with the test. A reader might wonder: do these matter? With aProduct(), the randomization makes it obvious
  they don't.
- The given() helper removes mechanical noise -- sut.save(... .build()) is infrastructure. given(aProduct().name("productA")) is intent.
- 3 lines vs 3 lines, but very different readability -- The clean version has one concept per line (a product with a name). The raw version has seven concepts per line, six of which are irrelevant.
- only a very simple example. Real world code can and of does look much much messier
- The testdata is created through the business logic. Even with very complex test data it is easy to setup tests and the tests follow refactorings which is not the case if you use database dumps.