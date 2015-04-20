---
layout: post
title: "2.2 Abstract"
description: "convention model builder"
category: "2. Defining the model"
---

The data model is the basis of an OData service. OData service uses an abstract data model called **Entity Data Model** (*EDM*) to describe the exposed data in the service. OData client can issue a *GET* request to the root URL of the OData service with `$metadata` to get an XML representation of the service's data model. 
In Microsoft ASP.NET Web API 2.2 for OData v4.0, to build a data model for OData service is to create an `IEdmModel` object. There are three ways to build an EDM model in Web API OData:
1.	Explicit Edm Model Builder
2.	Implicit, non-convention model builder or fluent API
3.	Implicit, convention model builder.

### 2.1.1 Build Edm Model
Let’s see the difference between them.

#### Explicit model builder
To build an Edm Model explicitly is to create an IEdmModel object by directly using APIs in ODatalib. The basic code structure to build an Edm Model explicitly is shown as:
{% highlight csharp %}
public IEdmModel GetEdmModel()
{
    EdmModel model = new EdmModel();
    // ......
    return model;
}
{% endhighlight %}
The Edm Model built by this way is called un-typed (or typeless, week type) Edm model. Because there is no CLR classes.


#### Non-convention model builder
To build an Edm Model using non-convention model builder is to create an IEdmModel object by directly call fluent APIs of ODataModelBuilder. The developer should take all responsibility to add all types, operations, associations, etc into the data model one by one. The basic code structure of this way is shown as:
{% highlight csharp %}
public static IEdmModel GetEdmModel()
{
    var builder = new ODataModelBuilder();
    // ......
    return builder.GetEdmModel();
}
{% endhighlight %}

#### Convention model builder
To build an Edm Model using convention model builder is to create an IEdmModel object by a set of conventions. Such conventions are pre-defined in Web API OData to help model builder to identify types, keys, association etc in the finial Edm model. ODataConventionModelBuilder wrappers these conventions and apply them to the Edm model when building. The basic code structure of this way is shown as:
{% highlight csharp %}
public static IEdmModel GetEdmModel()
{
    var builder = new ODataConventionModelBuilder();
    // ......
    return builder.GetEdmModel();
}
{% endhighlight %}

Basically, it’s recommended to use convention model builder to build Edm Model for its simplicity and convenient. However, if user wants to make more control on the model building, or he doesn’t have the corresponding CLR classes, the non-convention model builder and the explicitly method are also useful.

### 2.1.2 Edm Model Sample 
We’ll build an Edm model using the above three methods in the following sections respectively. Each section is designed to walk you through every required aspect to build such Edm model. First of all, let’s have a brief view about the Edm model we will build.
This is a customer-order model, in which three entity types, two complex types and one enum type are included. Here’s the detail information about each types:
*	Customer is served as an entity type with three properties. 
*	CustomerId: structural property, primitive type, key
*	Location: structural property, complex type
*	Orders: navigation property, entity type


{% highlight csharp %}

            ODataConventionModelBuilder builder = new ODataConventionModelBuilder();
            builder.Namespace = "ODataSamples.WebApiService.Models";
            builder.ContainerName = "DefaultContainer";
            builder.EntityType<Trip>();
            builder.GetEdmModel()
{% endhighlight %}

It will generate the below entity type in the resulted EDM document:

{% highlight xml %}

      <EntityType Name="Trip">
        <Key>
          <PropertyRef Name="TripId" />
        </Key>
        <Property Name="TripId" Type="Edm.Int32" Nullable="false" />
        <Property Name="ShareId" Type="Edm.Guid" />
        <Property Name="Name" Type="Edm.String" />
      </EntityType>
{% endhighlight %}

Convention: any .NET class with one or more key properties can be an entity type. (What makes a key property? Please continue reading)


### Entity key

In a model class, if one and only one property’s name is ‘Id’ or ‘`<entity class name>`Id’ (case insensitive), it becomes entity key property.

### Key attribute
The [`KeyAttribute`] specifies key property, it forces a property without 'id' to be Key property:

{% highlight csharp %}

    public class Trip    {
        [Key]
        public int TripNum { get; set; }
        public Guid? ShareId { get; set; }
        public string Name { get; set; }
    }
{% endhighlight %}

The result is:
{% highlight xml %}

	<EntityType Name="Trip">
	  <Key>
	    <PropertyRef Name="Tripnum" />
	  </Key>
	  <Property Name="Tripnum" Type="Edm.Int32" Nullable="false" />
	  <Property Name="ShareId" Type="Edm.Guid" />
	  <Property Name="Name" Type="Edm.String" />
	</EntityType>
{% endhighlight %}

### ComplexType
The convention is: a model class without Key property is a complex type. There are 2 ways to create complex type:

(1)	Create a class without any ‘id’ or ‘<class name>id’ or [`KeyAttribute`] property, like

{% highlight csharp %}

    public class City
    {
        public string Name { get; set; }
        public string CountryRegion { get; set; }
        public string Region { get; set; }
    }
{% endhighlight %}

