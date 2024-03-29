# relfield
This extension provides query-syntax and -semantics for inclusion and exclusion of fields for requested resource objects.

```
Note: 
JSON:API defines itself to be always backwards compatible.For this reason, any mentioning of the JSON:API in this document won't refer to a specific version.

The following examples show unencoded [ and ] and : characters in query strings simply for readability. In practice, these characters must be percent-encoded, per the requirements in RFC 3986.
```

## URI
This extension has the URI [`https://conjoon.org/json-api/ext/relfield`](https://conjoon.org/json-api/ext/relfield).

## Namespace 
This extension uses the namespace `relfield`.

```
Note: JSON:API extensions can only introduce new document 
members using a reserved namespace as a prefix.
```

## Example data for this document
This document uses the following data for demonstrating behavior specified by the JSON:API, and for demonstrating behavior introduced with this extension:

### `article` Entity (type: `article`)

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

## Sparse Fieldsets with the `relfied`-extension
This extension adds new semantics and syntax for querying fieldsets. It introduces a new character as a possible prefix for fields:

- `- (U+002D HYPHEN-MINUS, “-“)`

When it is used as the first character in front of a field name provided with the `relfield:fields[TYPE]`-parameter, it describes an **excluded** (`-`) field.

Any field name provided with the `relfield:fields[TYPE]`-parameter that does not carry this character is considered to be an **additional** field[^1].

This extension also introduces a wildcard character `* (U+002A ASTERISK, “*”)` for inclusion (`*`) of whole sets of fields of a resource. 

[^1]: Previous versions of this draft used the `+ (U+002B PLUS SIGN, “+”)` as a prefix for additional fields. This syntax was rejected to comply with already existing semantics regarding 'additional'/ 'excluded' fields. See [https://jsonapi.org/format/#fetching-sorting](https://jsonapi.org/format/#fetching-sorting).

### Additional fields

**Additional fields** are fields that are **added** to the list of **default fields** that should be included with the response object:

```http
GET /articles/1?relfield:fields[article]=version HTTP/1.1
Accept: application/vnd.api+json;ext=https://conjoon.org/json-api/ext/relfield
```

returns all the default fields, including the `version` field. 

### Excluded fields

**Excluded fields** are fields that should be **excluded** from the list of **default fields** returned with the response object and prefixed with `- (U+002D HYPHEN-MINUS, “-“)`: 

```http
GET /articles/1?relfield:fields[article]=-text,-teaser HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

includes the default fields for the `article` resource object, without the fields `text` and `teaser`. 

### Wildcard Queries
The `* (U+002A ASTERISK, “*”)`-character serves as a wildcard for requesting all accessible fields - **default** and **optional** - for a specific resource type and serves mainly for reducing visual complexity of otherwise large queries. 

```http
GET /articles/1?relfield:fields[article]=* HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

returns the resource object with **all** (default & optional) fields available.

Wildcards represent all fields of a resource object. Additional fields specified with the query and prefixed with a `- (U+002D HYPHEN-MINUS, “-“)` (or not prefixed at all) are computed relative against the list of **all** (`*`) fields of a resource object. Thus, including a wildcard with a `relfielf:fields[TYPE]` query makes sense if the client wants to explicitly exclude fields.

## Processing
There are no restrictions to the methods used with the HTTP requests sent with this extension.

If any operation in a request with this extension fails, the server **MUST** respond as described in the [base specification](https://jsonapi.org/) of the JSON:API. An array of one or more [error objects](https://jsonapi.org/format/#errors) **SHOULD** be returned, each with a source member that contains a pointer to the source of the problem in the request document.
 
### Redundancy of fields
There is **no** specific treatment for 
 - fields requested as **additional**, that are already part of the default list of fields of the resource object
 - fields requested as **excluded**, that are already missing from the default list of fields of the resource object


### Mutual exclusivity of `relfield:fields[TYPE]`- and `fields[TYPE]`-parameter
If a client sends a request containing both `relfield:fields[TYPE]`- and `fields[TYPE]`-parameter, where `TYPE` refers to the same type of resource object, the server **MUST** respond with a `400 Bad Request`.

If the client refers to different types of resource objects with `TYPE` and each fieldset specified with the query string, the server **MUST** treat the requested fieldsets as per their specifications.

### Error Handling
The response **SHOULD** include a document with a top-level error member that contains one or more error objects providing further details on the fields that caused the error. If a prefix for a field is missing, this field **SHOULD** appear in the error object.

```json
{
    "errors": [{
        "code": "123",
        "source": {"parameter": "relfield:fields[article]"},
        "title": "Bad Request",
        "detail": "Missing prefix for field 'version'."
    }]
}
```

#### Inaccessible fields

 - If the client requests an **additional** field that is not readable by the client, the server **SHOULD** respond with a `403 Forbidden`. The response **SHOULD** include a document with a top-level error member that contains one or more error objects providing further details on the inaccessible field.

- If the client issues a request that **excludes** a field that is usually not readable by the client, the server **SHOULD** treat the request as valid and proceed with its response according to its implementation. A `403 Forbidden` response **MAY** be sent for very restrictive systems.

## Examples

### `article` includes all default fields and the `version` field
```http
GET /articles/1?relfield:fields[article]=version HTTP/1.1
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

### Excluding `text` and `teaser` from the list of default fields of an `article`
```http
GET /articles/1?relfield:fields[article]=-text,-teaser HTTP/1.1
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

### Using sparse fieldsets with this extension for a different `TYPE`
```http
GET /articles/1?relfield:fields[article]=version&fields[comment]=author HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

In this example, the fields for `article` must be computed as per `relfield` specifications, and the fields for `comment` must be computed as per [base specifications](https://jsonapi.org/format/#fetching-sparse-fieldsets).

### `400 Bad Request`: Using sparse fieldsets with this extension for the same `TYPE`
```http
GET /articles/1?relfield:fields[article]=version&fields[article]=title HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**MUST** respond with
```http
HTTP/1.1 400 Bad Request
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

### `400 Bad Request`: Omitting prefixes with this extension for field values
```http
GET /articles/1?relfield:fields[article]=version,-title HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**MUST** respond with
```http
HTTP/1.1 400 Bad Request
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

### Requesting fields with inaccessible fields excluded
The client issues a request that **excludes** `secretfield`, which is not redable for the client.

```http
GET /articles/1?relfield:fields[article]=-secretfield HTTP/1.1
Accept: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```

**SHOULD** respond with 

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json;ext="https://conjoon.org/json-api/ext/relfield"
```


### `403 Bad Request`: Requesting inaccessible fields
The client requests `secretfield`, which is not readable for this client:

```http
GET /articles/1?relfield:fields[article]=secretfield HTTP/1.1
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

### Wildcard Queries

#### Requesting all fields for an `article`
```http
GET /articles/1?relfield:fields[article]=* HTTP/1.1
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

#### Requesting all fields for an `article`, excluding `version` and `teaser`
```http
GET /articles/1?relfield:fields[article]=*,-version,-teaser HTTP/1.1
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
            "text": "Lorem ipsum dolor sit amet, consectetuer adipiscing elit, [...]"
        }
    }
}
```

## Similar API implementations 
The [bitbucket](https://bitbucket.org/) API of Atlassian provides similar behavior with their use of the `fields paramter` with [partial responses](https://developer.atlassian.com/cloud/bitbucket/rest/intro/#partial-response).
