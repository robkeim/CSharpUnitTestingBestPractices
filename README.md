# Unit Testing in C#

## Purpose
The purpose of this best practices document is to offer suggested best practices when writing unit tests in C#.

## What Makes a Good Unit Test
- **Fast**. It is not uncommon for mature projects to have thousands of unit tests. Unit tests should take very little time to run. Milliseconds.
- **Isolated**. Unit tests are to be run in isolation and have no dependencies on an outside factor such as a file system or database.
- **Repeatable**. Running a unit test should be consistent in its results, that is, it always returns the same result if you do not change anything inbetween runs.
- **Self-Checking**. The test should be able to automatically detect if it passed or failed without any human interaction.
- **Timely**. A unit test and the code being tested should not take a disporpotionaly long time to write.

## Lets Speak the Same Language
The term *mock* is unfortunately very misused when talking about testing. The following defines the most common types of *fakes* when writing unit tests:

*Fake* - A fake is a generic term which can be used to describe either a Stub or a Mock object. Whether it is a Stub or a Mock depends on the context in which it's used. So in other words, a Fake can be a Stub or a Mock.

*Mock* - A mock object is a fake object in the system that decides whether or not a unit test has passed or failed. A Mock starts out as a Fake until it is asserted against.

*Stub* - A stub is a controllable replacement for an existing dependency (or collaborator) in the system. By using a stub, you can test your code without dealing with the dependency directly. By default, a fake starts out as a stub.

Consider the following code snippet:

```csharp
var mockOrder = new MockOrder();
var purchase = new Purchase(mockOrder);

purchase.ValidateOrders();

Assert.True(purchase.CanBeShipped);
```

This would be an example of Mock being used improperly. In this case, it is a stub. We're just passing in the Order as a means to be able to instantiate `Purchase` (the system under test). The name `MockOrder` is also very misleading because again, the order is not a mock.

A better approach would be

```csharp
var fakeOrder = new FakeOrder();
var purchase = new Purchase(fakeOrder);

purchase.ValidateOrders();

Assert.True(purchase.CanBeShipped);
```

By renaming the class to `FakeOrder`, we've made the class a lot more generic, the class can be used as a mock or a stub. Whichever is better for the test case. In the above example, `FakeOrder` is used as a stub. We're not using the `FakeOrder` in any shape or form during the assert. We just passed it into the `Purchase` class to satisfy the requirements of the constructor.

To use it as a Mock, we could do something like this

```csharp
var fakeOrder = new FakeOrder();
var purchase = new Purchase(fakeOrder);

purchase.ValidateOrders();

Assert.True(fakeOrder.Validated);
```

In this case, we are checking a property on the Fake (asserting against it), so in the above code snippet the `fakeOrder` is a Mock.

**It's important to get this terminology correct. If you call your stubs "mocks", other developers are going to make false assumptions about your intent.**

The main thing to remember about mocks versus stubs is that mocks are just like stubs, but you assert against the mock object, whereas you do not assert against a stub. Which means that only mocks can break your tests, not stubs.


## Best Practices
1. [Arranging Your Tests](#arranging-your-tests)
1. [Naming Your Tests](#naming-your-tests)

### Arranging Your Tests
The AAA (Arrange, Act, Assert) pattern is a typical pattern when unit testing, and consists of three main actions:
- *Arrange* your objects, creating and setting them up as necessary.
- *Act* on an object.
- *Assert* that something is as expected.

#### Why?
- Clearly separates what is being tested from the setup and verification steps.
- Less chance to intermix assertions with "Act" code

#### Bad:
```csharp
public void IsValidWord_WhenInputIsNull_ReturnsFalse()
{
    // Arrange
    var glossary = new Glossary();

    // Assert
    Assert.False(glossary.IsValidWord(null));
}
```

#### Best:
```csharp
public void IsValidWord_WhenInputIsNull_ReturnsFalse()
{
    var numberValidator = new NumberValidator();
    
    var result = numberValidator.IsValidWord(null);

    Assert.False(result);
}
```

### Naming Your Tests
The name of your test should consist of three parts
- The name of the method being tested
- The scenario under which it's being tested
- The expected behavior when the scenario is invoked

#### Why?
- Naming standards are important because they explicitly express intent of the test.

#### Bad:
```csharp
public void Test_Invalid()
{
    var glossary = new Glossary();

    var result = glossary.IsValidWord(null);

    Assert.False(result);
}
```

### Better:
```csharp
public void IsValidWord_WhenInputIsNull_ReturnsFalse()
{
    var glossary = new Glossary();

    var result = glossary.IsValidWord(null);

    Assert.False(result);
}
```

