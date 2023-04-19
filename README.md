# STAC API - Transaction Extension

*based on [**OGC API - Features - Part 4: Simple Transactions**](https://www.ogc.org/standards/ogcapi-features)*

- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance URIs:**
  - <https://api.stacspec.org/v1.0.0-rc.2/ogcapi-features/extensions/transaction>
  - <http://www.opengis.net/spec/ogcapi-features-4/1.0/conf/simpletx>
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/README.md#maturity-classification):** Candidate
- **Dependencies**:
  - [STAC API - Features](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0-rc.2/ogcapi-features/README.md)

The core STAC API doesn't support adding, editing, or removing items.
The transaction API extension supports the creation, editing, and deleting of items through POST, PUT, PATCH, and DELETE requests.

STAC Transactions are based on the [OGC API - Features](https://ogcapi.ogc.org/features/) transactions, as 
specified in [Part 4: Simple Transactions](http://docs.opengeospatial.org/DRAFTS/20-002.html). The core
OGC standard lays out the end points for transactions, without specifying any content types. For STAC we
use STAC Item objects in our transactions, and those transaction must be done at the OGC API - Features endpoints,
under `/collections/{collectionID}/items`. The OpenAPI document (specified as an OpenAPI fragment that 
gets build in the full STAC OpenAPI document) simply gives the STAC examples of using the
Simple Transactions API mechanism.

OGC API [Simple Transactions](http://docs.opengeospatial.org/DRAFTS/20-002.html) is still a draft standard, so 
once it is released STAC will align to a released one, but we anticipate few changes as it is a very simple document.

STAC Transactions additionally support optimistic locking through use of the ETag header, as specified in the
OpenAPI document. This is not currently specified in *OGC API - Features*, but it is compatible and we will 
work to get it incorporated.

## Methods

| Path                                                   | Content-Type Header | Body                                   | Success Status | Description                                                                  |
| ------------------------------------------------------ | ------------------- | -------------------------------------- | -------------- | ---------------------------------------------------------------------------- |
| `POST /collections/{collectionID}/items`               | `application/json`  | partial Item or partial ItemCollection | 201, 202       | Adds a new item to a collection.                                             |
| `PUT /collections/{collectionId}/items/{featureId}`    | `application/json`  | partial Item                           | 200, 202, 204  | Updates an existing item by ID using a complete item description.            |
| `PATCH /collections/{collectionId}/items/{featureId}`  | `application/json`  | partial Item                           | 200, 202, 204  | Updates an existing item by ID using a partial item description.             |
| `DELETE /collections/{collectionID}/items/{featureId}` | n/a                 | n/a                                    | 200, 202, 204  | Deletes an existing item by ID.                                              |
| `PATCH /collections/{collectionID}`                    | `application/json`  | partial Collection                     | 200, 202, 204  | Updates an existing collection by ID using a partial collection description. |
| `DELETE /collections/{collectionID}`                   | n/a                 | n/a                                    | 200, 202, 204  | Deletes an entire collection by ID and all related items                     |

### POST Item

When the body is a partial Item:

- Must only create a new resource.
- Must have an id field.
- Must return 409 if an Item exists for the same collection and id field values.
- Must populate the `collection` field in the Item from the URI.
- Must return 201 and a Location header with the URI of the newly added resource for a successful operation.
- May return the content of the newly added resource for a successful operation.

When the body is a partial ItemCollection:

- Must only create a new resource.
- Each Item in the ItemCollection must have an id field.
- Must return 409 if an Item exists for any of the same collection and id values.
- Must populate the `collection` field in each Item from the URI.
- Must return 201 without a Location header.
- May create only some of the Items in the ItemCollection. Implementations are not
  required to implement all-or-none sematics for this operation. For example, if an
  ItemCollection contains two Items and one is successfully created and the other
  fails to be created, the server is not required to then delete the successfully
  created one. When only some of the Items in the ItemCollection are created, the
  server should communicate this failure back to the client with an error status code.

All cases:

- Must return 202 if the operation is queued for asynchronous execution.

### PUT Item

- Must populate the `id` and `collection` fields in the Item from the URI.
- Must return 200 or 204 for a successful operation.
- If 200 status code is returned, the server shall return the content of the updated resource for a successful operation.
- Must return 202 if the operation is queued for asynchronous execution.
- Must return 404 if no Item exists for this resource URI.
- If the `id` or `collection` fields are different from those in the URI, status code 400 shall be returned.

### PATCH Item

- Must populate the `id` and `collection` fields in the Item from the URI.
- Must return 200 or 204 for a successful operation.
- If status code 200 is returned, the server shall return the content of the updated resource for a successful operation.
- May return the content of the updated resource for a successful operation.
- Must return 202 if the operation is queued for asynchronous execution.
- Must return 404 if no Item exists for this resource URI.
- If the `id` or `collection` fields are different from those in the URI, status code 400 shall be returned.

PATCH is compliant with [RFC 7386](https://tools.ietf.org/html/rfc7386).

### DELETE Item

- Must return 200 or 204 for a successful operation.
- Must return a 202 if the operation is queued for asynchronous execution.
- May return a 404 if no Item existed prior to the delete operation. Returning a 200 or 204 is also valid in this situation.

### PATCH Collection

- Must populate the `id` fields in the Item from the URI.
- Must return 200 or 204 for a successful operation.
- If status code 200 is returned, the server shall return the content of the updated resource for a successful operation.
- May return the content of the updated resource for a successful operation.
- Must return 202 if the operation is queued for asynchronous execution.
- Must return 404 if no Item exists for this resource URI.

PATCH is compliant with [RFC 7386](https://tools.ietf.org/html/rfc7386).

### DELETE Collection

- Must return 200 or 204 for a successful operation.
- Must return a 202 if the operation is queued for asynchronous execution.
- May return a 404 if no Item existed prior to the delete operation. Returning a 200 or 204 is also valid in this situation.