(2)	Or add [ComplexType] attribute to a model class: it will remove ‘id’ or ‘<class name>id’ or [Key] properties, the model class will have no entity key, thus becomes a complex type.

{% highlight csharp %}

    [ComplexType]
    public class PairItem
    {
        public int Id { get; set; }
        public string Value { get; set; }
    }
{% endhighlight %}

### Abstract entity
ODataConventionModelBuilder can generate abstract entity from abstract model class:

{% highlight csharp %}

    public abstract class EventBase
    {
        [Key]
        public string EventIdentifier { get; set; }
    }
{% endhighlight %}

The result is :

{% highlight xml %}
	
	<EntityType Name="EventBase" Abstract="true">
	  <Key>
	    <PropertyRef Name="EventIdentifier" />
	  </Key>
	  <Property Name="EventIdentifier" Type="Edm.String" Nullable="false" />
	</EntityType>
{% endhighlight %}


### DataContract & DataMember
They select what properties to be serialized & deserialized. The below example shows that ‘SharedId‘ property without [DataMember] attribute is eliminated from the EDM model.

{% highlight csharp %}

    [DataContract]
    public class Trip
    {
        [DataMember]
        [Key]
        public int TripNum { get; set; }
        public Guid? ShareId { get; set; }  // will be eliminated
        [DataMember]
        public string Name { get; set; }
    }
{% endhighlight %}

The resulted EDM document is:

{% highlight xml %}

	<EntityType Name="Trip">
	  <Key>
	    <PropertyRef Name="TripNum" />
	  </Key>
	  <Property Name="TripNum" Type="Edm.Int32" Nullable="false" />
	  <Property Name="Name" Type="Edm.String" />
	</EntityType>
{% endhighlight %}

They can also change namespace and Name in EDM document, if the above DataContract attribute is added with NameSpace:

{% highlight csharp %}

	[DataContract(Namespace="My.NewNameSpace")]
{% endhighlight %}

The result will become:

{% highlight csharp %}

	<Schema Namespace="My.NewNameSpace">
	  <EntityType Name="Trip">
	    <Key>
	      <PropertyRef Name="TripNum"/>
	    </Key>
	    <Property Name="TripNum" Type="Edm.Int32" Nullable="false"/>
	    <Property Name="Name" Type="Edm.String"/>
	  </EntityType>
	</Schema>
{% endhighlight %}

### NotMapped
NotMapped deselects the property to be serialized or deserialized, so to some extent, it can be seen as the converse of DataContract & DataMember. For example, the above if Trip class is changed to the below, it generates exactly the same Trip Entity in EDM document, that is, no ‘SharedId’ property.

{% highlight csharp %}

    [DataContract]
    public class Trip
    {
        [DataMember]
        [Key]
        public int TripNum { get; set; }
        [NotMapped]
        public Guid? ShareId { get; set; }
        [DataMember]
        public string Name { get; set; }
    }	
{% endhighlight %}


{% highlight xml %}

	<EntityType Name="Trip">
	  <Key>
	    <PropertyRef Name="TripNum"/>
	  </Key>
	  <Property Name="TripNum" Type="Edm.Int32" Nullable="false"/>
	  <Property Name="Name" Type="Edm.String"/>
	</EntityType>
{% endhighlight %}

### Required
It sets property to be Nullable false. The below has [Required] on Name property:

{% highlight csharp %}

    public class Trip
    {
        [Key]
        public int TripNum { get; set; }
        [NotMapped]
        public Guid? ShareId { get; set; }
        [Required]
        public string Name { get; set; }
    }
{% endhighlight %}

Then the result has Nullable=”false” for Name property:

{% highlight xml %}

    <EntityType Name="Trip">
	  <Key>
	    <PropertyRef Name="TripNum"/>
	  </Key>
	  <Property Name="TripNum" Type="Edm.Int32" Nullable="false"/>
	  <Property Name="Name" Type="Edm.String" Nullable="false"/>
    </EntityType>
{% endhighlight %}


### ConcurrencyCheck
It can mark one or more properties for doing optimistic concurrency check on entity updates.

{% highlight csharp %}

    public class Trip
    {
        [Key]
        public int TripNum { get; set; }
        [NotMapped]
        public Guid? ShareId { get; set; }
        [Required]
        public string Name { get; set; }
        [ConcurrencyCheck]
        public string UpdateVersion { get; set; }
    }
{% endhighlight %}


The expected result should be like the below: (however, because of a bug, as of the post being written, the result is still not correct for OData V4)

{% highlight xml %}

	<EntityType Name="Trip">
	  <Key>
	    <PropertyRef Name="TripNum"/>
	  </Key>
	  <Property Name="TripNum" Type="Edm.Int32" Nullable="false"/>
	  <Property Name="Name" Type="Edm.String" Nullable="false"/>
	  <Property Name="UpdateVersion" Type="Edm.String"/>
	  <Annotation Term="Core.OptimisticConcurrency">
	    <Collection>
	      <PropertyPath>UpdateVersion</PropertyPath>
	    </Collection>
	  </Annotation>
	</EntityType>
{% endhighlight %}


