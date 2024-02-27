
# How to use the Collection API / CollectionServiceClient

## Introduction

Collections are the second-level (and therefore optional) resource to organize stored data i.e. Objects in Projects. 
Before you can create Collections you need to create a Project in the AOS.

If you don't know how to create a Project you should read the previous chapter about the [**Project API basics**](04_How-To-Project.md).

## Create Collection

API example for creating a new Collection.

??? Abstract "Required permissions"

    This request requires at least APPEND permission on the Project in which the Collection is to be created.

!!! Info "Collection naming guidelines"

    * Collection and Dataset names are restricted to the safe characters specified in the [S3 object key naming guidelines](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html#object-key-guidelines){target=_blank}

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a simple Collection
    curl -d '
      {
        "name": "json-api-collection", 
        "description": "Created with JSON over HTTP.",
        "keyValues": [],
        "relations": [],
        "data_class": "DATA_CLASS_PUBLIC",
        "projectId": "<project-id>",
        "metadataLicenseTag": "CC-BY-4.0",
        "defaultDataLicenseTag": "CC-BY-4.0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/collections
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a Collection
    let request = CreateCollectionRequest {
        name: "rust-api-collection".to_string(),
        description: "Created with the gRPC Rust API client.".to_string(),
        key_values: vec![],
        relations: vec![],
        data_class: DataClass::Public as i32,
        metadata_license_tag: Some("CC-BY-4.0".to_string()),
        default_data_license_tag: Some("CC-BY-4.0".to_string()),
        parent: Some(Parent::ProjectId("<project-id>".to_string())),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.create_collection(request)
                                 .await
                                 .unwrap() 
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new Collection
    request = CreateCollectionRequest(
        name="python-api-project",
        description="Created with the gRPC Python API client.",
        key_values=[], 
        relations=[], 
        data_class=DataClass.DATA_CLASS_PUBLIC,
        project_id="<project-id>",
        metadata_license_tag="CC-BY-4.0",
        default_data_license_tag="CC-BY-4.0"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.CreateCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get Collection(s)

API examples of how to fetch information for one or multiple existing Collections.

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Collection or a parent Project

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a collection
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}
    ```

    ```bash linenums="1"
    # Native JSON request to fetch information of multiple Collections
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET 'https://<URL-to-AOS-instance-API-gateway>/v2/collections?collectionIds={collection-id-01}&collectionIds={collection-id-02}'
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a collection
    let request = GetCollectionRequest {
        collection_id: "<collection-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.get_collection(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of multiple Collections
    let request = GetCollectionsRequest {
        collection_ids: vec![
            "<collection-id-01>".to_string(),
            "<collection-id-02>".to_string(),
            "<...>".to_string(),
        ],
    };

    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.get_collections(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of a Collection
    request = GetCollectionRequest(
        collection_id="<collection-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.GetCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of multiple Collections
    request = GetCollectionsRequest(
        collection_ids=[
            "<collection-id-01>",
            "<collection-id-02>",
            "<...>"]
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.GetCollections(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Update Collection

API examples of how to update individual metadata of an existing Collection.

??? Abstract "Required permissions"

    * Name update needs at least WRITE permissions on the specific Collection or a parent Project
    * Description update needs at least WRITE permissions on the specific Collection or a parent Project
    * KeyValue update needs at least WRITE permissions on the specific Collection or a parent Project
    * Dataclass update needs at least WRITE permissions on the specific Collection or a parent Project
    * License update needs at least WRITE permissions on the specific Collection or a parent Project

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to update the name of a Collection
    curl -d '
      {
        "name": "updated-json-api-collection"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}/name
    ```

    ```bash linenums="1"
    # Native JSON request to update the description of a Collection
    curl -d '
      {
        "description": "Updated with JSON over HTTP."
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}/description
    ```

    ```bash linenums="1"
    # Native JSON request to update the key-values associated with a Collection
    curl -d '
      {
        "addKeyValues": [],
        "removeKeyValues": []
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}/key_values
    ```

    !!! Info

        Dataclass can only be relaxed: `Confidential` > `Workspace` > `Private` > `Public`

    ```bash linenums="1"
    # Native JSON request to update the dataclass of a Collection
    curl -d '
      {
        "dataClass": "DATA_CLASS_PUBLIC"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}/data_class
    ```

    ```bash linenums="1"
    # Native JSON request to update the license of a Collection
    curl -d '
      {
        "metadataLicenseTag": "CC0",
        "defaultDataLicenseTag": "CC0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}/licenses
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the name of a Collection
    let request = UpdateCollectionNameRequest {
        collection_id: "<collection-id>".to_string(),
        name: "updated-rust-api-collection".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.update_collection_name(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the description of a Collection
    let request = UpdateCollectionDescriptionRequest {
        collection_id: "<collection-id>".to_string(),
        description: "Updated with the gRPC Rust API client.".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.update_collection_description(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the key-values associated with a Collection
    let request = UpdateCollectionKeyValuesRequest {
        collection_id: "<collection-id>".to_string(),
        add_key_values: vec![], 
        remove_key_values: vec![]
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.update_collection_key_values(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    !!! Info

        Dataclass can only be relaxed: `Confidential` > `Private` > `Public`

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the datacalass of a Collection
    let request = UpdateCollectionDataClassRequest {
        collection_id: "<collection-id>".to_string(),
        data_class: DataClass::Public as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.update_collection_data_class(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the license of a Collection
    let request = UpdateCollectionLicensesRequest {
        collection_id: "<collection-id>".to_string(),
        metadata_license_tag: "CC0".to_string(),
        default_data_license_tag: "CC0".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.update_collection_licenses(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the name of a Collection
    request = UpdateCollectionNameRequest(
        collection_id="<collection-id>",
        name="updated-python-api-project"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.UpdateCollectionName(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the description of a Collection
    request = UpdateCollectionDescriptionRequest(
        collection_id="<collection-id>",
        description="Updated with the gRPC Python API client"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.UpdateCollectionDescription(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    !!! Warning

        Removal of KeyValues always triggers the creation of a new object revision.

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the key-values associated with a Collection
    request = UpdateCollectionKeyValuesRequest(
        collection_id="<collection-id>",
        add_key_values=[],
        remove_key_values=[]
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.UpdateCollectionKeyValues(request=request)
    
    # Do something with the response
    print(f'{response}')
    ``` 

    !!! Info

        Dataclass can only be relaxed: `Confidential` > `Private` > `Public`

    ```python linenums="1"
    # Create tonic/ArunaAPI request to relax the data_class of a Collection
    request = UpdateCollectionDescriptionRequest(
        collection_id="<collection-id>",
        data_class=DataClass.DATA_CLASS_PUBLIC
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.UpdateCollectionDataClass(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the licenses of a Collection
    request = UpdateCollectionLicensesRequest(
        collection_id="<collection-id>",
        metadata_license_tag="CC0",
        default_data_license_tag="CC0"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.UpdateCollectionLicenses(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Snapshot Collection

API examples of how to snapshot a Collection, i.e. create an immutable clone of the Collection and its underlying resources.

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the Collection or the Project under which the collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request request to snapshot a Collection
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
      -H 'Content-Type: application/json' \
      -X POST https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}/snapshot
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to snapshot a Collection
    let request = SnapshotCollectionRequest {
        collection_id: "<collection-id>".to_string()
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.snapshot_collection_version(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to snapshot a Collection
    request = SnapshotCollectionRequest(
        collection_id="<collection-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.SnapshotCollectionVersion(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Delete Collection

API examples of how to delete a Collection.

!!! Info

    Deletion does not remove the Collection from the database, but sets the status of the Collection and the underlying resources to "DELETED".

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the Collection or the Project under which the collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a Collection
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/collections/{collection-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a Collection
    let request = DeleteCollectionRequest {
        collection_id: "<collection-id>".to_string()
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.delete_collection(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a collection
    request = DeleteCollectionRequest(
        collection_id="<collection-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.DeleteCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```
