# Fixture customization

AutoFixture is designed around the 80-20 (Pareto) principle. It is expected that the default `Fixture` instance will be able to create specimens witout too much trouble, in most cases. However there are situations, where AutoFixture needs some extra help.

- When a class consumes an interface, the default `Fixture` will not be able to create an instance. (Solved with AutoMocking Customizations)
- When the API has circular references, Fixture might enter an infinite recursion.
- When constructors accept only arguments that don't fit with the default specimens created by the `Fixture`.

To address these special cases, AutoFixture offers extension points, allowing client code to customize the default behavior.

AutoFixture also offers an idiomatic way to encapsulate these customizations and also keep the code DRY, called _Customizations_. This concept corresponds closely to modularization APIs of seveal well-known DI containers, like Castle Windsor's _Installers_ or Autofac's _Modules_.

## Using customizations

Customizing AutoFixture using _Customizations_ is a simple and straight-forward process. All that is necessary is to provide the `Fixture` an implementation of the `ICustomization` interface, using the `Customize` method.

```cs
var fixture = new Fixture()
    .Customize(new AutoMoqCustomization());
```

In the example above, the `Fixture` instance is customized by the client code using the `AutoMoqCustomization` provided in the [AutoFixture.AutoMoq](https://www.nuget.org/packages/AutoFixture.AutoMoq/) package. Note that almost every extension library provided by AutoFixture offers at least one Customization.

## Implementing customizations

Customizations are simply implementations of the `ICustomization` interface, that requires clients to implement a single method `Customize`. The method receives a single argument, which is the `Fixture` instance itself.

```cs
public interface ICustomization
{
    void Customize(IFixture fixture);
}
```

Anything that can be done using the `Fixture` instance can also be done with the `IFixture` instance passed in the customization.

## Reusing customizations

By design customizations are intended to be small in scope, reusable and composable. Whenever implementing a custom customization, it is recommended to adhere to the DRY principle and avoid defining all the configuration inside a single customizatino instance.

To be able to use multiple atomic customizations it is recommended to use the composite pattern. For convenience AutoFixture offers the `CompositeCustomization` class that implements the composite pattern for the `ICustomization` interface.

```cs
var fixture = new Fixture()
    .Customize(new CompositeCustomization(
      new AutoMoqCustomization(),
      new NullRecursionCustomization(),
      new CustomersCustomization()
    ));
```

Composite customizations can also be implemented by inheriting from `CompositeCustomization`. This offers the benefit of having descriptive names for sets of related customizations.

```cs
class DomainCustomization : CompositeCustomization
{
  public DoaminCustomization()
    : base(
        new CustomersCustomization(),
        new AddressCustomization())
  {
  }
}
```

## Extension points

The `Fixture` class is a specialization that packages the components of the kernel in a very specific way to implement the features and behavior of AutoFixture as shown on this illustration:

<img src="../../assets/images/high-level-layout.png" class="center" />

When you create a `Fixture` instance using the default constructor it prepares a set of Specimen Builders called the *engine parts*, corresponding to the middle block of the illustration. These Specimen Builders contain logic that handles well-known primitive types such as integers and strings, as well as a set of components that use Reflection to instantiate complex types via their constructors.

### Customizations

In a default `Fixture`, the `Customizations` collection is empty, but we can use it to customize the Fixture instance by adding `ISpecimenBuilder` instances. Since the kernel uses the first specimen created by an `ISpecimenBuilder`, adding a Specimen Builder to the Customization collection intercepts the default engine parts, allowing the customization to get a shot at each request before the default engine parts handle it.

The `Customize<T>` method of Fixture itself uses this structure to customize the instance.

### Residue Collectors

There are requests that a Fixture instance cannot satisfy. As an example, a request for an interface type cannot be satisfied because an interface has no public constructors. Adding an `ISpecimenBuilder` instance to the `ResidueCollectors` collection gives that Specimen Builder a chance to handle such residue before Fixture throws an exception. The optional [AutoMoq extension](http://blog.ploeh.dk/2010/08/19/AutoFixtureAsAnAutomockingContainer.aspx), for instance, is implemented via a custom Residue Collector that handle requests for interfaces by returning a mock of that interface.

### Behaviors

A Behavior is simply a [Decorator](http://en.wikipedia.org/wiki/Decorator_pattern) that can monitor or intercept requests and specimens going in and out of the Fixture instance. Tracing and Recursion Guarding are implemented as Behaviors.
