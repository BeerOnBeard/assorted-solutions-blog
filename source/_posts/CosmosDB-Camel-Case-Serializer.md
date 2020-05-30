---
title: CosmosDB Camel Case Serializer
date: 2020-05-30 11:46:31
---

Naming public properties in .NET is usually done using Pascal case. However, CosmosDB uses camel case for special properties on an object such as "id". So how do we square this circle?

Looking at the examples from the [Azure Samples Cosmos To Do App](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app), attributes seem to be the answer. As seen on the [Item model class](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app/blob/master/src/Models/Item.cs), the attribute `JsonProperty(PropertyName = "id")` is used. Adding that property to every class seems a bit laborious and prone to fat-finger syndrome.

In order to use those attributes, we also have to tie our solution to the `Newtonsoft.Json` package. Dotnet Core has a built-in JSON library in the `System.Text.Json` namespace. The `Microsoft.Azure.Cosmos` library [has plans to ditch Newtonsoft.Json and use System.Text.Json in the next major release of the library](https://github.com/Azure/azure-cosmos-dotnet-v3/issues/202). Adding these attributes all over the models seems like an even worse idea.

Using the following code snippet when creating the `CosmosClient` will serialize all properties using camel case without the need for peppering soon to be deprecated attributes all over the code:

```csharp
var options = new CosmosClientOptions
{
  SerializerOptions = new CosmosSerializationOptions
  {
     PropertyNamingPolicy = CosmosPropertyNamingPolicy.CamelCase
  }
};

var client = new CosmosClient(account, key, options);
```
