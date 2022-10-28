
# How to use the Object API / ObjectServiceClient

## Introduction

Objects are the primary resource to actually hold the data you upload. 
Before you can initialize any Objects you have to create Collection to ensure consistent 

If you don't know how to create a Collection you should read the previous chapter about the Collection API basics: [How to Collection](04_How-To-Collections.md)


## Initialize Object

The first step is to initialize an Object.
This creates an Object in the AOS and marks it as _Staging_.
As long as an Object is in the staging area data can be uploaded to it.

This request needs at least APPEND permissions on the Object's Collection or the Project under which the Collection is registered.

### Bash:
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

### Rust:
```rust
// Create tonic/ArunaAPI request to initialize an staging object
let init_request = InitializeNewObjectRequest {
    object: Some(StageObject {
        filename: "aruna.png".to_string(),
        description: "Aruna Object Storage logo".to_string(),
        collection_id: "<collection-id>".to_string(),
        content_len: 123456,
        source: None,
        dataclass: DataClass::Private as i32,
        labels: vec![KeyValue {
            key: "LabelKey".to_string(),
            value: "LabelValue".to_string(),
        }],
        hooks: vec![KeyValue {
            key: "HookKey".to_string(),
            value: "HookValue".to_string(),
        }],
    }),
    collection_id: "<collection-id>".to_string(),
    preferred_endpoint_id: "".to_string(),
    multipart: false,
    is_specification: false,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.initialize_new_object(init_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Upload data to an Staging Object

After initializing an Object you can request an upload url with the object id you received from the initialization.
Data then can be uploaded through the received url to the AOS data proxy.

If the data associated with the Object is greater than 4 Gigabytes you have to request a _multipart_ upload and chunk your data in parts which are at most 4 Gigabytes in size.
You also have to request an upload url for each part individually.

Requesting an upload URL needs at least APPEND permissions on the Object's Collection or the Project under which the Collection is registered.

**Note:** For each uploaded part of the multipart upload you will receive a so called `ETag` in the response header which has to be saved with the correlating part number for the Object finishing.

**Note:** Numbers of upload parts start with 1, not 0.

> :information_source: **Upload of parts from multipart upload can be done parallel.**

> :warning: **If the data is stored in an S3 endpoint individual parts of multipart upload have to be at least 5MiB in size (5242880 bytes) or the Object finishing will fail.**

### Single part upload

#### Bash:
```bash
# Native JSON request to request an upload url for single part upload
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/upload

# Native JSON request to upload single part data through the generated data proxy upload url
curl -X PUT -T <path-to-local-file> <upload-url>
```

#### Rust:
```rust
// Create tonic/ArunaAPI request to request an upload url for single part upload
let get_request = GetUploadUrlRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    upload_id: "<upload-id>".to_string(),
    multipart: false,
    part_number: 1,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_upload_url(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
let upload_url = response.url.unwrap().url;

// Upload local file to the generated upload URL
let path = Path::new("/path/to/local/file");
let file = tokio::fs::File::open(path).await.unwrap();

let client = reqwest::Client::new();
let stream = FramedRead::new(file, BytesCodec::new());
let body   = Body::wrap_stream(stream);

// Send the request to the upload url
let response = client.put(upload_url)
                     .body(body)
                     .send()
                     .await
                     .unwrap();

// Do something with the response
println!("{:#?}", response);
```

### Multipart upload

#### Bash:
```bash
# Native JSON request to request an upload url for specific part of multipart upload
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}/staging/{upload-id}/upload?multipart=true&partNumber=<part-number>

