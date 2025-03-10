---
title: GraphQL - Conditional filters
sidebar_position: 4
image: og/docs/api.jpg
# tags: ['graphql', 'filters']
---

import Badges from '/_includes/badges.mdx';

<Badges/>

import TryEduDemo from '/_includes/try-on-edu-demo.mdx';

<TryEduDemo />

## Overview

Conditional filters can be added to queries on the class level. The operator used for filtering is also called a `where` filter.
<!--
import GraphQLFiltersExample from '/_includes/code/graphql.filters.example.mdx';

<GraphQLFiltersExample/> -->

## Single operand (condition)

Each set of algebraic conditions is called an "operand". For each operand, the required properties are:
- The GraphQL property path,
- The operator type, and
- The valueType with the value.

For example, this filter will only allow objects from the class `Article` with a `wordCount` that is `GreaterThan` than `1000`.

import GraphQLFiltersWhereSimple from '/_includes/code/graphql.filters.where.simple.mdx';

<GraphQLFiltersWhereSimple/>

<details>
  <summary>Expected response</summary>

```
{
  "data": {
    "Get": {
      "Article": [
        {
          "title": "Anywhere but Washington: an eye-opening journey in a deeply divided nation"
        },
        {
          "title": "The world is still struggling to implement meaningful climate policy"
        },
        ...
      ]
    }
  }
}
```

</details>

## Filter structure

Supported by the [`Get{}`](./get.md) and [`Aggregate{}`](./aggregate.md) functions.

