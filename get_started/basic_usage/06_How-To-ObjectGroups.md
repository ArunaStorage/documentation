
# How to use the ObjectGroup API / ObjectGroupServiceClient

## Introduction

ObjectGroups are a secondary, and therefore optional, resource to organize Objects inside Collections.

ObjectGroups can be used to group objects that are closely related to each other into a logical unit and to describe them with additional metadata.


## Create ObjectGroup

API example for creating an ObjectGroup.

This request needs at least APPEND permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to create a new object group
curl -d '
  {
    "name": "DummyGroup",
    "description": "This is a dummy object group for testing purposes.",
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

### Rust:
```rust
// Create tonic/ArunaAPI request to create a new object group
let create_request = CreateObjectGroupRequest {
    name: "Rust-API-Test-ObjectGroup".to_string(),
    description: "This object group was created through the Rust API.".to_string(),
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
        key: "LabelKey".to_string(),
        value: "LabelValue".to_string(),
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


## Get ObjectGroup

Fetching information of an ObjectGroup only returns information of the ObjectGroup itself, not the containing Objects.

This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to fetch information about a specific object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>
```

```bash
# Native JSON request to fetch information about the first 20 revisions of an object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history
```

```bash
# Native JSON request to fetch information about the first 250 revisions of an object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history?pageRequest.pageSize=250
```

```bash
# Native JSON request to fetch information about the revisions 21-40 (i.e. next page) of an object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history?pageRequest.lastUuid=<last-received-object-group-id>
```

### Rust:
```rust
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

```rust
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

```rust
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

```rust
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


## Get all ObjectGroups of Collection

You can also fetch all ObjectGroups of a Collection at once.

This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to fetch information about the first 20 object groups of a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group
```

### Rust:
```rust
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


## Get ObjectGroup Objects

Information of the containing Objects have to be requested separately.
Analogous to the [Get Objects](05_How-To-Objects.md#get-objects) functionality, the number of returned objects can be customized/paginated.
There is also the possibility to only fetch the objects marked as metadata of the ObjectGroup.

This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to fetch information of the first 20 objects of an object group including meta objects
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects
```

```bash
# Native JSON request to fetch information of the first 250 objects of an object group including meta objects
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects?pageRequest.pageSize=250
```

```bash
# Native JSON request to fetch information only of meta objects of an object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects?metaOnly=true
```

### Rust:
```rust
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

```rust
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

```rust
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


## Update ObjectGroup

Updating an ObjectGroup itself always creates a new revision of the ObjectGroup.

This request needs at least APPEND permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

**Note:** Updating an Object which is part of the ObjectGroup also initiates the update process and creates a new revision of the ObjectGroup.

> :warning: **An object group update overwrites all the fields in the request, even if they're empty.**

### Bash:
```bash
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

```bash
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

### Rust:
```rust
// Create tonic/ArunaAPI request to update the description of an object group
let update_request = UpdateObjectGroupRequest {
    group_id: "<object-group-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    name: "Rust-API-Test-ObjectGroup".to_string(),
    description: "This object group was updated through the Rust API.".to_string(),
    object_ids: vec![
        "<object-id-001>".to_string(),
        "<object-id-002>".to_string(),
        "<object-id-003>".to_string(),
    ],
    meta_object_ids: vec!["<object-id-004>".to_string()],
    labels: vec![KeyValue {
        key: "LabelKey".to_string(),
        value: "LabelValue".to_string(),
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

```rust
// Create tonic/ArunaAPI request to update the description and objects contained in the object group
let update_request = UpdateObjectGroupRequest {
    group_id: "<object-group-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    name: "Rust-API-Test-ObjectGroup".to_string(),
    description: "This object group was updated through the Rust API.".to_string(),
    object_ids: vec![
        "<object-id-001>".to_string(),
        "<object-id-002>".to_string(),
        "<object-id-005>".to_string(),
    ],
    meta_object_ids: vec!["<object-id-004>".to_string()],
    labels: vec![KeyValue {
        key: "LabelKey".to_string(),
        value: "LabelValue".to_string(),
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


## Delete ObjectGroup

Deletion of an ObjectGroup only removes the ObjectGroup and its revisions.
The Objects included in the ObjectGroup are not affected by the deletion of the ObjectGroup.

* **Deleting last revision of ObjectGroup:**

Upon deletion the Labels, Hooks, object references of the revision and the ObjectGroup itself is permanently removed from the database.

* **ObjectGroup has more than one revision:**

Upon deletion the labels, hooks and object references of the specific revision are removed directly but the revision itself will retain with the name and description set to "DELETED".
Deleted ObjectGroups are excluded from the general methods which fetch multiple ObjectGroups except the id is specifically provided.

This request needs at least MODIFY permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

> :warning: **ObjectGroup revisions can not be restored even if the revision still exists as DELETED.**

### Bash:
```bash
# Native JSON request to delete ObjectGroup with all its revisions
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<object-group-id>
```

### Rust:
```rust
// Create tonic/ArunaAPI request to create a new object group
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
