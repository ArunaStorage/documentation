
# How to use the StorageStatusService API / StorageStatusClient

## Introduction

To get general information about an Aruna instance and its running components you can use the StorageStatusService API.

For example, this can be either interesting for developers to know which versions they're dealing with or maybe to 
introduce a service health monitor with the current status of the instance.


## Get Storage Version

API example for fetching the versions of all components running in the specific Aruna instance.

??? Abstract "Required permissions"

    This request does not require any permissions.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get Aruna instance component versions 
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/info/version
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch component versions of the Aruna instance
    let get_request = GetStorageVersionRequest {};
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = info_client.get_storage_version(get_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch component versions of the Aruna instance
    request = GetStorageVersionRequest()

    # Send the request to the Aruna instance gRPC endpoint
    response = client.info_client.GetStorageVersion(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## Get Storage Status

API example for fetching current status of the Aruna instance.

??? Abstract "Required permissions"

    This request does not require any permissions.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get Aruna stoarge instance status
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/info/status
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch the current status of the Aruna instance
    let get_request = GetStorageStatusRequest {};
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = info_client.get_storage_status(get_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch the current status of the Aruna instance
    request = GetStorageStatusRequest()

    # Send the request to the Aruna instance gRPC endpoint
    response = client.info_client.GetStorageStatus(request=request)

    # Do something with the response
    print(f'{response}')
    ```

## Get available public keys 

API example for fetching a list of the available public keys associated with the ArunaServer and DataProxy instances.

These can be used to validate signatures of tokens and/or presigned URLs.

??? Abstract "Required permissions"

    This request does not require any permissions.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get the list of available public keys
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/info/pubkeys
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to get the list of available public keys
    let get_request = GetPubkeysRequest {};
    
    // Send the request to the Aruna instance gRPC endpoint
    let response = info_client.get_pubkeys(get_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to get the list of available public keys
    request = GetPubkeysRequest()

    # Send the request to the Aruna instance gRPC endpoint
    response = client.info_client.GetPubkeys(request=request)

    # Do something with the response
    print(f'{response}')
    ```
