
# How to use the Collection API / CollectionServiceClient

## Introduction

Before you can create Collections you need to create a Project in the AOS.

If you don't know how to create a Project you should read the previous chapter about the Project API basics: 
* [**How to Project**](03_How-To-Project.md)


## Create Collection

API example for creating a new Collection.
This request needs at least WRITE permission on the project.

### Bash:
```bash
# Native JSON request to create a collection
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


## Get Collection

API examples for fetching an existing Collection.
This request needs at least READ permissions on the Collection or the Project.

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


## Update Collection

API examples for updating a Collection.

> :warning: **A collection update overwrites all the fields in the request, even if they're empty.**

**Note:** You can pin a still unversioned Collection by providing an initial version in the update request. 
This effectively creates a copy of the collection with a stable version and with all its objects pinned to their explicit revision number.
Pinned collections can not be updated in place anymore.

If you try to update an already versioned Collection you have to provide a (semantically) greater version in the request.

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


## Pin Collection

API examples to pin a Collection, i.e. give it a fixed semantic version.

This effectively creates a copy of the collection with a stable version and with all its objects pinned to their explicit revision number.
Pinned collections can not be updated in place anymore.

### Bash:
```bash
# Native JSON request to update a collections name and description
curl -d '
  {
    "version": {
        "major": 0,
        "minor": 0,
        "patch": 0
      }
  }' \
  -H 'Authorization: Bearer <API_TOKEN>' \
  -H 'Content-Type: application/json' \
  -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/pin
```

## Delete Collection

API examples for deleting a Collection.

Collections can only be deleted under the following conditions:
* Collection has to be empty (all Objects have to be deleted/moved)
* User needs ADMIN permissions on the specific collection or the project

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
# Native JSON request to force delete a collection
curl -d '
  {
    "projectId": "<project-id>",
    "force": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>
```
