
# How to use the ObjectGroup API / ObjectGroupServiceClient

## Introduction

ObjectGroups are a secondary, and therefore optional, resource to organize Objects inside Collections.

ObjectGroups can be used to group objects that are closely related to each other into a logical unit and to describe them with additional metadata.


## Create ObjectGroup

API example for creating an ObjectGroup.

This request needs at least APPEND permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

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


## Get ObjectGroup

Fetching information of an ObjectGroup only returns information of the ObjectGroup itself, not the containing Objects.

This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

```bash
# Native JSON request to fetch information on specific object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>
```

```bash
# Native JSON request to fetch information on all revisions of an object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/history
```


## Get all ObjectGroups of Collection

You can also fetch all ObjectGroups of a Collection at once.

This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

```bash
# Native JSON request to fetch information on specific object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group
```


## Get ObjectGroup Objects

Information of the containing Objects have to be requested separately.
Analogous to the [Get Objects](05_How-To-Objects.md#get-objects) functionality, the number of returned objects as well as the filtering for specific IDs can be customized.

This request needs at least READ permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

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
# Native JSON request to fetch information of all objects in an object group matching one of the provided ids
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/group/<group-id>/objects?labelIdFilter.ids=<object-id-001>&labelIdFilter.ids=<object-id-002>
```

There is also the possibility to only fetch the objects marked as metadata of the ObjectGroup.

```bash
# Native JSON request to fetch information on all meta objects of an object group
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<group-id>/objects?metaOnly=true
```


## Update ObjectGroup

Updating an ObjectGroup itself always creates a new revision of the ObjectGroup.

This request needs at least APPEND permissions on the ObjectGroup's Collection or the Project under which the Collection is registered.

**Note:** Updating an Object which is part of the ObjectGroup also initiates the update process and creates a new revision of the ObjectGroup.

> :warning: **An object group update overwrites all the fields in the request, even if they're empty.**

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

```bash
# Native JSON request to delete ObjectGroup with all its revisions
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/group/<object-group-id>
```