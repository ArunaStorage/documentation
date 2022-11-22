
# How to use the ObjectGroup API / ObjectGroupServiceClient

## Introduction

ObjectGroups are a secondary, and therefore optional, resource to organize Objects inside Collections.

ObjectGroups can be used to group objects that are closely related to each other into a logical unit and to describe them with additional metadata.


## Create ObjectGroup

API example for creating an ObjectGroup.

!!! Info

    This request needs at least APPEND permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new object group
    curl -d '
      {
        "name": "cURL-API-Test-ObjectGroup",
        "description": "This object group was created with a cURL request.",
        "objectIds": [
          "<object-id-001>",
          "<object-id-002>",
          "<object-id-003>"
        ],
        "metaObjectIds": [
          "<object-id-004>"
        ],
        "labels": [
          {
            "key": "isDummyGroup",
            "value": "true"
          }
        ],
        "hooks": []
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a new object group
    let create_request = CreateObjectGroupRequest {
        name: "Rust-API-Test-ObjectGroup".to_string(),
        description: "This object group was created with the gRPC Rust API client.".to_string(),
        collection_id: "<collection-id>".to_string(),
        object_ids: vec![
            "<object-id-001>".to_string(),
            "<object-id-002>".to_string(),
            "<object-id-003>".to_string(),
        ],
        meta_object_ids: vec![
            "<object-id-004>".to_string(),
        ],
        labels: vec![KeyValue {
            key: "isDummyGroup".to_string(),
            value: "true".to_string(),
        }],
        hooks: vec![],
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.create_object_group(create_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new object group
    request = CreateObjectGroupRequest(
        name="Python-API-Test-ObjectGroup",
        description="This object group was created with the gRPC Python API client.",
        collection_id="<collection-id>",
        object_ids=["<object-id-001>",
                    "<object-id-002>",
                    "<object-id-003>"],
        meta_object_ids=["<object-id-004>"],
        labels=[KeyValue(
            key="isDummyGroup",
            value="true"
        )],
        hooks=None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.CreateObjectGroup(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get ObjectGroup (and its revisions)

Fetching information of an ObjectGroup only returns information of the ObjectGroup itself, not the containing Objects.

!!! Info

    This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information about a specific object group
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>
    ```
    
    ```bash linenums="1"
    # Native JSON request to fetch information about the first 20 revisions of an object group
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history
    ```
    
    ```bash linenums="1"
    # Native JSON request to fetch information about the first 250 revisions of an object group
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history?pageRequest.pageSize=250
    ```
    
    ```bash linenums="1"
    # Native JSON request to fetch information about the revisions 21-40 (i.e. next page) of an object group
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history?pageRequest.lastUuid=<last-received-object-group-id>
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about a specific object group
    let get_request = GetObjectGroupByIdRequest { 
        group_id: "<object-group-id>".to_string(), 
        collection_id: "<collection-id>".to_string(), 
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_by_id(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about the first 20 revisions of an object group
    let get_request = GetObjectGroupHistoryRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        page_request: None
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_by_id(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about the first 250 revisions of an object group
    let get_request = GetObjectGroupHistoryRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        page_request: Some(PageRequest {
            last_uuid: "".to_string(),
            page_size: 250,
        }),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_by_id(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about the revisions 21-40 (i.e. next page) of an object group
    let get_request = GetObjectGroupHistoryRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        page_request: Some(PageRequest {
            last_uuid: "<last-received-object-group-id>".to_string(),
            page_size: 0,
        }),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_by_id(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about a specific object group
    request = GetObjectGroupByIdRequest(
        group_id="<object-group-id>",
        collection_id="<collection-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupById(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about the first 20 revisions of an object group
    request = GetObjectGroupHistoryRequest(
        collection_id="<collection-id>",
        group_id="<object-group-id>",
        page_request=None  # Parameter can also be omitted if None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupHistory(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about the first 250 revisions of an object group
    request = GetObjectGroupHistoryRequest(
        collection_id="<collection-id>",
        group_id="<object-group-id>",
        page_request=PageRequest(
            last_uuid="",  # Parameter can also be omitted if empty
            page_size=250
        )
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupHistory(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about the revisions 21-40 (i.e. next page) of an object group
    request = GetObjectGroupHistoryRequest(
        collection_id="<collection-id>",
        group_id="<object-group-id>",
        page_request=PageRequest(
            last_uuid="<last-received-object-group-id>",  
            page_size=0  # Parameter can also be omitted if <= 0 (Defaults to 20)
        )
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupHistory(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get ObjectGroups of Collection

You can also fetch multiple ObjectGroups of a Collection at once.

!!! Info

    This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information about the first 20 object groups of a collection
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/groups
    ```

    ```bash linenums="1"
    # Native JSON request to fetch information about the first 250 object groups of a collection
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/groups?pageRequest.pageSize=250
    ```

    ```bash linenums="1"
    # Native JSON request to fetch information about the object group 21-40 (i.e. next page) of an collection
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/groups?pageRequest.lastUuid=<last-received-object-group-id>
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about the first 20 object groups of a collection
    let get_request = GetObjectGroupsRequest {
        collection_id: "<collection-id>".to_string(),
        page_request: None,
        label_id_filter: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_groups(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about the first 250 object groups of a collection
    let get_request = GetObjectGroupsRequest {
        collection_id: "<collection-id>".to_string(),
        page_request: Some(PageRequest {
            last_uuid: "".to_string(),
            page_size: 250,
        }),
        label_id_filter: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_groups(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information about the object groups 21-40 (i.e. next page) of an collection
    let get_request = GetObjectGroupsRequest {
        collection_id: "<collection-id>".to_string(),
        page_request: Some(PageRequest {
            last_uuid: "<id-of-last-received-object-group>".to_string(),
            page_size: 0,
        }),
        label_id_filter: None,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_groups(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about the first 20 object groups of a collection
    request = GetObjectGroupsRequest(
        collection_id="<collection-id>",
        page_request=None,
        label_id_filter=None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroups(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about the first 250 object groups of a collection
    request = GetObjectGroupsRequest(
        collection_id="<collection-id>",
        page_request=PageRequest(
            last_uuid="",  # Parameter can also be omitted if empty
            page_size=250
        ),
        label_id_filter=None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroups(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information about the object groups 21-40 (i.e. next page) of an collection
    request = GetObjectGroupsRequest(
        collection_id="<collection-id>",
        page_request=PageRequest(
            last_uuid="<last-received-object-group-id>",  
            page_size=0  # Parameter can also be omitted if <= 0 (Defaults to 20)
        )
        label_id_filter=None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroups(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get ObjectGroup Objects

Information of the containing Objects have to be requested separately.
Analogous to the [Get Objects](05_How-To-Objects.md#get-objects) functionality, the number of returned objects can be customized/paginated.
There is also the possibility to only fetch the objects marked as metadata of the ObjectGroup.

!!! Info

    This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of the first 20 objects of an object group including meta objects
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects
    ```
    
    ```bash linenums="1"
    # Native JSON request to fetch information of the first 250 objects of an object group including meta objects
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects?pageRequest.pageSize=250
    ```
    
    ```bash linenums="1"
    # Native JSON request to fetch information only of meta objects of an object group
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects?metaOnly=true
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of the first 20 objects of an object group including meta objects
    let get_request = GetObjectGroupObjectsRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        page_request: None,
        meta_only: false
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_objects(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of the first 250 objects of an object group including meta objects
    let get_request = GetObjectGroupObjectsRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        page_request: Some(PageRequest {
            last_uuid: "".to_string(),
            page_size: 250
        }), 
        meta_only: false
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_objects(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information only of meta objects of an object group
    let get_request = GetObjectGroupObjectsRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        page_request: None, 
        meta_only: true
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.get_object_group_objects(get_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of the first 20 objects of an object group including meta objects
    request = GetObjectGroupObjectsRequest(
        collection_id="<collection-id>",
        group_id="<object-group-id>",
        page_request=None,
        meta_only=False
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupObjects(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of the first 250 objects of an object group including meta objects
    request = GetObjectGroupObjectsRequest(
        collection_id="<collection-id>",
        group_id="<object-group-id>",
        page_request=PageRequest(
            last_uuid="",
            page_size=250
        ),
        meta_only=False
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupObjects(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to to fetch information only of meta objects of an object group
    request = GetObjectGroupObjectsRequest(
        collection_id="<collection-id>",
        group_id="<object-group-id>",
        page_request=None,
        meta_only=True
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.GetObjectGroupObjects(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Update ObjectGroup

ObjectGroups can also be updated after creation. 

!!! Info

    This request needs at least MODIFY permissions on the Object's Collection or the Project under which the Collection is registered.

### Update which does not create a new revision

Just adding one or multiple labels to an Object does not create a new revision with this specific request.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to add a label to an object 
    curl -d '
      {
        "labelsToAdd": [
          {
            "key": "AnotherKey",
            "value": "AnotherValue"
         }
       ]
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection_id}/group/{group_id}/add_labels
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to add a label to an object group
    let add_request = AddLabelsToObjectGroupRequest {
        collection_id: "<collection-id>".to_string(),
        group_id: "<object-group-id>".to_string(),
        labels_to_add: vec![KeyValue {
            key: "AnotherKey".to_string(),
            value: "AnotherValue".to_string(),
        }],
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.add_labels_to_object_group(add_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to add a label to an object group
    request = AddLabelsToObjectGroupRequest(
        collection_id="<collection-id>",
        group_id="<object-id>",
        labels_to_add=[KeyValue(
            key="AnotherKey",
            value="AnotherValue"
        )]
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.AddLabelsToObjectGroup(request=request)

    # Do something with the response
    print(f'{response}')
    ```

### Update which creates a new revision

Otherwise, updating an ObjectGroup itself always creates a new revision of the ObjectGroup.

!!! Note 

    Updating an Object which is part of the ObjectGroup also initiates the update process and creates a new revision of the ObjectGroup.

!!! Info

    This request needs at least APPEND permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

!!! Warning

    **A object group update overwrites all the fields in the request, even if they're empty. 
    If you want to retain a field you have to explicitly set the old value.**

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to update the description of an object group
    curl -d '
      {
        "name": "DummyGroup",
        "description": "This is an updated description.",
        "objectIds": [
          "<object-id-001>",
          "<object-id-002>",
          "<object-id-003>"
        ],
        "metaObjectIds": [
          "<object-id-004>"
        ],
        "labels": [
          {
            "key": "isDummyGroup",
            "value": "true"
          }
        ],
        "hooks": []
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<object-group-id>
    ```

    ```bash linenums="1"
    # Native JSON request to update the description and objects contained in the object group
    curl -d '
      {
        "name": "DummyGroup",
        "description": "This is an updated updated description.",
        "objectIds": [
          "<object-id-002>",
          "<object-id-003>",
          "<object-id-005>"
        ],
        "metaObjectIds": [
          "<object-id-004>"
        ],
        "labels": [
          {
            "key": "isDummyGroup",
            "value": "true"
          }
        ],
        "hooks": []
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<object-group-id>
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the description of an object group
    let update_request = UpdateObjectGroupRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        name: "Rust-API-Test-ObjectGroup".to_string(),
        description: "This object group was updated with the gRPC Rust API client.".to_string(),
        object_ids: vec![
            "<object-id-001>".to_string(),
            "<object-id-002>".to_string(),
            "<object-id-003>".to_string(),
        ],
        meta_object_ids: vec!["<object-id-004>".to_string()],
        labels: vec![KeyValue {
            key: "isDummyGroup".to_string(),
            value: "true".to_string(),
        }],
        hooks: vec![],
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.update_object_group(update_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the description and objects contained in the object group
    let update_request = UpdateObjectGroupRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
        name: "Rust-API-Test-ObjectGroup".to_string(),
        description: "This object group was updated with the gRPC Rust API client.".to_string(),
        object_ids: vec![
            "<object-id-001>".to_string(),
            "<object-id-002>".to_string(),
            "<object-id-005>".to_string(),
        ],
        meta_object_ids: vec!["<object-id-004>".to_string()],
        labels: vec![KeyValue {
            key: "isDummyGroup".to_string(),
            value: "true".to_string(),
        }],
        hooks: vec![],
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.update_object_group(update_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the description of an object group
    request = UpdateObjectGroupRequest(
        group_id="<object-group-id>",
        name="Python-API-Test-ObjectGroup",
        description="This object group was updated with the gRPC Python API client.",
        collection_id="<collection-id>",
        object_ids=["<object-id-001>",
                    "<object-id-002>",
                    "<object-id-003>"],
        meta_object_ids=["<object-id-004>"],
        labels=[KeyValue(
            key="isDummyGroup",
            value="true"
        )],
        hooks=None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.UpdateObjectGroup(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the description of an object group
    request = UpdateObjectGroupRequest(
        group_id="<object-group-id>",
        name="Python-API-Test-ObjectGroup",
        description="This object group was updated with the gRPC Python API client.",
        collection_id="<collection-id>",
        object_ids=["<object-id-001>",
                    "<object-id-002>",
                    "<object-id-005>"],
        meta_object_ids=["<object-id-004>"],
        labels=[KeyValue(
            key="isDummyGroup",
            value="true"
        )],
        hooks=None
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.UpdateObjectGroup(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Delete ObjectGroup

Deletion of an ObjectGroup only removes the ObjectGroup and its revisions.
The Objects included in the ObjectGroup are not affected by the deletion of the ObjectGroup.

**Deleting last revision of ObjectGroup:**

: Upon deletion the Labels, Hooks, object references of the revision and the ObjectGroup itself is permanently removed from the database.

**ObjectGroup has more than one revision:**

: Upon deletion the labels, hooks and object references of the specific revision are removed directly but the revision itself will retain with the name and description set to "DELETED".
Deleted ObjectGroups are excluded from the general methods which fetch multiple ObjectGroups except the id is specifically provided.

!!! Info

    This request needs at least MODIFY permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

!!! Warning

    **ObjectGroup revisions can not be restored even if the revision still exists as DELETED.**

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete an ObjectGroup revision
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<object-group-id>
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete an ObjectGroup revision
    let delete_request = DeleteObjectGroupRequest {
        group_id: "<object-group-id>".to_string(),
        collection_id: "<collection-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = object_group_client.delete_object_group(delete_request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete an ObjectGroup revision
    request = DeleteObjectGroupRequest(
        group_id="<object-group-id>",
        collection_id="<collection-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.object_group_client.DeleteObjectGroup(request=request)

    # Do something with the response
    print(f'{response}')
    ```
