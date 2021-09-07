# Welcome to AutoFixture

AutoFixture is a tool designed to make Test-Driven Development more productive and unit tests more refactoring-safe. It does so by removing the need for hand-coding anonymous variables as part of a test's Arrange phase:

```cs
[Fact]
public void IntroductoryTest()
{
    // Arrange
    Fixture fixture = new Fixture();

    int expectedNumber = fixture.Create<int>();
    MyClass sut = fixture.Create<MyClass>();
    // Act
    int result = sut.Echo(expectedNumber);
    // Assert
    Assert.Equal(expectedNumber, result);
}
```

This example illustrates the basic principle of AutoFixture: it can create values of virtually any type without the need for you to explicitly define which values should be used.

AutoFixture offers integration with **xUnit** and **NUnit** making your unit tests even more concise:

```cs
[Theory, AutoData]
public void IntroductoryTest(int expectedNumber, MyClass sut)
{
    int result = sut.Echo(expectedNumber);
    Assert.Equal(expectedNumber, result);
}
```

AutoFixture also provides integration with popular mocking libraries (**NSubstitute**, **Moq**, **FakeItEasy**). See AutoFixture + xUnit + NSubstitute in action:

```cs
[Theory, AutoNSubstituteData] // Notice, AutoNSubstituteData __is not__ provided out of the box.
public void IntroductoryTest(int expectedNumber, INumberSource numberSource, MyClass sut)
{
    numberSource.Number.Returns(expectedNumber);

    int result = sut.Echo(numberSource);

    Assert.Equal(expectedNumber, result);
}
```

## Integration Libraries

Additionally AutoFixture provides a wide selection of integration libraries, that integrate the library with unit-testing frameworks, mocking tools and more.

Currently AutoFixture supports the most popular unit-testing tools in the .NET ecosystem (e.g. **xUnit**, **NUnit**, **Moq**, **NSubstitute** and more).
