﻿# Creating and Deploying Indexes

**Indexes are used by the server to satisfy queries.** Whenever a user issues a query, RavenDB will use an existing index if it matches the query. If it doesn't, RavenDB will create a new one.

{INFO:Remember}

Indexes are created by issuing a query are called `dynamic` or `Auto` indexes, and can be easily identified. Their name starts with `Auto/` prefix.

Indexes created explicitly by user are called `static`.

{INFO/}

{PANEL:**Static indexes**}

There are a couple of ways to create a `static index` and send it to server. We can use [maintenance operations](../indexes/creating-and-deploying#using-maintenance-operations) or create a [custom class](../indexes/creating-and-deploying#using-abstractindexcreationtask). You can also [scan an assembly](../indexes/creating-and-deploying#using-assembly-scanner) and deploy all found indexes.

<hr />

### Using AbstractIndexCreationTask

If you are interested in having a **strongly-typed syntax** during index creation, or have an **ability to deploy indexes using assembly scanner**, **avoid hard-coding index names in every query**, then `AbstractIndexCreationTask` should be your choice. 

{NOTE We recommend creating and using indexes in this form due to its simplicity. There are many benefits and few disadvantages. /}

#### Naming Convention

There is only one naming convention: each `_` in the class name will be translated to `/` in the index name.

e.g.

In `Northwind` samples there is a index called `Orders/Totals`. To get such a index name, we need to create a class called `Orders_Totals`.

{CODE indexes_1@Indexes/Creating.cs /}

#### Sending to Server

There is not much use from an index if it is not deployed to the server. In order to do so, we need to create instance of our class that inherits from `AbstractIndexCreationTask` and use one of the deployment methods: `Execute` or `ExecuteAsync` for an asynchronous call.

{CODE indexes_2@Indexes/Creating.cs /}

{CODE indexes_3@Indexes/Creating.cs /}

{SAFE If index exists on the server and stored definition is the same as the one that was sent, then it will not be overwritten. This implies that indexed data will not be deleted and indexation will not start from scratch. /}

#### Using Assembly Scanner

All classes that inherit from `AbstractIndexCreationTask` can be deployed at once using one of `IndexCreation.CreateIndexes` method overloads.

{CODE indexes_4@Indexes/Creating.cs /}

Underneath, the `IndexCreation` will attempt to create all indexes in a single request. If it fails, then it will repeat the execution by calling the `Execute` method one-by-one for each of the found indexes in separate requests.

#### Example

{CODE indexes_8@Indexes/Creating.cs /}

<hr />

### Using Maintenance Operations

`PutIndexesOperation` maintenance operation (which API reference can be found [here](../client-api/commands/indexes/put)) can be used also to send index(es) to the server.

The benefit of this approach is that you can choose the name as you feel fit, and change various settings available in `IndexDefinition`. But you lose the ability to deploy using the assembly scanner. Also you will have to use string-based names of indexes when querying.

{CODE indexes_5@Indexes/Creating.cs /}

#### IndexDefinitionBuilder

`IndexDefinitionBuilder` is a very useful class that enables you to create `IndexDefinitions` using strongly-typed syntax with access to low-level settings not available when the `AbstractIndexCreationTask` approach is used.

{CODE indexes_6@Indexes/Creating.cs /}

#### Remarks

{INFO Maintenance Operations or `IndexDefinitionBuilder` approaches are not recommended and should be used only if you can't do it by inheriting from `AbstractIndexCreationTask`. /}

{PANEL/}

{PANEL:**Auto indexes**}

Auto-indexes are **created** when queries that do **not specify an index name** are executed and (after in-depth query analysis) **no matching auto index is found** on the server-side.

### Naming Convention

Auto-indexes can be recognized by the `Auto/` prefix in their name. Their name also contains name of a collection that was queried, and list of fields that were required to find valid query results.

For instance, issuing a query like this

{CODE-TABS}
{CODE-TAB:csharp:C# indexes_7@Indexes/Creating.cs /}
{CODE-TAB-BLOCK:csharp:RQL}
FROM Employees
WHERE FirstName = 'Robert' AND LastName = 'King'
{CODE-TAB-BLOCK/}
{CODE-TABS/}

will result in a creation of a index named `Auto/Employees/ByFirstNameAndLastName`.

### Auto Indexes and Indexing State

To reduce the server load, if auto-indexes are not queried for a certain amount of time defined in `Indexing.TimeToWaitBeforeMarkingAutoIndexAsIdleInMin` setting (30 minutes by default), then they will be marked as `Idle`. You can read more about the implications of marking index as `Idle` [here](../server/administration/index-administration#index-prioritization).

Setting this configuration option to a high value may result in performance degradation due to the possibility of having a high amount of unnecessary work that is all redundant and not needed by indexes to perform. This is _not_ a recommended configuration.

{PANEL/}

## Related Articles

- [What are indexes?](../indexes/what-are-indexes)
- [Indexing : Basics](../indexes/indexing-basics)
- [Querying : Basics](../indexes/querying/basics)
