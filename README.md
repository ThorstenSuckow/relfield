# relfield

This extension provides additional query syntax for clients that use [Sparse Fieldsets](https://jsonapi.org/format/#fetching-sparse-fieldsets) with their requests.

It allows the client to inform the server about the fields that should be included with or excluded from a resource object.

```
Note: JSON:API defines itself to be always backwards compatible. 
For this reason, any mentioning of the JSON:API in this document
won't refer to a specific version.
```

## URI
This extension has the URI [`https://conjoon.org/json-api/ext/relfield`](https://conjoon.org/json-api/ext/relfield).

## Namespace 
This extension does not introduce new document members and therefor does not require any namespace.

```
Note: JSON:API extensions can only introduce new document 
members using a reserved namespace as a prefix.
```

## Example data for this document
This document uses the following data for demonstrating behavior specified by the JSON:API, and for demonstrating behavior introduced with this extension:

### `article` Value Object (type: `article`)

|  Field | Type | Description |   Default | Optional | 
|--------|------|--------------|----------|----------|
| `id` | `integer` | The unique id of the article. |   :heavy_check_mark: |  |
| `title` | `string`| The title for the article.|   :heavy_check_mark: |  |
| `author`|`string`|The name of the article's author.|   :heavy_check_mark: |  |
|`date`| `string`|The date the article was edited.|   :heavy_check_mark: |  |
| `teaser`| `string`| A teaser text for the article.|   :heavy_check_mark: |  |
|`text` | `string`| The complete text of the article.|   :heavy_check_mark: |  |
| `version` | `string` | The article's version. |   |  :heavy_check_mark: |
| `secretfield` | `string` | A field with a value that is a mystery to everyone. |   |  :heavy_check_mark: |

### Sample Data

```json
{
    "id": 1,
    "title": "Lorem ipsum",
    "author": "Jo Vongoe The",
    "date": "2022-06-25 18:00:00",
    "teaser": "Lorem ipsum dolor sit amet!"
    "text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, [...]",
    "version": "v1.0",
    "secretfield": "?"
}
```

### Endpoint

`articles/{articleId}`

Example for requesting an `article` with the id `1`:
```http
GET /articles/1 HTTP/1.1
Accept: application/vnd.api+json
```

#### Default fields for the `article` resource object
The endpoint defines the `default fields` returned with each `article`. If no `fields[article]` parameter was specified, the server guarantees to make the following fields available with a resource object of the type `article`:

 - `title`
 - `author`
 - `date`
 - `teaser`
 - `text`

#### Optional fields for the `article` resource object
The optional fields for the `article` resource object are 
 - `version`
 - `secretfield`
 
An `article` resource object will only have the default fields and optional fields included with a response if the client requests them explicitly by using the `fields[article]` parameter:

```http
GET /articles/1?fields[article]=title,author,date,teaser,text,version HTTP/1.1
Accept: application/vnd.api+json
```
gives the response

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
    "data": {
        "id": 1,
        "type": "article",
        "attributes": {
            "title": "Lorem ipsum",
            "author": "Jo Vongoe The",
            "date": "2022-06-25 18:00:00",
            "teaser": "Lorem ipsum dolor sit amet!"
            "text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, [...]",
            "version": "v1.0"
        }
    }
}
```

In this example,  `secretfield` is not readable by the client. Trying to access it would result in a `403 Forbidden` response: 

```http
GET /articles/1?fields[article]=secretfield HTTP/1.1
Accept: application/vnd.api+json
```
gives the response

```http
HTTP/1.1 403 Forbidden
Content-Type: application/vnd.api+json
```

## Sparse Fieldsets with the JSON:API as per specification

Clients **MAY** request that an endpoint includes only specific fields in the response in a per-type basis by including a `fields[TYPE]` parameter.

The following is specified:

1. If the client uses the `fields[TYPE]` parameter and leaves its value empty, the endpoint should not return any fields for resource objects with the given type in its response (please refer to [this discussion](https://github.com/json-api/json-api/pull/1636) for clarifications on the wording with this paragraph).

2. If a client requests a restricted set of fields for a given resource type, an endpoint **MUST NOT** include any additional fields in resource objects of that type in its response. 

3. If a client does not specify the set of fields for a given resource type, the server **MAY** send all fields, a subset of fields, or no fields for that resource type.

## Sparse fieldset with the `relfied`-extension
This extension expands on the syntax for the values used with sparse fieldsets and introduces two new characters as possible prefixes for fields:

- `+ (U+002B PLUS SIGN, “+”)`
- `- (U+002D HYPHEN-MINUS, “-“)`

Used as the first character in front of a field name provided with the `fields[TYPE]`-parameter, they describe **additional** (`+`) and **excluded** (`-`) fields.

### Additional fields

**Additional fields** are fields that are **added** to the list of **default fields** that should be included with the response object, and prefixed with `+ (U+002B PLUS SIGN, “+”)`:

```http
GET /articles/1?fields[article]=+version HTTP/1.1
Accept: application/vnd.api+json;ext=https://conjoon.org/json-api/ext/relfield
```

responds with

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"

{
    "data": {
        "id": 1,
        "type": "article",
        "attributes": {
            "title": "Lorem ipsum",
            "author": "Jo Vongoe The",
            "date": "2022-06-25 18:00:00",
            "teaser": "Lorem ipsum dolor sit amet!"
            "text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, [...]",
            "version": "v1.0"
        }
    }
}
```

If the `+`-character would be omitted in this example, the extension would respond in accordance with the base specification: 

```http
GET /articles/1?fields[article]=version HTTP/1.1
Accept: application/vnd.api+json;ext=https://conjoon.org/json-api/ext/relfield
```

responds with

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"

{
    "data": {
        "id": 1,
        "type": "article",
        "attributes": {
            "version": "v1.0"
        }
    }
}
```

### Excluded fields

**Excluded fields** are fields that should be **excluded** from the list of **default fields** returned with the response object and prefixed with `- (U+002D HYPHEN-MINUS, “-“)`: 

```http
GET /articles/1?fields[article]=-text,-teaser HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

responds with

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"

{
    "data": {
        "id": 1,
        "type": "article",
        "attributes": {
            "title": "Lorem ipsum",
            "author": "Jo Vongoe The",
            "date": "2022-06-25 18:00:00",
        }
    }
}
```

## Processing
There are no restrictions to the methods used with the HTTP requests sent with this extension.

If any operation in a request with this extension fails, the server **MUST** respond as described in the [base specification](https://jsonapi.org/) of the JSON:API. An array of one or more [error objects](https://jsonapi.org/format/#errors) **SHOULD** be returned, each with a source member that contains a pointer to the source of the problem in the request document.
 
### Redundancy of fields

There is **no** specific treatment for 
 - fields requested as **additional**, that are already part of the default list of fields of the resource object
 - fields requested as **excluded**, that are already missing from the default list of fields of the resource object

### Mutual exclusivity of `fields[TYPE]`-parameter syntax

 - If a client uses the additional prefixes for identifying fields as **additional** `+ (U+002B PLUS SIGN, “+”)` and/or **excluded** `- (U+002D HYPHEN-MINUS, “-“)` for the `fields[TYPE]` parameter, all fields appearing in the parameter's comma `(U+002C COMMA, “,”)` separated value list **MUST** be introduced with a `+`- or `-`-character. Omitting such a character for any field-identifier in a list where these prefixes appear **MUST** be responded with a `400 BAD REQUEST`: 

Example (using the syntax according to base specification with this extension):
```http
GET /articles/1?fields[article]=version,title HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

responds with

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

Example (using the syntax according to base specification mixed with syntax introduced with this extension):
```http
GET /articles/1?fields[article]=version,-title HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**MUST** respond with
```http
HTTP/1.1 400 Bad Request
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

The response **SHOULD** include a document with a top-level error member that contains one or more error objects providing further details on the fields that caused the error. If this extension is used, prefixed fields are given precedence, and only fields where the prefix is missing **SHOULD** appear in the error object.

```json
{
    "errors": [{
        "code": "123",
        "source": {"parameter": "fields[article]"},
        "title": "Bad Request",
        "detail": "Missing prefix for field version."
    }]
}
```

### Inaccessible fields

 - If the client requests an **additional** field that is not readable by the client, the server **SHOULD** respond with a `403 Forbidden`. The response **SHOULD** include a document with a top-level error member that contains one or more error objects providing further details on the inaccessible field.

Example (`secretfield` not readable by the client):
```http
GET /articles/1?fields[article]=+secretfield HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**SHOULD** respond with 

```http
HTTP/1.1 403 Forbidden
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

and **SHOULD** include
```json
{
    "errors": [{
        "code": "123",
        "source": {"pointer": "/data/attributes/secretfield"},
        "title": "Access forbidden",
        "detail": "secretfield may not be accessed."
    }]
}
```

- If the client issues a request that **excludes** a field that is usually not readable by the client, the server **SHOULD** treat the request as valid and proceed with its response according to its implementation. A `403 Forbidden` response **MAY** be sent for very restrictive systems.

Example (`secretfield` not readable by the client):
```http
GET /articles/1?fields[article]=-secretfield HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**SHOULD** respond with 

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

## Wildcard Queries

A server using this extension **SHOULD** suport the `* (U+002A ASTERISK, “*”)` character as a wildcard for requesting a resource object of a given type. The `*`-character serves as a wildcard for requesting all accessible fields - **default** and **optional** - for a specific resource type and serves mainly for reducing visual complexity of otherwise large queries.

```http
GET /articles/1?fields[article]=* HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**SHOULD** respond with

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"

{
    "data": {
        "id": 1,
        "type": "article",
        "attributes": {
            "title": "Lorem ipsum",
            "author": "Jo Vongoe The",
            "date": "2022-06-25 18:00:00",
            "teaser": "Lorem ipsum dolor sit amet!"
            "text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, [...]",
            "version": "1.0"
        }
    }
}
```

If the server does not support wildcard-queries with this extension, a request including a wildcard for the query-parameter `fields[TYPE]` must be responded with `400 Bad Request`:

```http
GET /articles/1?fields[article]=* HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

If wildcards are not supported, the server **MUST** respond with

```http
HTTP/1.1 400 Bad request
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

The response **SHOULD** include a document with a top-level error member that contains one or more error objects providing further details on the fields that caused the error. If this extension is used, but wildcards are not supported, an appropriate message **SHOULD** appear in the error object.

```json
{
    "errors": [{
        "code": "123",
        "source": {"parameter": "fields[article]"},
        "title": "Bad Request",
        "detail": "This server does not support wildcards for fieldsets."
    }]
}
```

Wildcards represent **all** fields of a resource object. Fields being prefixed with a `+ (U+002B PLUS SIGN, “+”)` and `- (U+002D HYPHEN-MINUS, “-“)` are computed relative against the list of **all** fields of a resource object. Thus, including a wildcard with a `fields[TYPE]` query makes sense if the client wants to explixitly exclude fields, since fields marked as **additional** are already represented by the wildcard:

```http
GET /articles/1 HTTP/1.1
Accept: application/vnd.api+json
```

gives the same response as 

```http
GET /articles/1?fields[article]=*,-version HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

And if the server supports wildcard with this extension, it **MUST** respond with

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"

{
    "data": {
        "id": 1,
        "type": "article",
        "attributes": {
            "title": "Lorem ipsum",
            "author": "Jo Vongoe The",
            "date": "2022-06-25 18:00:00",
            "teaser": "Lorem ipsum dolor sit amet!"
            "text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, [...]"
        }
    }
}
```
