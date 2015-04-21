---
layout: post
title: "2.7 Complex Type"
description: "convention model builder"
category: "2. Defining the model"
---

[From OData V4 Spec](http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398351), it says:

```
***Complex types*** are **keyless** named structured types consisting of a set of properties.These are value types whose instances cannot be referenced outside of their containing entity. Complex types are commonly used as property values in an entity or as parameters to operations.
```

Besides, **Complex type** can be abstract (that is, it cannot be instantiated directly) and can be open (that is, it can has both declared properties and dynamic properties).

To build an Edm Model using non-convention model builder is to create an `IEdmModel` object by directly call fluent APIs of `ODataModelBuilder`. The developer should take all responsibility to add all Edm types, operations, associations, etc into the data model one by one.
Let's see how to build the customer-order business model by `ODataModelBuilder`.

### Build complex type

non-convention model builder is based on CLR classes to build the Edm Model. The customer-oder business CLR classes are present in ![](./2015-04-17-02-01-model-builder-abstract.md)



### Complex Type

In Web API OData, there are three ways to build complex type.

#### Untyped

To build complex type explicitly is to call ODataLib APIs directly. For example:

{% highlight csharp %}
public static IEdmModel GetExplicitModel()
{
    EdmModel model = new EdmModel();
	
	EdmComplexType address = new EdmComplexType("WebApiDocNS", "Address");
    address.AddStructuralProperty("Country", EdmPrimitiveTypeKind.String);
    address.AddStructuralProperty("City", EdmPrimitiveTypeKind.String);
    model.AddElement(address);
	
	return model;
}
	
#### Typed