# Upload multipart data with native JSON requests through the generated data proxy upload urls
curl -X PUT -T <path-to-local-file> <upload-url>
```

#### Rust:
```rust
// Create tonic/ArunaAPI to request an upload url for specific part of multipart upload
let get_request = GetUploadUrlRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    upload_id: "<upload-id>".to_string(),
    multipart: true,
    part_number: <part-number> as i32,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_upload_url(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

##### Multipart upload example

###### Bash:
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

###### Rust:
```rust
let mut file = tokio::fs::File::open("/path/to/local/file").await.unwrap();     // File handle
let mut remaining_bytes: usize = file.metadata().await.unwrap().len() as usize; // File size in bytes
let mut upload_part_counter: i64 = 0; 
let mut completed_parts: Vec<CompletedParts> = Vec::new();

const UPLOAD_BUFFER_SIZE: usize = 1024 * 1024 * 50; // 50MiB chunks
let mut buffer_size = UPLOAD_BUFFER_SIZE;           // Variable buffer size for loop

loop {
    // Increment part number
    upload_part_counter += 1;
    
    // Set buffer size
    if remaining_bytes < UPLOAD_BUFFER_SIZE {
        buffer_size = remaining_bytes;
    }

    // Fill buffer with bytes from file
    let mut data_buf = vec![0u8; buffer_size];
    file.read_exact(&mut data_buf).await.unwrap();

    // Create tonic/ArunaAPI request to request an upload url for multipart upload part
    let upload_url = object_client
        .get_upload_url(GetUploadUrlRequest {
            object_id: object_id.to_string(),
            upload_id: upload_id.to_string(),
            collection_id: collection_id.to_string(),
            multipart: true,
            part_number: upload_part_counter as i32,
        })
        .await
        .unwrap()
        .into_inner()
        .url.unwrap().url;

    // Upload buffer content to upload url and parse ETag from response header
    let client   = reqwest::Client::new();
    let response = client.put(upload_url).body(data_buf).send().await.unwrap();
    let etag_raw = response.headers().get("ETag").unwrap().as_bytes();
    let etag     = std::str::from_utf8(etag_raw).unwrap().to_string();

    // Collect ETag with corresponding part number
    completed_parts.push(CompletedParts {
        etag: etag,
        part: upload_part_counter,
    });

    // Update the amount of remaining bytes for the next loop
    remaining_bytes -= buffer_size;
    if remaining_bytes == 0 {
        break;
    }
}

// Retain the completed parts for usage in the FinishObjectRequest
println!("{:#?}", completed_parts);
```


## Finish Object

Finishing the Object transfers it from the staging area into production. 
From this moment the Object is generally available for other functions other than [Get Object](#get-object).
On success the response will contain all the information on the finished Object.

This request needs at least APPEND permissions on the Object's Collection or the Project under which the Collection is registered.

### Bash:
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

### Rust:
```rust
// Create tonic/ArunaAPI request to finish a single part upload staging object
let finish_request = FinishObjectStagingRequest {
    object_id: "<object-id>".to_string(),
    upload_id: "<upload-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    hash: Some(Hash {
        alg: Hashalgorithm::Sha256 as i32,
        hash: "8e83e391f7d4bd995e772029a097d42f9fa4f433a8e99585d4a902f599dc7b9c".to_string()
    }),
    no_upload: false,
    completed_parts: vec![],
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.finish_object_staging(finish_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

```rust
// Create tonic/ArunaAPI request to finish a multipart upload staging object
let finish_request = FinishObjectStagingRequest {
    object_id: "<object-id>".to_string(),
    upload_id: "<upload-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    hash: Some(Hash {
        alg: Hashalgorithm::Sha256 as i32,
        hash: "8e83e391f7d4bd995e772029a097d42f9fa4f433a8e99585d4a902f599dc7b9c".to_string()
    }),
    no_upload: false,
    completed_parts: vec![
        CompletedParts {
            etag: "bfa7d257edcc9b962b7ab1a0a75b9808".to_string(),
            part: 1,
        },
        CompletedParts {
            etag: "af62c64eb6bcc9890d0aadf5720bf1ff".to_string(),
            part: 2,
        },
    ],
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.finish_object_staging(finish_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Get Object

Information on finished or staging Objects can be fetched with their id and the id of their collection.

This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to fetch information of an object by its unique id
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/object/{object-id}
```

### Rust:
```rust
// Create tonic/ArunaAPI request to fetch information of an object
let get_request = GetObjectByIdRequest {
    collection_id: "<collection-id>".to_string(),
    object_id: "<object-id>".to_string(),
    with_url: false
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_object(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Get Objects

You can also fetch information on multiple Objects of a Collection. 
The Objects returned will not include staging Objects.

By default, this request also returns only the first 20 Objects.
You can either increase the page size to receive more Objects per request and/or request the next Objects by pagination. 
Additionally, you can include id or label filters to narrow the returned Objects.

This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to fetch information of the first 20 unfiltered objects in a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects
```

```bash
# Native JSON request to fetch information of the first 250 unfiltered objects in a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?pageRequest.pageSize=250
```

```bash
# Native JSON request to fetch information of the objects 21-40 (i.e. the next page) in a collection
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/{collection-id}/objects?pageRequest.lastUuid=<id-of-last-received-object>
```

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

### Rust:
```rust
// Create tonic/ArunaAPI to fetch information of the first 20 unfiltered objects in a collection
let get_request = GetObjectsRequest {
    collection_id: "<collection-id>".to_string(),
    page_request: None,
    label_id_filter: None,
    with_url: false,
    include_history: false,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_objects(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

```rust
// Create tonic/ArunaAPI to fetch information of the first 250 unfiltered objects in a collection
let get_request = GetObjectsRequest {
    collection_id: "<collection-id>".to_string(),
    page_request: Some(PageRequest {
        last_uuid: "".to_string(),
        page_size: 250,
    }),
    label_id_filter: None,
    with_url: false,
    include_history: false,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_objects(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

```rust
// Create tonic/ArunaAPI to fetch information of the objects 21-40 (i.e. the next page) in a collection
let get_request = GetObjectsRequest {
    collection_id: "<collection-id>".to_string(),
    page_request: Some(PageRequest {
        last_uuid: "<id-of-last-received-object>".to_string(),
        page_size: 0,
    }),
    label_id_filter: None,
    with_url: false,
    include_history: false,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_objects(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

```rust
// Create tonic/ArunaAPI to fetch information of all objects in a collection matching one of the provided ids
let get_request = GetObjectsRequest {
    collection_id: "<collection-id>".to_string(),
    page_request: None,
    label_id_filter: Some(LabelOrIdQuery {
        labels: None,
        ids: vec!["<object-id-001>".to_string(), "<object-id-001>".to_string()],
    }),
    with_url: false,
    include_history: false,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_objects(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

```rust
// Create tonic/ArunaAPI to fetch multiple object of a collection filtered by label key(s)
let get_request = GetObjectsRequest {
    collection_id: "<collection-id>".to_string(),
    page_request: None,
    label_id_filter: Some(LabelOrIdQuery {
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
    with_url: false,
    include_history: false,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_objects(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Download Object data

To download the data associated with an Object you have to request a download url. 
This can be done with an individual request or directly while getting information on an Object.

This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

### Bash:
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

### Rust:
```rust
// Send GET request to download url
let response = reqwest::get(download_url).await.unwrap();

// Only proceed if response status is ok
if response.status() != StatusCode::OK {
    panic!("Get request for file download failed.");
}

// Set default filename to "object.<object-id>"
let mut file_name = format!(
    "object.{}",
    response.url().path_segments().unwrap().last().unwrap()
);

// Try to extract filename from download url query parameter
for elem in response.url().query_pairs() {
    if elem.0 == "filename" {
        file_name = elem.1.to_string();
    }
}

// Create local file
let target_path = Path::new("/tmp").join();
let mut target_file = File::create(target_path).unwrap();

// Write response content to file
let mut content = Cursor::new(response.bytes().await.unwrap());
copy(&mut content, &mut target_file).unwrap();
```


## Update Object

Objects can still be updated after finishing.

This request needs at least MODIFY permissions on the Object's Collection or the Project under which the Collection is registered.

### Update which does not create a new revision

Just adding one or multiple labels to an Object does not create a new revision.

#### Bash:
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

#### Rust:
```rust
// Create tonic/ArunaAPI request to add a label to an object
let add_request = AddLabelToObjectRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    labels_to_add: vec![KeyValue {
        key: "AnotherKey".to_string(),
        value: "AnotherValue".to_string(),
    }],
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.add_label_to_object(add_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


### Update which creates a new revision

Every other update creates a new revision of an Object and basically an updated clone of the Object. 
A regular update of an Object returns the updated Object which is still in the staging area.
Comparable to the Object initialization process, the updated Object must be finished before it is generally available.

> :information_source: **The Object revision will be created as specified in the request. No public fields will be transferred from the old revision of the Object.**

#### Update without data re-upload

##### Bash:
```bash
# Native JSON request to update an objects description
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

##### Rust:
```rust
// Create tonic/ArunaAPI request to update an objects description
let update_request = UpdateObjectRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    object: Some(StageObject {
        filename: "aruna.png".to_string(),
        description: "An updated description of the AOS logo.".to_string(),
        collection_id: "<collection-id>".to_string(),
        content_len: 123456,
        source: None,
        dataclass: DataClass::Private as i32,
        labels: vec![KeyValue {
            key: "LabelKey".to_string(),
            value: "LabelValue".to_string(),
        }],
        hooks: vec![KeyValue {
            key: "HookKey".to_string(),
            value: "HookValue".to_string(),
        }],
    }),
    reupload: false,
    preferred_endpoint_id: "".to_string(),
    multi_part: false,
    is_specification: false
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.update_object(update_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
let UpdateObjectResponse {object_id, staging_id, collection_id} = response;

// Create tonic/ArunaAPI request to finish the updated object
let finish_request = FinishObjectStagingRequest {
    object_id: object_id,
    upload_id_ staging_id,
    collection_id: collection_id,
    hash: Some(Hash {
        alg: Hashalgorithm::Sha256 as i32,
        hash: "8e83e391f7d4bd995e772029a097d42f9fa4f433a8e99585d4a902f599dc7b9c".to_string()
    }),
    no_upload: true,
    completed_parts: vec![],
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.finish_object_staging(finish_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


#### Update with data re-upload

##### Bash:
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

##### Rust:
```rust
// Create tonic/ArunaAPI request to update an objects description as well as re-upload the data
let update_request = UpdateObjectRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    object: Some(StageObject {
        filename: "aruna.png".to_string(),
        description: "An updated description of the AOS logo.".to_string(),
        collection_id: "<collection-id>".to_string(),
        content_len: 1234567,
        source: None,
        dataclass: DataClass::Private as i32,
        labels: vec![KeyValue {
            key: "LabelKey".to_string(),
            value: "LabelValue".to_string(),
        }],
        hooks: vec![KeyValue {
            key: "HookKey".to_string(),
            value: "HookValue".to_string(),
        }],
    }),
    reupload: true,
    preferred_endpoint_id: "".to_string(),
    multi_part: false,
    is_specification: false
};

// Send the request to the AOS instance gRPC gateway
let update_response = object_client.update_object(update_request)
                                   .await
                                   .unwrap()
                                   .into_inner();

// Do something with the response
println!("{:#?}", update_response);
let UpdateObjectResponse {
    object_id, 
    staging_id, 
    collection_id
} = update_response;

// Create tonic/ArunaAPI request to request an upload url for single part upload
let get_request = GetUploadUrlRequest {
    object_id: object_id.clone(),
    collection_id: collection_id.clone(),
    upload_id: staging_id.clone(),
    multipart: false,
    part_number: 1,
};

// Send the request to the AOS instance gRPC gateway
let get_response = object_client.get_upload_url(get_request)
                                .await
                                .unwrap()
                                .into_inner();

// Do something with the response
println!("{:#?}", get_response);
let upload_url = get_response.url.unwrap().url;

// Upload local file to the generated upload URL
let path = Path::new("/path/to/local/file");
let file = tokio::fs::File::open(path).await.unwrap();

let client = reqwest::Client::new();
let stream = FramedRead::new(file, BytesCodec::new());
let body   = Body::wrap_stream(stream);

// Send the request to the upload url
let upload_response = client.put(upload_url)
                            .body(body)
                            .send()
                            .await
                            .unwrap();

// Do something with the response
println!("{:#?}", upload_response);

// Create tonic/ArunaAPI request to finish a single part upload staging object
let finish_request = FinishObjectStagingRequest {
    object_id: object_id,
    upload_id: staging_id,
    collection_id: collection_id,
    hash: Some(Hash {
        alg: Hashalgorithm::Sha256 as i32,
        hash: "c1557cf6f426823f248f7285b3540812521935ebca372fb6689463efd6cadb61".to_string()
    }),
    no_upload: false,
    completed_parts: vec![],
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let finish_response = object_client.finish_object_staging(finish_request)
                                   .await
                                   .unwrap()
                                   .into_inner();

// Do something with the response
println!("{:#?}", finish_response);
```


## Create Object reference

References of an Object can be created in other Collections, which points to the same object in the database. 
A reference can be either _"read only"_, which means that the Object can not be modified from the specific Collection, or writeable, in which case the object in the target collection will be no different from the one in the source collection.

> :information_source: **Updates on writeable references also update the Object in the source Collection.**

This request needs permission in the source and target Collection.
The required permissions on the source collection depend on whether the reference should be writeable or not:

* `true`: At least MODIFY on the source Collection or the Project under which the Collection is registered
* `false`: At least READ on the source Collection or the Project under which the Collection is registered

Additionally, the request needs MODIFY permissions on the target Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to create an auto-updated read-only reference to keep track of an object
curl -d '
  {
    "writeable": false,
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<source-collection-id>/object/<object-id>/reference/<destination-collection-id>
```

```bash
# Native JSON request to create a writeable reference in another collection for collaborative work
curl -d '
  {
    "writeable": true,
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<source-collection-id>/object/<object-id>/reference/<destination-collection-id>
```

### Rust:
```rust
// Create tonic/ArunaAPI request to create an auto-updated read-only reference to keep track of an object
let create_request = CreateObjectReferenceRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<source-collection-id>".to_string(),
    target_collection_id: "<target-collection-id>".to_string(),
    writeable: false,
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.create_object_reference(create_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```

```rust
// Create tonic/ArunaAPI request to create a writeable reference in another collection for collaborative work
let create_request = CreateObjectReferenceRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<source-collection-id>".to_string(),
    target_collection_id: "<target-collection-id>".to_string(),
    writeable: true,
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.create_object_reference(create_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Move Object to other Collection

Objects can be moved to another Collection without cloning. 
E.g. this can be used to transfer Collection ownership of an Object.

This process consists of two steps:
1. Create writeable reference of the Object in another Collection
2. Delete reference of the Object in the source Collection

### Bash:
```bash
# Native JSON request to create a writeable reference in another collection
curl -d '
  {
    "writeable": true,
    "autoUpdate": true
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/collection/<source-collection-id>/object/<object-id>/reference/<target-collection-id>

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

### Rust:
```rust
// Create tonic/ArunaAPI request to create a writeable reference in another collection
let create_request = CreateObjectReferenceRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<source-collection-id>".to_string(),
    target_collection_id: "<target-collection-id>".to_string(),
    writeable: true,
    auto_update: true,
};

// Send the request to the AOS instance gRPC gateway
let create_response = object_client.create_object_reference(create_request)
                                   .await
                                   .unwrap()
                                   .into_inner();

// Do something with the response
println!("{:#?}", create_response);

// Create tonic/ArunaAPI request to delete the object in the source collection
let delete_request = DeleteObjectRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<source-collection-id>".to_string(),
    with_revisions: true,
    force: false,
};

// Send the request to the AOS instance gRPC gateway
let delete_response = object_client.delete_object(delete_request)
                                   .await
                                   .unwrap()
                                   .into_inner();

// Do something with the response
println!("{:#?}", delete_response);
```


## Get all Object references

You can fetch information of all references an Object has in different Collections.

This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

### Bash:
```bash
# Native JSON request to fetch all references of an object
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/object/<object-id>/references
```

### Rust:
```rust
// Create tonic/ArunaAPI request to fetch all references of an object
let get_request = GetReferencesRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    with_revisions: false
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.get_references(get_request)
                            .await
                            .unwrap()
                            .into_inner();

// Do something with the response
println!("{:#?}", response);
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

The required permissions of this request depend on the value of the `force` parameter:
* `true`: At least ADMIN on the source Collection or the Project under which the Collection is registered
* `false`: At least APPEND on the source Collection or the Project under which the Collection is registered

> :warning: **Non-writeable references will also be deleted alongside with the last writeable reference.**

### Bash:
```bash
# Native JSON request to force delete an object with all its revisions
curl -d \
  '{
    "withRevisions": "true", 
    "force": "true"
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/collection/<collection-id>/object/<object-id>
```

### Rust:
```rust
// Create tonic/ArunaAPI request to delete an object with all its revisions
let delete_request = DeleteObjectRequest {
    object_id: "<object-id>".to_string(),
    collection_id: "<collection-id>".to_string(),
    with_revisions: true,
    force: true,
};

// Send the request to the AOS instance gRPC gateway
let response = object_client.delete_object(delete_request)
                                   .await
                                   .unwrap()
                                   .into_inner();

// Do something with the response
println!("{:#?}", response);
```