The `where` filter is an [algebraic object](https://en.wikipedia.org/wiki/Algebraic_structure), which takes the following arguments:

- `Operator` (which takes one of the following values)
  - `And`
  - `Or`
  - `Equal`
  - `NotEqual`
  - `GreaterThan`
  - `GreaterThanEqual`
  - `LessThan`
  - `LessThanEqual`
  - `Like`
  - `WithinGeoRange`
  - `IsNull`
- `Operands`: Is a list of `Operator` objects of this same structure, only used if the parent `Operator` is set to `And` or `Or`.
- `Path`: Is a list of strings in [XPath](https://en.wikipedia.org/wiki/XPath#Abbreviated_syntax) style, indicating the property name of the class.
  If the property is a beacon (i.e., cross-reference), the path should be followed to the property of the beacon which should be specified as a list of strings. For a schema structure like:
 ```json
 {
   "inPublication": {
     "Publication": {
       "name": "Wired"
     }
   }
 }
 ```
  Here, the path selector for `name` will be `["inPublication", "Publication", "name"]`.

- `valueType`

### Example filter structure

```graphql
{
  Get {
    <Class>(where: {
        operator: <operator>,
        operands: [{
          path: [path],
          operator: <operator>
          <valueType>: <value>
        }, {
          path: [<matchPath>],
          operator: <operator>,
          <valueType>: <value>
        }]
      }) {
      <propertyWithBeacon> {
        <property>
        ... on <ClassOfWhereBeaconGoesTo> {
          <propertyOfClass>
        }
      }
    }
  }
}
```

### Available `valueType` values

- `valueInt`: The integer value that the last property in the `Path` selector should be compared to.
- `valueBoolean`: The boolean value that the last property in `Path` should be compared to.
- `valueString`: The string value that the last property in `Path` should be compared to. (Note: `string` has been deprecated.)
- `valueText`: The text value that the last property in `Path` should be compared to.
- `valueNumber`: The number (float) value that the last property in `Path` should be compared to.
- `valueDate`: The date (ISO 8601 timestamp, formatted as [RFC3339](https://datatracker.ietf.org/doc/rfc3339/)) value that the last property  in `Path` should be compared to.

### Filter behavior of multi-word queries in `Equal` operator

The behavior for the `Equal` operator on multi-word textual properties in `where` filters depends on the `tokenization` of the property.

Refer to [this section](../../config-refs/schema.md#property-tokenization) on the difference between the two types.

### Stopwords in `text`/`string` filter values

Starting with `v1.12.0` you can configure your own [stopword lists for the inverted index](/developers/weaviate/config-refs/schema.md#invertedindexconfig--stopwords-stopword-lists).

### Example response

```json
{
  "data": {
    "Get": {
      "Article": [
        {
          "title": "Opinion | John Lennon Told Them ‘Girls Don't Play Guitar.' He Was So Wrong."
        }
      ]
    }
  },
  "errors": null
}
```

## Filter by id

You can filter object by their unique id or uuid, where you give the `id` as `valueText`.

import GraphQLFiltersWhereId from '/_includes/code/graphql.filters.where.id.mdx';

<GraphQLFiltersWhereId/>

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Article": [
        {
          "title": "Backs on the rack - Vast sums are wasted on treatments for back pain that make it worse"
        }
      ]
    }
  }
}
```

</details>

## Filter by timestamps

Filtering can be performed with internal timestamps as well, such as `creationTimeUnix` and `lastUpdateTimeUnix`. These values can be represented either as Unix epoch milliseconds, or as [RFC3339](https://datatracker.ietf.org/doc/rfc3339/) formatted datetimes. Note that epoch milliseconds should be passed in as a `valueText`, and an RFC3339 datetime should be a `valueDate`.

:::info
Filtering by timestamp requires the target class to be configured to index  timestamps. See [here](/developers/weaviate/config-refs/schema.md#invertedindexconfig--indextimestamps) for details.
:::

import GraphQLFiltersWhereTimestamps from '/_includes/code/graphql.filters.where.timestamps.mdx';

<GraphQLFiltersWhereTimestamps />

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Article": [
        {
          "title": "Army builds new body armor 14-times stronger in the face of enemy fire"
        },
        ...
      ]
    }
  }
}
```

</details>

## Filter by property length

Filtering can be performed with the length of properties.

The length of properties is calculated differently depending on the type:
- array types: the number of entries in the array is used, where null (property not present) and empty arrays both have the length 0.
- strings and texts: the number of characters, unicode characters such as 世 count as one character.
- numbers, booleans, geo-coordinates, phone-numbers and data-blobs are not supported.

```graphql
{
  Get {
    <Class>(where: {
        operator: <Operator>,
        valueInt: <value>
        path: [len(<property>)]
  }
}
```
Supported operators are `(not) equal` and `greater/less than (equal)` and values need to be 0 or larger.

:::note
Filtering by property length requires the target class to be configured to index the length. See [here](/developers/weaviate/config-refs/schema.md#invertedindexconfig--indexpropertylength) for details.
:::


## Multiple operands

You can set multiple operands by providing an array.

For example, these filters select based on the class Article with a wordCount higher than 1000 and who are published before January 1st 2020.

:::tip
You can filter datetimes similarly to numbers, with the `valueDate` given as `string` in [RFC3339](https://datatracker.ietf.org/doc/rfc3339/) format.
:::

import GraphQLFiltersWhereOperands from '/_includes/code/graphql.filters.where.operands.mdx';

<GraphQLFiltersWhereOperands />

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Article": [
        {
          "title": "China\u2019s long-distance lorry drivers are unsung heroes of its economy"
        },
        {
          "title": "\u2018It\u2019s as if there\u2019s no Covid\u2019: Nepal defies pandemic amid a broken economy"
        },
        {
          "title": "A tax hike threatens the health of Japan\u2019s economy"
        }
      ]
    }
  }
}
```

</details>

## Like operator

Using the `Like` operator allows you to do string searches based on partial match. The capabilities of this operator are:

- `?` -> exactly one unknown character
  - `car?` matches `cart`, `care`, but not `car`
- `*` -> zero, one or more unknown characters
  - `car*` matches `car`, `care`, `carpet`, etc
  - `*car*` matches `car`, `healthcare`, etc.

import GraphQLFiltersWhereLike from '/_includes/code/graphql.filters.where.like.mdx';

<GraphQLFiltersWhereLike/>

### Notes
Each query using the `Like` operator iterates over the entire inverted index for that property. The search time will go up linearly with the dataset size. Be aware that there might be a point where this query is too expensive and will not work anymore. We will improve this implementation in a future release. You can leave feedback or feature requests in a [GitHub issue](https://github.com/weaviate/weaviate/issues).

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Publication": [
        {
          "name": "The New York Times Company"
        },
        {
          "name": "International New York Times"
        },
        {
          "name": "New York Times"
        },
        {
          "name": "New Yorker"
        }
      ]
    }
  }
}
```

