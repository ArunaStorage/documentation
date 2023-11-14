
# How to use the Dataset API / DatasetServiceClient

## Introduction

Datasets are the third-level (and therefore also optional) resource to organize stored data.
Hence, they need at least a Project or a Collection as parent for their creation. 
Datasets should be used to group objects that are closely related to each other into a logical unit and to describe them with additional metadata.

If you don't know how to create a Project you should read the previous chapter about the [**Project API basics**](04_How-To-Project.md).

If you don't know how to create a Collection you should read the previous chapter about the [**Collection API basics**](05_How-To-Collections.md) which is eerily similar to the Project API.


## Create Dataset

API example for creating a new Dataset.

??? Abstract "Required permissions"

    This request requires at least APPEND permission on the parent resource in which the Dataset is to be created.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a simple Dataset
    curl -d '
      {
        "name": "json-api-dataset", 
        "description": "Created with JSON over HTTP.",
        "keyValues": [],
        "relations": [],
        "data_class": "DATA_CLASS_PUBLIC",
        "projectId": "<project-id>",
        "collectionId": "<dataset-id>",
        "metadataLicenseTag": "CC-BY-4.0",
        "defaultDataLicenseTag": "CC-BY-4.0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/dataset
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a Dataset
    let request = CreateDatasetRequest {
        name: "rust-api-dataset".to_string(),
        description: "Created with the gRPC Rust API client.".to_string(),
        key_values: vec![],
        external_relations: vec![],
        data_class: DataClass::Public as i32,
        metadata_license_tag: Some("CC-BY-4.0".to_string()),
        default_data_license_tag: Some("CC-BY-4.0".to_string()),
        parent: Some(Parent::ProjectId("<project-id>".to_string())),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.create_dataset(request)
                                 .await
                                 .unwrap() 
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new Dataset
    request = CreateDatasetRequest(
        name="python-api-project",
        description="Created with the gRPC Python API client.",
        key_values=[], 
        external_relations=[], 
        data_class=DataClass.DATA_CLASS_PUBLIC,
        project_id="<project-id>",
        collection_id="<collection-id>",
        metadata_license_tag="CC-BY-4.0",
        default_data_license_tag="CC-BY-4.0"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.CreateDataset(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get Dataset(s)

API examples of how to fetch information for one or multiple existing Dataset(s).

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Dataset or one if its parent resources.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a Dataset
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET 'https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}'
    ```

    ```bash linenums="1"
    # Native JSON request to fetch information of multiple Datasets
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET 'https://<URL-to-AOS-instance-API-gateway>/v2/datasets?datasetIds=dataset-id-01&datasetIds=dataset-id-02'
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a Dataset
    let request = GetDatasetRequest {
        dataset_id: "<dataset-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.get_dataset(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of multiple Datasets
    let request = GetDatasetsRequest {
        dataset_ids: vec![
            "<dataset-id-01>".to_string(),
            "<dataset-id-02>".to_string(),
            "<...>".to_string(),
        ],
    };

    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.get_datasets(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of a Dataset
    request = GetDatasetRequest(
        dataset_id="<dataset-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.GetDataset(request=request)

    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of multiple Datasets
    request = GetDatasetsRequest(
        dataset_ids=[
            "<dataset-id-01>",
            "<dataset-id-02>",
            "<...>"]
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.GetDatasets(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Update Dataset

API examples of how to update individual metadata of an existing Dataset.

!!! Warning "Update operations that trigger the creation of a new revision"

    * Name update
    * Removal of KeyValues
    * License update except from the default `AllRightsReserved`

??? Abstract "Required permissions"

    * Name update needs at least WRITE permissions on the specific Dataset or one of its parent resources
    * Description update needs at least WRITE permissions on the specific Dataset or one of its parent resources
    * KeyValue update needs at least WRITE permissions on the specific Dataset or one of its parent resources
    * Dataclass update needs at least WRITE permissions on the specific Dataset or one of its parent resources

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to update the name of a Dataset
    curl -d '
      {
        "name": "updated-json-api-dataset"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}/name
    ```

    ```bash linenums="1"
    # Native JSON request to update the description of a Dataset
    curl -d '
      {
        "description": "Updated with JSON over HTTP."
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}/description
    ```

    ```bash linenums="1"
    # Native JSON request to update the key-values associated with a Dataset
    curl -d '
      {
        "addKeyValues": [],
        "removeKeyValues": []
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}/key_values
    ```

    !!! Info

        Dataclass can only be relaxed: `Confidential` > `Private` > `Public`

    ```bash linenums="1"
    # Native JSON request to update the dataclass of a Dataset
    curl -d '
      {
        "dataClass": "DATA_CLASS_PUBLIC"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}/data_class
    ```

    ```bash linenums="1"
    # Native JSON request to update the license of a Dataset
    curl -d '
      {
        "metadataLicenseTag": "CC0",
        "defaultDataLicenseTag": "CC0"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}/licenses
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the name of a Dataset
    let request = UpdateDatasetNameRequest {
        dataset_id: "<dataset-id>".to_string(),
        name: "updated-rust-api-dataset".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.update_dataset_name(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the description of a Dataset
    let request = UpdateDatasetDescriptionRequest {
        dataset_id: "<dataset-id>".to_string(),
        description: "Updated with the gRPC Rust API client.".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.update_dataset_description(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the key-values associated with a Dataset
    let request = UpdateDatasetKeyValuesRequest {
        dataset_id: "<dataset-id>".to_string(),
        add_key_values: vec![], 
        remove_key_values: vec![]
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.update_dataset_key_values(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    !!! Info

        Dataclass can only be relaxed: `Confidential` > `Private` > `Public`

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the datacalass of a Dataset
    let request = UpdateDatasetDataClassRequest {
        dataset_id: "<dataset-id>".to_string(),
        data_class: DataClass::Public as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.update_dataset_data_class(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the licenses of a Dataset
    let request = UpdateDatasetLicensesRequest {
        dataset_id: "<dataset-id>".to_string(),
        metadata_license_tag: "CC0".to_string(),
        default_data_license_tag: "CC0".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.update_dataset_licenses(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the name of a Dataset
    request = UpdateDatasetNameRequest(
        dataset_id="<dataset-id>",
        name="updated-python-api-project"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.UpdateDatasetName(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the description of a Dataset
    request = UpdateDatasetDescriptionRequest(
        dataset_id="<dataset-id>",
        description="Updated with the gRPC Python API client"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.UpdateDatasetDescription(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the key-values associated with a Dataset
    request = UpdateDatasetKeyValuesRequest(
        dataset_id="<dataset-id>",
        add_key_values=[],
        remove_key_values=[]
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.UpdateDatasetKeyValues(request=request)
    
    # Do something with the response
    print(f'{response}')
    ``` 

    !!! Info

        Dataclass can only be relaxed: `Confidential` > `Private` > `Public`

    ```python linenums="1"
    # Create tonic/ArunaAPI request to relax the data_class of a Dataset
    request = UpdateDatasetDataClassRequest(
        dataset_id="<dataset-id>",
        data_class=DataClass.DATA_CLASS_PUBLIC
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.UpdateDatasetDataClass(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the licenses of a Dataset
    request = UpdateDatasetLicensesRequest(
        dataset_id="<dataset-id>",
        metadata_license_tag="CC0",
        default_data_license_tag="CC0"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.UpdateDatasetLicenses(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Snapshot Dataset

API examples of how to snapshot a Dataset, i.e. create an immutable clone of the Dataset and its underlying resources.

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the Dataset or one if its parent resources.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to snapshot a Dataset
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
      -H 'Content-Type: application/json' \
      -X POST https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}/snapshot
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to snapshot a Dataset
    let request = SnapshotDatasetRequest {
        dataset_id: "<dataset-id>".to_string()
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.snapshot_dataset_version(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to snapshot a Dataset
    request = SnapshotDatasetRequest(
        dataset_id="<dataset-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.SnapshotDatasetVersion(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Delete Dataset

API examples of how to delete a Dataset.

!!! Info

    Deletion does not remove the Dataset from the database, but sets the status of the Dataset and the underlying resources to "DELETED".

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the Dataset or one if its parent resources.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a Dataset
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/dataset/{dataset-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a Dataset
    let request = DeleteDatasetRequest {
        dataset_id: "<dataset-id>".to_string()
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = dataset_client.delete_dataset(request)
                                    .await
                                    .unwrap()
                                    .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a Dataset
    request = DeleteDatasetRequest(
        dataset_id="<dataset-id>"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.dataset_client.DeleteDataset(request=request)

    # Do something with the response
    print(f'{response}')
    ```
