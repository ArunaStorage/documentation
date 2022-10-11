
# How to use the Object API / ObjectServiceClient

## Introduction

Objects are the primary resource to actually hold the data you upload. 
Before you can initialize any Objects you have to create Collection to ensure consistent 

If you don't know how to create a Collection you should read the previous chapter about the Collection API basics: [How to Collection](04_How-To-Collections.md)


## Initialize Object

The first step is to initialize an Object.
This creates an Object in the AOS and marks it as _Staging_.
As long as an Object is in the staging area data can be uploaded to it.

```bash
# Native JSON request to initialize an staging object
curl -d '
  {
    "object": {
      "filename": "aruna.png",
      "description": "Aruna Object Storage logo.",
      "collectionId": "<collection-id>",
      "contentLen": "123456",
      "dataclass": "DATA_CLASS_PRIVATE",
      "labels": [
        {
          "key": "LabelKey",
          "value": "LabelValue"
        }
      ],
      "hooks": []
    },
    "preferredEndpointId": "",
    "multipart": false,
    "isSpecification": false
  }' \
  -H 'Authorization: Bearer <API_TOKEN>' \
  -H 'Content-Type: application/json' \
  -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object
```


## Upload data to an Staging Object

After initializing an Object you can request an upload url with the object id you received from the initialization.
Data then can be uploaded through the received url to the AOS data proxy.

If the data associated with the Object is greater than 4 Gigabytes you have to request a _multipart_ upload and chunk your data in parts which are at most 4 Gigabytes in size.
You also have to request an upload url for each part individually.

**Note:** For each uploaded part of the multipart upload you will receive a so called `ETag` in the response header which has to be saved with the correlating part number for the Object finishing.

**Note:** Upload of parts from multipart upload can be done parallel.

> :warning: **If the data is stored in an S3 endpoint individual parts of multipart upload have to be at least 5MiB in size (5242880 bytes) or the Object finishing will fail.**

### Single part upload
```bash
# Native JSON request to request an upload url for single part upload
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/upload

# Native JSON request to upload single part data through the generated data proxy upload url
curl -X PUT -T <path-to-local-file> <upload-url>
```

### Multipart upload
```bash
# Native JSON request to request an upload url for specific part of multipart upload
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/upload?multipart=true&partNumber=<part-number>

# Upload multipart data with native JSON requests through the generated data proxy upload urls
curl -X PUT -T <path-to-local-file> <upload-url>
```

#### Multipart upload example
```bash
COLLECTION_ID=<collection-id>
OBJECT_ID=<staging-object-id>
UPLOAD_ID=<objects-upload-id>

# Loop over all file parts, request an upload url, upload the data and display ETag with its part number
for i in "Dummy_Archive.tar.gz.aa",1 "Dummy_Archive.tar.gz.ab",2 "Dummy_Archive.tar.gz.ac",3 "Dummy_Archive.tar.gz.ad",4; 
  do IFS=","; # Split input at comma
  set -- $i;  # Convert the "tuple" into the param args $1 $2 ...
    PART_FILE=$1
    PART_NUM=$2

    UPLOAD_URL=$(curl -s -H "$API_HEADER" \
                         -H 'Content-Type: application/json' \
                         -X GET "https://<URL-to-AOS-instance-API-gateway>/v1/collection/${COLLECTION_ID}/object/${OBJECT_ID}/staging/${UPLOAD_ID}/upload?multipart=true&partNumber=${PART_NUM}" | jq -r '.url.url')    
    ETAG=$(curl -X PUT -T ${PART_FILE} -i "${UPLOAD_URL}" | grep etag)

    echo -e "\nPart: ${PART_NUM}, ETag: ${ETAG}\n"
done
```


## Finish Object

