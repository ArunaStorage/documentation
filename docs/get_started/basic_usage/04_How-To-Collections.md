
# How to use the Collection API / CollectionServiceClient

## Introduction

Collections are the basic resource to organize stored data i.e. Objects. 
Before you can create Collections you need to create a Project in the AOS.

If you don't know how to create a Project you should read the previous chapter about the [**Project API basics**](03_How-To-Project.md).


## Create Collection

API example for creating a new Collection.

!!! Info

    This request requires at least MODIFY permission on the Project in which the Collection is to be created.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new collection
    curl -d '
      {
        "name": "cURL-API-Test-Collection",
        "description": "This collection was created with a cURL request.",
        "projectId": "<project-id>",
        "labels": [
          {
            "key": "LabelKey",
            "value": "LabelValue"
          }
        ],
        "hooks": [
          {
            "key": "HookKey",
            "value": "HookValue"
          }
        ],
        "labelOntology": {
          "requiredLabelKeys": [
            "LabelKey"
          ]
        },
        "dataclass": "DATA_CLASS_PRIVATE"
      }' \
      -H 'Authorization: Bearer <API_TOKEN>' \
      -H 'Content-Type: application/json' \
      -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a new collection
    let create_request = CreateNewCollectionRequest {
        name: "Rust-API-Test-Collection".to_string(),
        description: "This collection was created with the gRPC Rust API client.".to_string(),
        project_id: "<project-id>".to_string(),
        labels: vec![KeyValue {
            key: "LabelKey".to_string(),
            value: "LabelValue".to_string(),
        }],
        hooks: vec![KeyValue {
            key: "HookKey".to_string(),
            value: "HookValue".to_string(),
        }],
        label_ontology: Some(LabelOntology {
            required_label_keys: vec!["LabelKey".to_string()],
        }),
        dataclass: DataClass::Private as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.create_new_collection(create_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new collection
    request = CreateNewCollectionRequest(
        name="Python-API-Test-Collection",
        description="This collection was created with the gRPC Python API client.",
        project_id="<project-id>",
        labels=[KeyValue(
            key="LabelKey",
            value="LabelValue"
        )],
        hooks=[KeyValue(
            key="HookKey",
            value="HookValue"
        )],
        label_ontology=LabelOntology(["LabelKey"]),
        dataclass=DataClass.Value("DATA_CLASS_PRIVATE")
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.CreateNewCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get Collection(s)

API examples for fetching one or multiple existing Collection/s.

!!! Info

    This request needs at least READ permissions on the Collection or the Project under which the collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a collection
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>
    ```
    
    ```bash linenums="1"
    # Native JSON request to fetch multiple collections of a project
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET "https://<URL-to-AOS-instance-API-gateway>/v1/collections/<project-id>?labelOrIdFilter.ids=<collection-id-001>&labelOrIdFilter.ids=<collection-id-002>"
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a collection
    let get_request = GetCollectionByIdRequest {
        collection_id: "<collection-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.get_collection_by_id(get_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch all collections of a project
    let get_request = GetCollectionsRequest {
        project_id: "<project-id>".to_string(),
        label_or_id_filter: None,
        page_request: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.get_collections(get_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch multiple collections of a project filtered by their ids
    let get_request = GetCollectionsRequest {
        project_id: "<project-id>".to_string(),
        label_or_id_filter: Some(LabelOrIdQuery {
            labels: None,
            ids: vec![
                "<collection-id-001".to_string(),
                "<collection-id-002".to_string(),
            ],
        }),
        page_request: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.get_collections(get_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch multiple collections of a project filtered by label keys
    let get_request = GetCollectionsRequest {
        project_id: "<project-id>".to_string(),
        label_or_id_filter: Some(LabelOrIdQuery {
            labels: Some(LabelFilter {
                labels: vec![KeyValue {
                    key: "LabelKey".to_string(),
                    value: "".to_string(),
                }],
                and_or_or: false,
                keys_only: true,
            }),
            ids: vec![],
        }),
        page_request: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.get_collections(get_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of a collection
    request = GetCollectionByIDRequest(
        collection_id="<collection-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.GetCollectionByID(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request fetch first 20 collections of a project
    request = GetCollectionsRequest(
        project_id="<project-id>",
        label_or_id_filter=None,  # Parameter can also be omitted if None
        page_request=None  # Parameter can also be omitted if None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.GetCollections(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch multiple collections of a project filtered by their ids
    GetCollectionsRequest(
        project_id="<project-id>",
        label_or_id_filter=LabelOrIDQuery(
            labels=None,  # Parameter can also be omitted if None
            ids=["<collection-id-001",
                 "<collection-id-002"]
        ),
        page_request=None  # Parameter can also be omitted if None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.GetCollections(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch multiple collections of a project filtered by label keys
    request = GetCollectionsRequest(
        project_id="<project-id>",
        label_or_id_filter=LabelOrIDQuery(
            labels=LabelFilter(
                labels=[KeyValue(key="LabelKey")],
                and_or_or=False,
                keys_only=True
            ),
            ids=None  # Parameter can also be omitted if None
        ),
        page_request=None  # Parameter can also be omitted if None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.GetCollections(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Update Collection

API example for updating a Collection.

!!! Warning

    **A collection update overwrites all the fields in the request, even if they're empty. 
    If you want to retain a field you have to explicitly set the old value.**

!!! Note 

    You can pin a still unversioned Collection by providing an initial version in the update request. 
    This effectively creates a copy of the collection with a stable version and with all its objects pinned to their explicit revision number.
    Pinned collections can not be updated in place anymore. If you try to update an already versioned Collection you have to provide a (semantically) greater version in the request.

!!! Info

    This request needs at least MODIFY permissions on the Collection or the Project under which the collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to update a collections name and description
    curl -d '
      {
        "name": "UpdatedDummyCollection",
        "description": "Updated description of a dummy collection.",
        "projectId": "<project-id>",
        "labels": [
          {
            "key": "LabelKey",
            "value": "LabelValue"
          }
        ],
        "hooks": [
          {
            "key": "HookKey",
            "value": "HookValue"
          }
        ],
        "labelOntology": {
          "requiredLabelKeys": [
            "LabelKey"
          ]
        },
        "dataclass": "DATA_CLASS_PRIVATE"
      }' \
      -H 'Authorization: Bearer <API_TOKEN>' \
      -H 'Content-Type: application/json' \
      -X PUT https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update a collections name and description
    let update_request = UpdateCollectionRequest {
        collection_id: "<collection-id>".to_string(),
        name: "Rust-API-Updated-Collection".to_string(),
        description: "This collection was updated through the Rust API.".to_string(),
        project_id: "<project-id>".to_string(),
        labels: vec![KeyValue {
            key: "LabelKey".to_string(),
            value: "LabelValue".to_string(),
        }],
        hooks: vec![KeyValue {
            key: "HookKey".to_string(),
            value: "HookValue".to_string(),
        }],
        label_ontology: Some(LabelOntology {
            required_label_keys: vec!["LabelKey".to_string()],
        }),
        dataclass: DataClass::Private as i32,
        version: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.update_collection(update_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update a collections name and description
    request = UpdateCollectionRequest(
        project_id="<project-id>",
        collection_id="<collection-id>",
        name="Python-API-Updated-Collection",
        description="This collection was updated with the gRPC Python API client.",
        labels=[KeyValue(
            key="LabelKey",
            value="LabelValue"
        )],
        hooks=[KeyValue(
            key="HookKey",
            value="HookValue"
        )],
        label_ontology=LabelOntology(["LabelKey"]),
        dataclass=DataClass.Value("DATA_CLASS_PRIVATE"),
        version=None  # Parameter can also be omitted if None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.UpdateCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Pin Collection

API examples to pin a Collection, i.e. give it a fixed semantic version.

This effectively creates a copy of the collection with a stable version and with all its objects pinned to their explicit revision number.
Pinned collections can not be updated in place anymore.

!!! Info

    This request needs at least MODIFY permissions on the Collection or the Project under which the collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to pin a collection to a specific version
    curl -d '
      {
        "version": {
            "major": 1,
            "minor": 2,
            "patch": 3
          }
      }' \
      -H 'Authorization: Bearer <API_TOKEN>' \
      -H 'Content-Type: application/json' \
      -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/pin
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to pin a collection to a specific version
    let pin_request = PinCollectionVersionRequest {
        collection_id: "<collection-id>".to_string(),
        version: Some(Version {
            major: 1,
            minor: 2,
            patch: 3,
        }),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.pin_collection_version(pin_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to pin a collection to a specific version
    request = PinCollectionVersionRequest(
        collection_id="<collection-id>",
        version=Version(
            major=1,
            minor=2,
            patch=3
        )
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.PinCollectionVersion(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Delete Collection

API examples for deleting a Collection.

!!! Note

    Before a collection can be deleted, all objects within the collection must have been deleted or moved to other collections. 
    In other words, the collection must be empty.

!!! Info

    This request needs at least MODIFY permissions on the Collection or ADMIN permissions on the Project under which the collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a collection
    curl -d '
      {
        "projectId": "<project-id>",
        "force": false
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>
    ```
    
    ```bash linenums="1"
    # Native JSON request to force delete a collection with force
    curl -d '
      {
        "projectId": "<project-id>",
        "force": true
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a collection
    let delete_request = DeleteCollectionRequest {
        collection_id: "<collection-id>".to_string(),
        project_id: "<project-id>".to_string(),
        force: false
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.delete_collection(delete_request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a collection with force
    let delete_request = DeleteCollectionRequest {
        collection_id: "<collection-id>".to_string(),
        project_id: "<project-id>".to_string(),
        force: true
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = collection_client.delete_collection(delete_request)
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
        collection_id="<collection-id>",
        project_id="<project-id>",
        force=False
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.DeleteCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a collection with force
    request = DeleteCollectionRequest(
        collection_id="<collection-id>",
        project_id="<project-id>",
        force=True
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.collection_client.DeleteCollection(request=request)

    # Do something with the response
    print(f'{response}')
    ```