{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<ComplexType Name="Address">
  <Property Name="Country" Type="Edm.String" />
  <Property Name="City" Type="Edm.String" />
</ComplexType>
{% endhighlight %}

#### Derived Complex type

The following codes are used to add a complex type:
{% highlight csharp %}
var subAddress = builder.ComplexType<SubAddress>().DerivesFrom<Address>();
subAddress.Property(s => s.Street);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<ComplexType Name="SubAddress" BaseType="WebApiDocNS.Address">
  <Property Name="Street" Type="Edm.String" />
</ComplexType>
{% endhighlight %}

#### Abstract Complex type

The following codes are used to add an abstract complex type:
{% highlight csharp %}
builder.ComplexType<Address>().Abstract();
......
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<ComplexType Name="Address" Abstract="true">
  ......
</ComplexType>
{% endhighlight %}

#### Open Complex type

In order to build an open complex type, you should change the CLR class by adding an `IDictionary<string, object>` property, the property name doesn't matter. For example:

{% highlight csharp %}
public class Address
{
    public string Country { get; set; }
    public string City { get; set; }
    public IDictionary<string, object> Dynamics { get; set; }
}
{% endhighlight %}

Then you can build the open complex type as:
{% highlight csharp %}
var address = builder.ComplexType<Address>();
address.Property(a => a.Country);
address.Property(a => a.City);
address.HasDynamicProperties(a => a.Dynamics);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<ComplexType Name="Address" OpenType="true">
  <Property Name="Country" Type="Edm.String" />
  <Property Name="City" Type="Edm.String" />
</ComplexType>
{% endhighlight %}
You can find that the complex type `Address` only has two properties, while it has `OpenType="true"` attribute.

### Entity Type

#### Basic Entity Type

The following codes are used to add two entity types:
{% highlight csharp %}
var customer = builder.EntityType<Customer>();
customer.HasKey(c => c.CustomerId);
customer.ComplexProperty(c => c.Location);
customer.HasMany(c => c.Orders);

var order = builder.EntityType<Order>();
order.HasKey(o => o.OrderId);
order.Property(o => o.Token);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<EntityType Name="Customer">
    <Key>
        <PropertyRef Name="CustomerId" />
    </Key>
    <Property Name="CustomerId" Type="Edm.Int32" Nullable="false" />
    <Property Name="Location" Type="WebApiDocNS.Address" Nullable="false" />
    <NavigationProperty Name="Orders" Type="Collection(WebApiDocNS.Order)" />
</EntityType>
<EntityType Name="Order">
    <Key>
        <PropertyRef Name="OrderId" />
    </Key>
    <Property Name="OrderId" Type="Edm.Int32" Nullable="false" />
    <Property Name="Token" Type="Edm.Guid" Nullable="false" />
</EntityType>
{% endhighlight %}

#### Abstract Open type

The following codes are used to add an abstract complex type:
{% highlight csharp %}
builder.EntityType<Customer>().Abstract();
......
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<EntityType Name="Customer" Abstract="true">
  ......
</EntityType>
{% endhighlight %}


#### Open Entity type

In order to build an open entity type, you should change the CLR class by adding an `IDictionary<string, object>` property, while the property name doesn't matter. For example:

{% highlight csharp %}
public class Customer
{
    public int CustomerId { get; set; }
    public Address Location { get; set; }
    public IList<Order> Orders { get; set; }
    public IDictionary<string, object> Dynamics { get; set; }
}
{% endhighlight %}

Then you can build the open entity type as:
{% highlight csharp %}
var customer = builder.EntityType<Customer>();
customer.HasKey(c => c.CustomerId);
customer.ComplexProperty(c => c.Location);
customer.HasMany(c => c.Orders);
customer.HasDynamicProperties(c => c.Dynamics);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<EntityType Name="Customer" OpenType="true">
    <Key>
        <PropertyRef Name="CustomerId" />
    </Key>
    <Property Name="CustomerId" Type="Edm.Int32" Nullable="false" />
    <Property Name="Location" Type="WebApiDocNS.Address" Nullable="false" />
    <NavigationProperty Name="Orders" Type="Collection(WebApiDocNS.Order)" />
</EntityType>
{% endhighlight %}
You can find that the entity type `Customer` only has three properties, while it has `OpenType="true"` attribute.

### Entity Container

Non-convention model builder will build the default entity container automatically. However, you should build your own entity sets as:
{% highlight csharp %}
builder.EntitySet<Customer>("Customers");
builder.EntitySet<Order>("Orders");
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<Schema Namespace="Default" xmlns="http://docs.oasis-open.org/odata/ns/edm">
    <EntityContainer Name="Container">
        <EntitySet Name="Customers" EntityType="WebApiDocNS.Customer">
          <NavigationPropertyBinding Path="Orders" Target="Orders" />
        </EntitySet>
        <EntitySet Name="Orders" EntityType="WebApiDocNS.Order" />
    </EntityContainer>
</Schema>
{% endhighlight %}

Besides, you can call `Singleton<T>()` to add singleton into entity container.

### Function

We define two functions. One is bound, the other is unbound as:
{% highlight csharp %}
IEdmTypeReference stringType = EdmCoreModel.Instance.GetPrimitive(EdmPrimitiveTypeKind.String, isNullable: false);
IEdmTypeReference intType = EdmCoreModel.Instance.GetPrimitive(EdmPrimitiveTypeKind.Int32, isNullable: false);
EdmFunction getFirstName = new EdmFunction("WebApiDocNS", "GetFirstName", stringType, isBound: true, entitySetPathExpression: null, isComposable: false);
getFirstName.AddParameter("entity", new EdmEntityTypeReference(customer, false));
model.AddElement(getFirstName);

EdmFunction getNumber = new EdmFunction("WebApiDocNS", "GetOrderCount", intType, isBound: false, entitySetPathExpression: null, isComposable: false);
model.AddElement(getNumber);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
<Function Name="GetFirstName" IsBound="true">
   <Parameter Name="entity" Type="WebApiDocNS.Customer" Nullable="false" />
   <ReturnType Type="Edm.String" Nullable="false" />
</Function>
<Function Name="GetOrderCount">
   <ReturnType Type="Edm.Int32" Nullable="false" />
</Function>
{% endhighlight %}