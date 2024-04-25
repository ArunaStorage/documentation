
# How to use the Search API / SearchServiceClient

## Introduction

In the FAIR context, the discoverability of data plays one of the most important roles, as even the best data will simply lie unused without a way to be found. Aruna provides a public search index in which specific metadata of all public objects can be searched. The user-defined scope of the search can be adjusted from fuzzy to detailed. A normal search query includes all fields but can be additionally filtered.

Depending on the type of field, different operators are useful for filtering:

| Field               | Type   | Examples               |
|---------------------|--------|------------------------|
| *name*              | String | `name != genome.fasta` |
| *description*       | String | `description != ''`    |
| *object_type*       | Enum   | `object_type = PROJECT`, `object_type IN [COLLECTION, DATASET]` |
| *object_type_id*    | Int    | `object_type_id = 1`, `object_type_id IN [2,3]`, `object_type_id > 1` |
| *count*             | Int    | `count > 1`, `count 1234 TO 123456` |
| *size*              | Int    | `size > 123456`, `size 1234 TO 123456` |
| *status*            | Enum   | `status = AVAILABLE`, `status NOT DELETED` |
| *labels*            | Struct | `labels.variant = LABEL AND labels.key = validated AND labels.value = true`,<br/>`labels.variant = HOOK AND labels.key = my_validator` |
| *data_class*        | Enum   | `data_class = PUBLIC` |
| *created_at*        | Int    | `created_at > 1698238293`, `created_at 1696111200 TO 1698793200` |
| *metadata_license*  | String | `metadata_license = CC0`, `metadata_license IN [CC0, CC-BY-SA-4.0, MIT]` |
| *data_license*      | String | `data_license = CC0`, `data_license IN [CC0, CC-BY-SA-4.0, MIT]` |

<!--
<div class="annotate" markdown>

* **Name:** (String)  
`name != genome.fasta`
* **Description:** (String)  
`description != ''`
* **Type:** (Enum)  
`type = OBJECT`, `type IN [PROJECT, COLLECTION]` (1)
* **Size:** (Int)  
`size > 123456`, `size 1234 TO 123456` (2)
* **Status:** (Enum)  
`type = AVAILABLE` (3)
* **Key-values:** (Struct)  
`labels.variant = LABEL AND labels.key = validated AND labels.value = true`, `labels.variant = HOOK and labels.key = my_validator` (4)
* **Data class:** (Enum)  
`data_class = PUBLIC` (5)
* **Creation time:** (Int)  
`created_at > 1698238293 `(6)
* **Metadata license:** (String/Enum)  
`metadata_license = CC0`, `metadata_license IN [CC0, CC-BY-SA-4.0, MIT]` (7)
* **Data license:** (String/Enum)  
`data_license = CC0`, `data_license IN [CC0, CC-BY-SA-4.0, MIT]`

</div>

1.  `Type` can be one of:
    * PROJECT
    * COLLECTION
    * DATASET
    * OBJECT  
2.  * Size of an Object is displayed in bytes
    * `TO` operator is equivalent to `>= AND <=`
3.  `Status` can be one of:
    * INITIALIZING
    * VALIDATING
    * AVAILABLE
    * UNAVAILABLE
    * ERROR
    * DELETED
4.  `Key-values` represent a struct with the following attributes:
    * `key`: Key of the key-value pair
    * `value`: Value of the key-value pair
    * `variant`: Can be one of `LABEL`, `HOOK`, `STATIC_LABEL` or `HOOK_STATUS`
5.  `Data_class` can be one of:
    * PUBLIC
    * PRIVATE
