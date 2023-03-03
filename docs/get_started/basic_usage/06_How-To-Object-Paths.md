
# How to use the Object Paths API

## Introduction

In order for the AOS to provide an S3 compatible interface, it is necessary that Objects can be accessed via one or more unique paths instead of their id.

Currently, these paths comply with the standardized specifications of AWS S3 and are represented in the format  
`<collection-version>.<collection-name>.<project-name>/<custom-path>/<object-filename>` which resembles the S3 path-style `s3://bucket/key` where:

* bucket: `<collection-version>.<collection-name>.<project-name>`
* key: `<custom-path>/<object-filename>`

<!--With the introduction of other AOS instances this will change to the S3 virtual-hosted-style which also includes the location in the URL `s3://bucket.location/key`.-->

When an object is initialized, a default path is automatically created if no custom path is specified. 
This also applies when creating a reference to another collection. An object is thus always accessible via at least one path in each of its collections.

!!! Warning

    **The fully qualified paths of objects are unique, which implies some conditions that must be met:**

    * Project and Collection names are restricted to the following characters: [a-z0-9\-] (i.e. alphanumeric lowercase and hyphens)
    * Project names are unique 
    * Collection names are unique within a project
    * Objects with equal filenames cannot have the same (custom) path within a collection


## Create Custom Object Path

API example for creating an additional path associated with an object.

!!! Note

    As specified in S3, you always get the latest version of an object via a path. 
    The request of a specific object version is currently not possible.

!!! Info

    This request needs at least APPEND permissions on the Object's Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a custom object path 
    curl -d '
      {
        "subPath": "/custom"
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object_id}/path
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create an additional custom path for an object
    let create_request = CreateObjectPathRequest {
        collection_id: "<collection-id>".to_string(),
        object_id: "<object-id>".to_string(),
        sub_path: "/custom".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_client.create_object_path(create_request)
                                .await
                                .unwrap()
                                .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to request an upload url for specific part of multipart upload
    request = CreateObjectPathRequest(
        object_id="<object-id>",
        collection_id="<collection-id>",
        sub_path="/custom"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_client.CreateObjectPath(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Set Object Path visibility

API example for modifying the visibility of an objects' path.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to modify the visibility of a path of an object
    curl -d '
      {
        "visibility": false
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection_id}/path/{path}/visibility
    ```

    To make the specific path visible again just use the parameter `"visibility": true`.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to modify the visibility of a path of an object
    let set_request = SetObjectPathVisibilityRequest {
        collection_id: "<collection-id>".to_string(),
        path: "<object-path>".to_string(),
        visibility: false,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_client.set_object_path_visibility(set_request)
                                .await
                                .unwrap()
                                .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    To make the specific path visible again just use the parameter `visibility: true`.

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to modify the visibility of a path of an object
    request = SetObjectPathVisibilityRequest(
        collection_id="<collection-id>",
        path="/<object-path>",
        visibility=True
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_client.SetObjectPathVisibility(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    To make the specific path visible again just use the parameter `visibility=True`.


## Get Object Paths

API example for fetching all paths associated with an object.

!!! Info

    This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get all object paths 
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object_id}/path?include_inactive=true
    ```

    ```bash linenums="1"
    # Native JSON request to get only active object paths 
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object_id}/path
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch all paths associated with an object
    let get_request = GetObjectPathRequest {
        collection_id: "<collection-id>".to_string(),
        object_id: "<object-id>".to_string(),
        include_inactive: true,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_client.get_object_path(get_request)
                                .await
                                .unwrap()
                                .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    To fetch only all of the active paths associated with an object just use the parameter `include_inactive: false`. 

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to to fetch all paths associated with an object
    request = GetObjectPathRequest(
        object_id="<object-id>",
        collection_id="<collection-id>",
        include_inactive=True
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_client.GetObjectPath(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    To fetch only all of the active paths associated with an object just use the parameter `include_inactive=False`.


## Get all Object Paths in Collection

API example for fetching all paths associated with the objects in a Collection.

!!! Info

    This request needs at least READ permissions on the Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get all object paths of collection
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/paths?include_inactive=true
    ```

    ```bash linenums="1"
    # Native JSON request to get only active object paths of collection
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/paths
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch all paths associated with objects in a collection
    let get_request = GetObjectPathsRequest {
        collection_id: "<collection-id>".to_string(),
        include_inactive: true,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_client.get_object_paths(get_request)
                                .await
                                .unwrap()
                                .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    To fetch only all of the active paths of a collection just use the parameter `include_inactive: false`. 

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to to fetch all paths associated with an object
    request = GetObjectPathsRequest(
        collection_id="<collection-id>",
        include_inactive=True
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_client.GetObjectPaths(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    To fetch only all of the active paths of a collection just use the parameter `include_inactiv=False`. 


## Get Object by Path

API example for fetching an Object associated with the specific path.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get object by any active path
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/path/{path}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch all paths associated with objects in a collection
    let get_request = GetObjectsByPathRequest {
        collection_id="<collection-id>",
        path: "".to_string(), // Default object path
        with_revisions: false, 
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_client.get_object_paths(get_request)
                                .await
                                .unwrap()
                                .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    To fetch all revisions of an Object associated with the specific path use the parameter `with_revisions: true`.

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to to fetch all paths associated with an object
    request = GetObjectsByPathRequest(
        collection_id="<collection-id>",
        with_revisions=False
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_client.GetObjectPaths(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    To fetch all revisions of an Object associated with the specific path use the parameter `with_revisions=True`.