</details>

## Beacon (reference) filters

You can also search for the value of the property of a beacon.

For example, these filters select based on the class Article but who have `inPublication` set to New Yorker.

import GraphQLFiltersWhereBeacon from '/_includes/code/graphql.filters.where.beacon.mdx';

<GraphQLFiltersWhereBeacon/>

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Article": [
        {
          "inPublication": [
            {
              "name": "New Yorker"
            }
          ],
          "title": "The Hidden Costs of Automated Thinking"
        },
        {
          "inPublication": [
            {
              "name": "New Yorker"
            }
          ],
          "title": "The Real Deal Behind the U.S.\u2013Iran Prisoner Swap"
        },
        ...
      ]
    }
  }
}
```

</details>

## Filter objects by count of reference

Above example shows how filter by reference can solve straightforward questions like "Find all articles that are published by New Yorker". But questions like "Find all articles that are written by authors that wrote at least two articles", cannot be answered by the above query structure. It is however possible to filter by reference count. To do so, simply provide one of the existing compare operators (`Equal`, `LessThan`, `LessThanEqual`, `GreaterThan`, `GreaterThanEqual`) and use it directly on the reference element. For example:

import GraphQLFiltersWhereBeaconCount from '/_includes/code/graphql.filters.where.beacon.count.mdx';

<GraphQLFiltersWhereBeaconCount/>

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Author": [
        {
          "name": "Agam Shah",
          "writesFor": [
            {
              "name": "Wall Street Journal"
            },
            {
              "name": "Wall Street Journal"
            }
          ]
        },
        {
          "name": "Costas Paris",
          "writesFor": [
            {
              "name": "Wall Street Journal"
            },
            {
              "name": "Wall Street Journal"
            }
          ]
        },
        ...
      ]
    }
  }
}
```

</details>

## GeoCoordinates filter

A special case of the `Where` filter is with geoCoordinates. This filter is only supported by the `Get{}` function. If you've set the `geoCoordinates` property type, you can search in an area based on kilometers.

For example, this curious returns all in a radius of 2KM around a specific geo-location:

import GraphQLFiltersWhereGeocoords from '/_includes/code/graphql.filters.where.geocoordinates.mdx';

<GraphQLFiltersWhereGeocoords/>

<details>
  <summary>Expected response</summary>

```json
{
  "data": {
    "Get": {
      "Publication": [
        {
          "headquartersGeoLocation": {
            "latitude": 51.512737,
            "longitude": -0.0962234
          },
          "name": "Financial Times"
        },
        {
          "headquartersGeoLocation": {
            "latitude": 51.512737,
            "longitude": -0.0962234
          },
          "name": "International New York Times"
        }
      ]
    }
  }
}
```

</details>

## Filter by null state

Using the `IsNull` operator allows you to do filter for objects where given properties are `null` or `not null`. Note that zero-length arrays and empty strings are equivalent to a null value.

```graphql
{
  Get {
    <Class>(where: {
        operator: IsNull,
        valueBoolean: <true/false>
        path: [<property>]
  }
}
```

:::note
Filtering by null-state requires the target class to be configured to index this. See [here](../../config-refs/schema.md#invertedindexconfig--indexnullstate) for details.
:::

## More Resources

import DocsMoreResources from '/_includes/more-resources-docs.md';

<DocsMoreResources />
