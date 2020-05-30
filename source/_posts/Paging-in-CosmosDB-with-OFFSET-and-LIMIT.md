---
title: Paging in CosmosDB with OFFSET and LIMIT
date: 2020-05-30 12:14:24
---

When searching for solutions for paging documents in CosmosDB using the default API provider (SQL API), [continuation tokens](https://docs.microsoft.com/en-us/samples/azure-samples/azure-cosmosdb-java-pagination/achieving-cosmosdb-pagination-with-continuationtoken/) come up a lot. They seem great for features like an infinite scroll where the user only wants to page forward and never back. However, for the tools I'm building, the user needs to page forward *and* back.

There are ways to build systems that use continuation tokens and allow users to page backward. For example, caching the results either on the server or the client and referencing that cache when the user requests a previous page. That seems like a nice place for hard bugs to hide.

In June of 2019, the Azure team released support for [OFFSET and LIMIT clauses in CosmosDB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-query-offset-limit). Now paging support can be built using the same skip and take paradigm that is so familiar.

Here's an example of a controller method that uses OFFSET and LIMIT to enable paging forward *and* backward:

```csharp
public async Task<QueryResponse> Get(int skip = 0, int take = 10)
{
  var query = QueryDefinition($"SELECT * FROM c ORDER BY c.date OFFSET {skip} LIMIT {take}");
  var iterator = _container.GetItemQueryIterator<Model>(query);

  var models = new List<Model>();
  while (iterator.HasMoreResults)
  {
    models.AddRange(await iterator.ReadNextAsync());
  }

  return new QueryResponse
  {
    Models = models
  };
}
```

In the above code, `_container` is an instance of `Microsoft.Azure.Cosmos.Container`. `QueryResponse` and `Model` are custom classes that make up the response contract of the API.
