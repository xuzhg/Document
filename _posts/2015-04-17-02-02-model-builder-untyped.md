---
layout: post
title: "2.2 Build Edm Model Explicitly"
description: "convention model builder"
category: "2. Defining the model"
---

As mentioned in previous section, to build Edm Model explicitly is to create an `IEdmModel` object directly using ODatalib API. The Edm model built by this method is called **type-less model**, or **week type model**, or just **un-typed model**.
Let's see how to build the customer-order business model. 

### Add Entity Type

#### Basic Entity Type

We can use `EdmEntityType` to define an entity type as:
{% highlight csharp %}
EdmEntityType customer = new EdmEntityType("WebApiDocNS", "Customer");
model.AddElement(customer);

EdmEntityType order = new EdmEntityType("WebApiDocNS", "Order");
model.AddElement(order);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
    <EntityType Name="Customer" />
    <EntityType Name="Order" />
    ......
{% endhighlight %}

#### Derived Entity type

We can set the base type in construct to define a derived entity type as:
{% highlight csharp %}
EdmEntityType vipCustomer = new EdmEntityType("WebApiDocNS", "vipCustomer", customer);
model.AddElement(vipCustomer);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
    ......
    <EntityType Name="vipCustomer" BaseType="WebApiDocNS.Customer" />
{% endhighlight %}

#### Other Entity Types

We can call the following construct to set an entity type whether it is abstract or open.
{% highlight csharp %}
public EdmEntityType(string namespaceName, string name, IEdmEntityType baseType, bool isAbstract, bool isOpen);
{% endhighlight %}

For example:
{% highlight csharp %}
EdmEntityType customer = new EdmEntityType("WebApiDocNS", "Customer", baseType: null, isAbstract: true, isOpen: true);
model.AddElement(customer);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
    ......
    <EntityType Name="Customer" Abstract="true" OpenType="true" />
{% endhighlight %}

### Add Complex Type

#### Basic Complex Type

We can use `EdmComplexType` to define a complex type as:
{% highlight csharp %}
EdmComplexType address = new EdmComplexType("WebApiDocNS", "Address");
model.AddElement(address);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
    <Complex Name="Address" />
    ......
{% endhighlight %}

#### Derived Complex type

We can set the base type in construct to define a derived complex type as:
{% highlight csharp %}
EdmComplexType subAddress = new EdmComplexType("WebApiDocNS", "SubAddress", address);
model.AddElement(subAddress);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
    ......
    <ComplexType Name="SubAddress" BaseType="WebApiDocNS.Address" />
{% endhighlight %}

#### Other Complex Types
We can call the following construct to set a complex type whether it is abstract or open.
{% highlight csharp %}
public EdmComplexType(string namespaceName, string name, IEdmComplexType baseType, bool isAbstract, bool isOpen);
{% endhighlight %}

For example:
{% highlight csharp %}
EdmComplexType address = new EdmComplexType("WebApiDocNS", "Address", baseType: null, isAbstract: true, isOpen: true);
model.AddElement(address);
{% endhighlight %}

It will generate the below metadata document:
{% highlight xml %}
    ......
    <ComplexType Name="Address" Abstract="true" OpenType="true" />
{% endhighlight %}