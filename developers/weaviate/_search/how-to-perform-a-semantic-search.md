---
title: (TBC) Vector/Semantic Search
sidebar_position: 2
image: og/docs/search.jpg
# tags: ['how to', 'perform a semantic search']
---

## Overview
<!-- TODO: Consider merging the whole doc into "Query basics" (prev. how-to-query-data) -->

Learn how to perform vector searches using Weaviate

<!-- TODO: Finish this page! -->
<!-- :::caution Under construction.
Migrated from "How to perform a semantic search" tutorial from Weaviate Docs Classic
::: -->

## Introduction

- Weaviate has RESTful API endpoints to query data, but Weaviate's query language is [GraphQL](https://graphql.org/).
- You can query a Weaviate after you've created a [schema](../tutorials/schema.md) and [populated it](/developers/weaviate/tutorials/import.md) with data.
- You can perform simple [`Get{}`](../api/graphql/get.md) queries to easily retrieve data, learn how [here](./how-to-query-data.md).
- To narrow down search results from a `Get{}` query based on semantics, use the `nearText` filter in the `Get{}` query. Read how in [this tutorial](#neartext-filter).
- To search and find for data objects in a fuzzy manner, you can use the GraphQL `Explore{}` function, read how in [this tutorial](#explore-graphql-function), and on the [reference page](../api/graphql/explore.md).

## Prerequisites
 1. **Connect to a Weaviate instance.**\\
 If you haven't set up a Weaviate instance yet, check the [Quickstart guide](/developers/weaviate/quickstart/index.md). In this guide we assume your instance is running at `http://localhost:8080` with [text2vec-contextionary](/developers/weaviate/modules/retriever-vectorizer-modules/text2vec-contextionary.md) as vectorization module.
 2. **Upload a schema**. \\
 Learn how to create and upload a schema [here](../tutorials/schema.md). In this guide we assume to have a similar schema uploaded with the classes `Publication`, `Article` and `Author`.
 3. **Add data**. \\
 Make sure there is data available in your Weaviate instance, you can read how to do this in the [previous guide](/developers/weaviate/tutorials/how-to-import-data.md). In this tutorial we assume there are data objects of `Publication`s, `Article`s and `Author`s present.

## nearText filter
If you want to perform semantic search on data objects in a known class, you can use the `nearText` filter in a `Get{}` query. Let's say we want to find `Publication`s that are related to "fashion", we can do the following query:

import HowtoSemanticSearchFilter from '/_includes/code/howto.semanticsearch.filter.mdx';

<HowtoSemanticSearchFilter/>

This will result in something like the following. Vogue was selected as only result, because there were no other publications related enough to "fashion" and "haute couture".

```json
{
  "data": {
    "Get": {
      "Things": {
        "Publication": [
          {
            "name": "Vogue"
          }
        ]
      }
    }
  },
  "errors": null
}
```

## Explore GraphQL function
If you are not sure what classes you want to query, or want to perform a fuzzy search through your whole dataset, then you can use the `Explore{}` function instead of the `Get{}` function. In the `Explore{}` function you don't specify which classes you perform the semantic search on, so the semantic search will be performed on all the data objects. Since this search is fuzzy, the only fields you can return are the `beacon`, `certainty`, `className`; you cannot request property values of the data objects, since the property value names depend on the data object, defined in the schema.

Let's search for data object about fashion again, but now we are not only interested in `Publication` data objects, but in all data objects that have something to do with "fashion".

import HowtoSemanticSearchFunction from '/_includes/code/howto.semanticsearch.function.mdx';

<HowtoSemanticSearchFunction/>

As you can see, the same arguments are applied in the "explore" filter and the `Explore{}` function. There is however no class specified. Instead, the `className` is returned as one of the GraphQL fields. The result of this query contains both data objects from the class `Publication` and `Article`:

```json
{
  "data": {
    "Explore": [
      {
        "beacon": "weaviate://localhost/65010df4-da64-333d-b1ce-55c3fc9174ab",
        "certainty": 0.8257073,
        "className": "Article"
      },
      {
        "beacon": "weaviate://localhost/ac884d35-ccb4-3937-81f8-8474a4d7a549",
        "certainty": 0.79948425,
        "className": "Publication"
      },
      {
        "beacon": "weaviate://localhost/f2b7c189-9183-3095-a5bb-b619d7fe9703",
        "certainty": 0.7862817,
        "className": "Article"
      },
      {
        "beacon": "weaviate://localhost/21239ca9-8f09-3747-b369-ff41e0dfebdd",
        "certainty": 0.7857753,
        "className": "Article"
      },
      {
        "beacon": "weaviate://localhost/8f2cb04e-c3bb-344f-8fbd-f742bf36e653",
        "certainty": 0.77738225,
        "className": "Article"
      }
    ]
  },
  "errors": null
}
```

If you are interested in property values of the returned objects, you will need to do a second query to to retrieve data of the beacon:

```bash
$ curl -s http://localhost:8080/v1/objects/{id}
```

So querying all property values of the first result can be done as follows:

```bash
$ curl -s http://localhost:8080/v1/objects/65010df4-da64-333d-b1ce-55c3fc9174ab
```

## Next steps

- Look for more ways to query your dataset in Weaviate with GraphQL queries, semantic search and other filters in the [GraphQL references guide](../api/graphql/index.md).
- Stay tuned for new tutorials, for example on interpretation of the semantic search results or how to set up a classification model!

## More Resources

import DocsMoreResources from '/_includes/more-resources-docs.md';

<DocsMoreResources />
