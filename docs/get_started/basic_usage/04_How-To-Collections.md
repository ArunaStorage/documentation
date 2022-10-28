
# How to use the Collection API / CollectionServiceClient

## Introduction

Before you can create Collections you need to create a Project in the AOS.

If you don't know how to create a Project you should read the previous chapter about the Project API basics: 
* [**How to Project**](03_How-To-Project.md)


## Create Collection

API example for creating a new Collection.

This request requires at least MODIFY permission on the Project in which the Collection is to be created.

### Bash:
```bash
# Native JSON request to create a new collection
curl -d '
  {
    "name": "DummyCollection",
    "description": "Description of a dummy collection.",
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

### Rust:
```rust
// Create tonic/ArunaAPI request to create a new collection
let create_request = CreateNewCollectionRequest {
    name: "Rust-API-Test-Collection".to_string(),
    description: "This collection was created through the Rust API.".to_string(),
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


## Get Collection(s)

API examples for fetching one or multiple existing Collection/s.

This request needs at least READ permissions on the Collection or the Project under which the collection is registered.

### Bash:
```bash
# Native JSON request to fetch information of a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>
```

```bash
# Native JSON request to fetch multiple collections of a project
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET "https://<URL-to-AOS-instance-API-gateway>/v1/collections/<project-id>?labelOrIdFilter.ids=<collection-id-001>&labelOrIdFilter.ids=<collection-id-002>"
```

### Rust:
```rust
// Create tonic/ArunaAPI request fetch information of a collection
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

```rust
// Create tonic/ArunaAPI request fetch all collections of a project
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

```rust
// Create tonic/ArunaAPI request fetch multiple collections of a project filtered by their ids
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

```rust
// Create tonic/ArunaAPI request fetch multiple collections of a project filtered by label keys
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


## Update Collection

API example for updating a Collection.

> :warning: **A collection update overwrites all the fields in the request, even if they're empty.**

**Note:** You can pin a still unversioned Collection by providing an initial version in the update request. 
This effectively creates a copy of the collection with a stable version and with all its objects pinned to their explicit revision number.
Pinned collections can not be updated in place anymore.

If you try to update an already versioned Collection you have to provide a (semantically) greater version in the request.

This request needs at least MODIFY permissions on the Collection or the Project under which the collection is registered.

### Bash:
```bash
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

### Rust:
```rust
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


## Pin Collection

API examples to pin a Collection, i.e. give it a fixed semantic version.

This effectively creates a copy of the collection with a stable version and with all its objects pinned to their explicit revision number.
Pinned collections can not be updated in place anymore.

This request needs at least MODIFY permissions on the Collection or the Project under which the collection is registered.

### Bash:
```bash
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

### Rust:
```rust
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


## Delete Collection

API examples for deleting a Collection.

Collections can only be deleted under the following conditions:
* Collection has to be empty (all Objects have to be deleted/moved)

This request needs at least MODIFY permissions on the Collection or ADMIN permissions on the Project under which the collection is registered.

### Bash:
```bash
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

```bash
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

### Rust:
```rust
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

```rust
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