### Timestamp


{% highlight csharp %}

    public class Trip
    {
        [Key]
        public int TripNum { get; set; }
        [NotMapped]
        public Guid? ShareId { get; set; }
        [Required]
        public string Name { get; set; }
        [ConcurrencyCheck]
        public string UpdateVersion { get; set; }
        [Timestamp]
        public string UpdateStamp { get; set; }
    }
{% endhighlight %}

The expected result is (again, there is bug preventing it from generating correct annotation now) :

{% highlight xml %}

	<EntityType Name="Trip">
	  <Key>
	    <PropertyRef Name="TripNum"/>
	  </Key>
	  <Property Name="TripNum" Type="Edm.Int32" Nullable="false"/>
	  <Property Name="Name" Type="Edm.String" Nullable="false"/>
	  <Property Name="UpdateVersion" Type="Edm.String"/>
	<Property Name="UpdateStamp" Type="Edm.String"/>
	  <Annotation Term="Core.OptimisticConcurrency">
	    <Collection>
	      <PropertyPath>UpdateVersion</PropertyPath>
	      <PropertyPath>UpdateStamp</PropertyPath>
	    </Collection>
	  </Annotation>
	</EntityType>
{% endhighlight %}


### IgnoreDataMember
It has the same effect as `[NotMapped]` attribute. It is able to revert the `[DataMember] `attribute on the property when the model class doesn’t have `[DataContract]` attribute.

### Singleton

{% highlight csharp %}

	builder.Singleton<Person>("Me");
{% endhighlight %}

The result is:

{% highlight xml %}
	
	<EntityContainer Name="DefaultContainer">
	<Singleton Name="Me" Type="ODataSamples.WebApiService.Models.Person">
	</Singleton>
	</EntityContainer>
{% endhighlight %}

### EntitySet

{% highlight csharp %}

	builder.EntitySet<Person>("People");
{% endhighlight %}

The result is:

{% highlight xml %}
	
	<EntityContainer Name="DefaultContainer">
	  <EntitySet Name="People" EntityType="ODataSamples.WebApiService.Models.Person">
	  </EntitySet>
	</EntityContainer>
{% endhighlight %}

### Bound function
            var personType = builder.EntityType<Person>();

            personType.Function("GetFriendsTrips")
                .ReturnsCollectionFromEntitySet<Airline>("Airlines")
                .Parameter<string>("userName");
The result is:

{% highlight xml %}
	
	<Function Name="GetFriendsTrips" IsBound="true">
	  <Parameter Name="bindingParameter" Type="ODataSamples.WebApiService.Models.Person"/>
	  <Parameter Name="userName" Type="Edm.String" Unicode="false"/>
	  <ReturnType Type="Collection(ODataSamples.WebApiService.Models.Airline)"/>
	</Function>
{% endhighlight %}


### Bound action
            var personType = builder.EntityType<Person>();

            var shareTripAction = personType.Action("ShareTrip");
            shareTripAction.Parameter<string>("userName");
            shareTripAction.Parameter<int>("tripId");

The result is:

{% highlight xml %}

	<Action Name="ShareTrip" IsBound="true">
	  <Parameter Name="bindingParameter" Type="ODataSamples.WebApiService.Models.Person"/>
	  <Parameter Name="userName" Type="Edm.String" Unicode="false"/>
	  <Parameter Name="tripId" Type="Edm.Int32" Nullable="false"/>
	</Action>
{% endhighlight %}



### Unbound function
            var GetNearestAirportFun = builder.Function("GetNearestAirport");
            GetNearestAirportFun.Parameter<double>("lat");
            GetNearestAirportFun.Parameter<double>("lon");
            GetNearestAirportFun.ReturnsFromEntitySet<Airport>("Airports");
The result is:

{% highlight xml %}

	<Function Name="GetNearestAirport">
	  <Parameter Name="lat" Type="Edm.Double" Nullable="false"/>
	  <Parameter Name="lon" Type="Edm.Double" Nullable="false"/>
	  <ReturnType Type="ODataSamples.WebApiService.Models.Airport"/>
	</Function>
	
	<FunctionImport Name="GetNearestAirport" Function="ODataSamples.WebApiService.Models.GetNearestAirport" EntitySet="Airports" IncludeInServiceDocument="true"/>
{% endhighlight %}


### Unbound action

            var checkNearestAirportAction = builder.Action("CheckNearestAirport");
            checkNearestAirportAction.Parameter<double>("lat");
            checkNearestAirportAction.Parameter<double>("lon");
The result is :

{% highlight xml %}
	
	<Action Name="CheckNearestAirport">
	  <Parameter Name="lat" Type="Edm.Double" Nullable="false"/>
	  <Parameter Name="lon" Type="Edm.Double" Nullable="false"/>
	</Action>
	
	<ActionImport Name="CheckNearestAirport" Action="ODataSamples.WebApiService.Models.CheckNearestAirport"/>
{% endhighlight %}

