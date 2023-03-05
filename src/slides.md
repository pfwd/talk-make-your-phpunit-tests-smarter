---
theme: uncover
class:
  - lead
  - invert
size: 16:9
footer: "Peter Fisher BSc MBCS howtocodewell.net @pfwd @howToCodeWell"
---

# Make your PHPUnit tests smarter

---
# Assumptions

- You know what PHPUnit is.
- PHPUnit is installed and configured.
- You have some tests.
<!--
- You want to know how to make tests smarter
-->

---
# Folder structure

Replicate the src structure

---
# Local src

![Src Structure](assets/images/folder_structure/src_structure.png)

---
# Test directory

![Src Structure](assets/images/folder_structure/unit_structure.png)

<!--
- It makes the tests searchable
- It makes the tests readable
- Treat integration and acceptance tests differently
- You can easily see what hasn't been tested
-->
---
# Namespaces
Replicate the namespaces

---
# src namespaces

```php
<?php

namespace App\AWS\Facade;

class Credentials
{
    //..
}
```

---
# Test namespaces

```php
<?php

namespace App\Tests\unit\AWS\Facade;

use PHPUnit\Framework\TestCase;

class CredentialsTest extends TestCase
{
    //..
}
```
<!--
- Less chance of namespace collisions
- Imports are easier add
- Keep App before Tests to allow for other applications 
-->
---
# Data provider

---
# No provider
```php
public function testReCalculateTotalPrice()
{
    $cartData = [
        ['total' => 10, /* ... */],
        ['total' => 20, /* ... */]        
    ];
    
    $cart = $this->mockCart($cartData);
    $manager = new CartManager($cart);
    $manager->reCalculate();

    self::assertSame(30, $cart->getTotalPrice());
}
```

---
# Using a data provider
```php
/**
 * @dataProvider cartDataProvider 
 */
public function testReCalculateTotalPrice(array $cartData, int|float $expected)
{     
    $cart = $this->mockCart($cartData);
    $manager = new CartManager($cart);
    $manager->reCalculate();

    self::assertSame($expected, $cart->getTotalPrice());
}
```
---
```php
    public function cartProvider(): array
    {
        return [
            [
                'cartData' => [
                    ['total' => 10, /* .. */],
                    ['total' => 20, /* .. */]
                ],
                'expected' => 30
            ],
            [
                'cartData' => [
                    ['total' => 5.50, /* .. */],
                    ['total' => 20.10, /* .. */]
                ],
                'expected' => 26
            ]
        ];
    }
```
<!--
- Keeps test code clean
- You only care about the output of the test
- Data can be shared across tests
- Data could come from DB fixture files
-->
---
#  Filters

```bash
phpunit --filter methodName 
```
This is safer
```bash
phpunit --filter methodName path/to/test/file.php
```
<!--
In example 1.
If you have two tests methods with the same methodName both will be processed.
Including the path to the test file isolates the test method.
-->
---

#  Groups

```bash
/**
 * @group fix-123
 * @group managers
 */
public function testSomeCode()

/**  @group managers */
public function testSomeMoreCode()
```
```bash
./bin/phpunit --group fix-123
./bin/phpunit --group managers
```
<!--
- Multiple groups can be used
- Groups can be associated to bugs
-->
---
# Test dependencies

A test method is not necessarily an encapsulated, independent entity. 

Often there are implicit dependencies between test methods, hidden in the implementation scenario of a test.

---

Test a product being added to an empty cart and then removed from the cart?

---

```php
public function testEmptyCart(): array
{
    $cart = new Cart();
    
    $this->assertEmpty($cart->getItems());
    
    return $cart;
}

```
---
```php
/** 
 * @depends testEmptyCart 
 */
public function testAddProduct(Cart $cart): array
{
    $cart->addItem(['id' => 123, /* ... */]);
    
    $this->assertSame(123, $cart->get(0)->getId());
    
    return $cart;
}
```
---
```php
/** 
 * @depends testAddProduct
 */
public function testRemoveProduct(Cart $cart): void
{
    $cart->removeItem(123);
    
    $this->assertFalse($cart->hasItem(123));
}
```

---
# Covers

`@covers` tells PHPUnit which parts of the code is supposed to be tested.
```php
/**
 * @covers \BankAccount
 */
public function testBalanceIsInitiallyZero(): void
{
    $this->assertSame(0, $this->ba->getBalance());
}
```
<!--
Tells the  coverage report to include the code which is covered.

-->

---
# Test everything

Automated code, models, entities, controllers and stuff outside src

<!--
- Test the automated code templates
- Test the automated code output
- Tests can help with spelling errors
- TDD helps with refactoring
-->
---
# Use multiple test suites

- Unit
- Integration
- Acceptance
- Different types of tests

--- 

```xml
<testsuites>
    <testsuite name="unit">
        <directory suffix="Test.php">tests/unit</directory>
    </testsuite>
    <testsuite name="command">
        <directory suffix="Test.php">tests/command</directory>
    </testsuite>
    <testsuite name="integration">
        <directory suffix="Test.php">tests/integration</directory>
    </testsuite>
    <testsuite name="all">
        <directory suffix="Test.php">tests/unit</directory>
        <directory suffix="Test.php">tests/command</directory>
        <directory suffix="Test.php">tests/integration</directory>
    </testsuite>
</testsuites>
```

---
# List test suites
```bash
./bin/phpunit --list-suites 
PHPUnit 9.5.21

Available test suite(s):
 - unit
 - command
 - integration
 - all
```
---
# Run test suites
```bash
./bin/phpunit --testsuite unit

./bin/phpunit --testsuite command

./bin/phpunit --testsuite integration

./bin/phpunit --testsuite all
```
---
# Naming your tests correctly

<!-- 
Your tests should be named in a way that explains exactly what the test is doing.

Your test should not do any more than its name.
-->
---
# Good test name

```php
public function testBalanceIsInitiallyZero() : void
```
<!-- 
- Says what that its testing (Balance)
- Says the expected state (Balance is initially)
- Says the expected end state (Zero)
-->
---

# Bad test name

```php
public function testCart() : void
```
<!-- 
- No idea what it's testing in the cart
- No idea what the end state should be
- Doesn't explain the how
-->

---

# Tighten up the mocks

```php
$mock = $this->createMock(AClass::class);
$mock->expects(self::once())    // Avoid any
    ->method('doSomething')     // Define the method name
    ->with('argumentValue')     // Define only the arguments needed
    ->willReturn('Response');   // Return exactly what will be returned
```


---

# Code coverage

<!--
- 80/20 rule
- Separate unit test coverage and acceptance test coverage
- Test you tests with infections
- Coverage helps when upgrading
-->

---
# MSI > code coverage
<!--
Mutation Score Indicator (MSI).

The MSI can be used to measure the effectiveness of a test set in terms of its ability to detect faults.

Mutants are modified parts of the code.
If the tests pass but don't handle the mutant then the mutant has escaped.
If the tests pass and the mutant has been handled then the mutant has been killed.
-->

---
# Resources

[infection.github.io](https://infection.github.io)

[docs.phpunit.de](https://docs.phpunit.de)