6.  `created_at` timestamp is transformed to UNIX timestamp for easier filtering with the comparison operators `>, <, >=, <=, TO`
7.  The predefined licenses can be found in the [:material-creative-commons: License](03_How-To-Licenses.md#introduction){target=_blank} section. These are always available, but this list is not exhaustive as it can be dynamically extended.
-->

These filters can be combined with the `AND` and `OR` expressions and nested with brackets `(...)` for complex queries.  
An in-depth documentation to the filter operator usage can be found in the Meilisearch [:simple-meilisearch:Filter expressions](https://www.meilisearch.com/docs/learn/fine_tuning_results/filtering#filter-expressions){target="_blank"} documentation.

If a user finds an interesting resource but does not have enough access permissions, an access request with a personal message can be sent to the creator of the resource which will be delivered via a [personal notification](09_How-To-EventNotification.md#persistent-personal-notifications){target=_blank}. This can be used to contact the user who created the Object on the first hand.


## Search resources

API examples of how to search resources.

The `limit` and `offset` parameter can be used to paginate the requests. 

* `limit` defines how many hits are returned and must be between 1 and 100 (inclusive).
* `offset` parameter defines how many hits are ignored before the next batch of hits are returned.

??? Abstract "Required permissions"

    This request does not require any permissions or authentication.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fuzzy search for a keyword
    curl '
      {
        "query": "fasta",
        "filter": "",
        "limit": "100",
        "offset": "0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/search
    ```

    ```bash linenums="1"
    # Native JSON request to specifically search for a keyword
    curl '
      {
        "query": "\"genome.fasta\"",
        "filter": "",
        "limit": "100",
        "offset": "0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/search
    ```

    ```bash linenums="1"
    # Native JSON request to fuzzy search for a keyword with additional filters
    curl '
      {
        "query": "fasta",
        "filter": "type = OBJECT",
        "limit": "100",
        "offset": "0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/search
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fuzzy search for a keyword
    let request = SearchResourcesRequest {
        query: "fasta".to_string(),
        filter: "".to_string(),
        limit: 100,
        offset: 0,
    };
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = search_client.search_resources(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to specifically search for a keyword
    let request = SearchResourcesRequest {
        query: "\"genome.fasta\"".to_string(),
        filter: "".to_string(),
        limit: 100,
        offset: 0,
    };
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = search_client.search_resources(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fuzzy search for a keyword with additional filters
    let request = SearchResourcesRequest {
        query: "fasta".to_string(),
        filter: "type = OBJECT".to_string(),
        limit: 100,
        offset: 0,
    };
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = search_client.search_resources(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fuzzy search for a keyword
    request = SearchResourcesRequest(
        query="fasta",
        filter="",
        limit=100,
        offset=0
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.search_client.SearchResources(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to specifically search for a keyword
    request = SearchResourcesRequest(
        query="\"genome.fasta\"",
        filter="",
        limit=100,
        offset=0
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.search_client.SearchResources(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fuzzy search for a keyword with additional filters
    request = SearchResourcesRequest(
        query="fasta",
        filter="type = OBJECT",
        limit=100,
        offset=0
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.search_client.SearchResources(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get public resource(s)

API examples of how to fetch Object information of public resources.

??? Abstract "Required permissions"

    This request does not require any permissions or authentication but the Object information will be returned redacted.

    If an authentication token is provided this request requires at least READ permissions on the specific resources.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a public Object
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/resource/{resource-id}
    ```

    ```bash linenums="1"
    # Native JSON request to fetch information of multiple public Objects
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET 'https://<URL-to-Aruna-instance-API-endpoint>/v2/resources?resourceIds=resource-id-01&resourceIds=resource-id-02'
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a public Object
    let request = GetResourceRequest {
        resource_id: "<resource-id>".to_string(),
    };
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = search_client.get_resource(request)
                                 .await
                                 .unwrap() 
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of multiple public Objects
    let request = GetResourcesRequest {
        resource_ids: vec![
          "<resource-id-01>".to_string(),
          "<resource-id-02>".to_string(),
          "<...>".to_string(),
        ],
    };
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = search_client.get_resources(request)
                                 .await
                                 .unwrap() 
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of a public Object
    request = GetResourceRequest(
        resource_id="<resource-id>"
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.search_client.GetResource(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of multiple public Objects
    request = GetResourceRequest(
        resource_ids=[
          "<resource-id-01>",
          "<resource-id-02>",
          "<...>"
        ]
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.search_client.GetResource(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Request access

API examples of how to request access for resources owned by other users.

??? Abstract "Required permissions"

    To request access to a resource you only have to be a registered Aruna user.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to request access to an Object of another user
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET 'https://<URL-to-Aruna-instance-API-endpoint>/v2/resource/resource-id/access?message=Would%20you%20please%20give%20me%20access%20to%20this%20dataset%3F'
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to request access to an Object of another user
    let request = RequestResourceAccessRequest {
        resource_id: "<resource-id>".to_string(),
        message: "Would you please give me access to this dataset?".to_string(),
    };
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = search_client.request_resource_access(request)
                                 .await
                                 .unwrap() 
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create Create tonic/ArunaAPI request to request access to an Object of another user
    request = RequestResourceAccessRequest(
        resource_id="<resource-id>",
        message="Would you please give me access to this dataset?"
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.search_client.RequestResourceAccess(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