Finishing the Object transfers it from the staging area into production. 
From this moment the Object is generally available for other functions other than [Get Object](#get-object).

On success the response will contain all the information on the finished Object.

```bash
# Native JSON request to finish a single upload staging object
curl -d '
  {
    "hash": {
      "alg": "HASHALGORITHM_SHA256",
      "hash": "8e83e391f7d4bd995e772029a097d42f9fa4f433a8e99585d4a902f599dc7b9c"
    },
    "noUpload": false,
    "completedParts": [],
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/finish
```

```bash
# Native JSON request to finish a multipart upload staging object
curl -d '
  {
    "hash": {
      "alg": "HASHALGORITHM_SHA256",
      "hash": "8e83e391f7d4bd995e772029a097d42f9fa4f433a8e99585d4a902f599dc7b9c"
    },
    "noUpload": false,
    "completedParts": [
      {
        "etag": "<etag-of-part-number-upload>",
        "part": "<part-number>"
      },
      {
        "etag": "<etag-of-part-number-upload>",
        "part": "<part-number>"
      }
    ],
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/finish
```


## Get Object

Information on finished or staging Objects can be fetched with their id and the id of their collection.

```bash
# Native JSON request to fetch information of an object by its unique id
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}
```

## Get Objects

You can also fetch information on multiple Objects of a Collection. 
The Objects returned will not include staging Objects.

```bash
# Native JSON request to fetch information of the first 20 unfiltered objects in a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects
```

By default, this request also returns only the first 20 Objects.
You can either increase the page size to receive more Objects per request and/or request the next Objects by pagination.

```bash
# Native JSON request to fetch information of the first 250 unfiltered objects in a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?pageRequest.pageSize=250
```

```bash
# Native JSON request to fetch information of the objects 21-40 in a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?pageRequest.lastUuid=<id-of-last-received-object>
```

This request can additionally include id filters to narrow the returned Objects.

```bash
# Native JSON request to fetch information of all objects in a collection matching one of the provided ids
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?labelIdFilter.ids=<object-id-001>&labelIdFilter.ids=<object-id-002>
```

<!--
This request can additionally include id or label filters to narrow the returned Objects.

```bash
# Native JSON request to fetch information of all objects in a collection matching one of the provided ids
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?labelIdFilter.ids=<object-id-001>&labelIdFilter.ids=<object-id-002>
```

```bash
# Native JSON request to fetch information of all objects in a collection matching one of the provided label keys
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?labelIdFilter.ids=<object-id-001>&labelIdFilter.ids=<object-id-002>
```

```bash
# Native JSON request to fetch information of all objects in a collection matching all of the provided label keys
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?labelIdFilter.ids=<object-id-001>&labelIdFilter.ids=<object-id-002>
```
-->

## Download Object data

To download the data associated with an Object you have to request a download url. 
This can be done with an individual request or directly while getting information on an Object.

```bash
# Native JSON request to fetch an Objects download url
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/download

# Download the data with the provided remote file name
curl -J -O -X GET <received-download-url>
```

```bash
# Native JSON request to fetch information of an Object including its download id
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}?withUrl=true

# Download the data
curl -J -O -X GET <received-download-url>
```


## Update Object

Objects can still be updated after finishing.

### Update which does not create a new revision

Just adding one or multiple labels to an Object does not create a new revision.

```bash
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
     -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/add_labels
```

### Update which creates a new revision

Every other update creates a new revision of an Object and basically an updated clone of the Object. 
A regular update of an Object returns the updated Object which is still in the staging area.
Comparable to the Object initialization process, the updated Object must be finished before it is generally available.

> :information_source: **The Object revision will be created as specified in the request. No public fields will be transferred from the old revision of the Object.**

#### Update without data re-upload

```bash
# Native JSON request to just update an Objects description
curl -d '
  {
    "object": {
      "filename": "aruna.png",
      "description": "An updated description of the AOS logo.",
      "collectionId": "<collection-id>",
      "contentLen": "123456",
      "dataclass": "DATA_CLASS_PRIVATE",
        "labels": [
          {
            "key": "LabelKey",
            "value": "LabelValue"
          }
        ],
        "hooks": []
      },
    "reupload": false,
    "preferredEndpointId": "",
    "multiPart": false,
    "isSpecification": false
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/update

# Native JSON request to finish the updated object
curl -d '
  {
    "hash": {
      "alg": "HASHALGORITHM_SHA256",
      "hash": "8e83e391f7d4bd995e772029a097d42f9fa4f433a8e99585d4a902f599dc7b9c"
    },
    "noUpload": true,
    "completedParts": [],
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/finish
```

#### Update with data re-upload

```bash
# Native JSON request to update an Objects description as well as re-upload the data
curl -d '
  {
    "object": {
      "filename": "aruna.png",
      "description": "An updated description of the AOS logo.",
      "collectionId": "<collection-id>",
      "contentLen": "1234567",
      "dataclass": "DATA_CLASS_PRIVATE",
        "labels": [
          {
            "key": "LabelKey",
            "value": "LabelValue"
          }
        ],
        "hooks": []
      },
    "reupload": true,
    "preferredEndpointId": "",
    "multiPart": false,
    "isSpecification": false
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/update

# Native JSON request to request an upload url for single part upload
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/upload

# Native JSON request to upload single part data through the generated data proxy upload url
curl -X PUT -T <path-to-local-file> <upload-url>

# Native JSON request to finish the updated object
curl -d '
  {
    "hash": {
      "alg": "HASHALGORITHM_SHA256",
      "hash": "cb380c5ed9724991fb91acb9e21ba11ff68c27bc3a6121284a946cccfdfaaf47"
    },
    "noUpload": false,
    "completedParts": [],
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/finish
```


## Move Object to other Collection

Objects can be moved to another Collection. This can be used to transfer Collection ownership of an Object.

This process consists of two steps:
1. Create writeable reference of the Object in another collection
2. Delete reference of the Object in the source collection

```bash
# Native JSON request to create a writeable reference in another collection
curl -d '
  {
    "writeable": true,
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<source-collection-id>/object/<object-id>/reference/<destination-collection-id>

# Native JSON request to delete writeable reference in source collection
curl -d '
  {
    "withRevisions": true, 
    "force": false
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<source-collection-id>/object/<object-id>
```


## Get all Object references

You can fetch information of all references an Object has to different Collections.

```bash
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/object/<object-id>/references
```


## Delete Object

Objects can only be deleted from a Collection if at least one other writeable reference of the Object exists in another Collection.
If at least one writeable reference still exists in another collection, only the object's reference to the specific collection is removed.

This also applies for all revisions of the Object and will be enforced if `withRevisions = true` is set.

Permanent deletion conditions:
* No writeable references of the specific Object and its revisions in other Collections
* Use of `force = true` in request

All conditions can be overwritten with the use of `force = true` in the request but this should be avoided at all costs. 
Therefore, only users with administrator permissions on the Project can use the _force_ parameter.

> :warning: **Non-writeable references will also be deleted alongside with the last writeable reference.**

```bash
# Force delete Object with all its revisions
curl -d \
  '{
    "withRevisions": "true", 
    "force": "true"
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/object/<object-id>
```