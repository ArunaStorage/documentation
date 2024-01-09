
# How to use the ServiceAccount API / ServiceAccountServiceClient

## Introduction

AOS offers the possibility to create service accounts that can be used impersonally, 
e.g. by several users at the same time or by a service that communicates with the AOS via the API.

In order for a service account to be used against the API, it must be assigned a permission for a specific resource. 
For security reasons, a service account can only have this one permission at the same time, which can, however, be adjusted afterwards. 

The service account also must create at least a personal token for communication with the API, 
which takes over the specific permission of the service account in the authorisation process. 
If the service account is also to be entrusted with the upload and download of data, S3 credentials must be requested once from each DataProxy where data is to be stored or read. 

!!! Warning "Service Account Limitations"

    Service accounts behave like normal user accounts with the following limitations:

    * The service account permission can only be set on Projects
    * Only one permission can be assigned at the same time
    * Tokens can only be created on the resource the current permission is associated with and its subresources
    * All service account tokens get deleted if the permission gets set to another Project
    * License and data class updates of resources are not allowed
    * Service accounts are not allowed to send requests against the following services:
        * EndpointService
        * AuthorizationService
        * UserService
        * LicenseService


## Create service account

API examples of how to create a service account.

??? Abstract "Required permissions"

    Service account creation requires at least ADMIN permissions on the specific resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a service account with ADMIN permission on a Project
    curl '
      {
        "name": "string",
        "permission": {
            "projectId": "<project-id>",
            "collectionId": "",
            "datasetId": "",
            "objectId": "",
            "permissionLevel": "PERMISSION_LEVEL_ADMIN"
        }
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a service account with ADMIN permission on a Project
    let request = CreateServiceAccountRequest {
        name: name.to_string(),
        permission: Some(Permission {
            permission_level: PermissionLevel::Admin as i32,
            resource_id: Some(ResourceId::ProjectId("<project-id>".to_string())),
        }),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.create_service_account(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a service account with ADMIN permission on a Project
    request = CreateServiceAccountRequest(
        name="<service-account-name>",
        permission=Permission(
            project_id="<project-id>", # (1)
            permission_level=PermissionLevel.PERMISSION_LEVEL_ADMIN
        )
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.CreateServiceAccount(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    1. **Empty fields of the request were omitted:**
    ```python linenums="1"
    request = CreateServiceAccountRequest(
        name="<service-account-name>",
        permission=Permission(
            project_id="<project-id>",
            collection_id="",
            dataset_id="",
            object_id="",
            permission_level=PermissionLevel.PERMISSION_LEVEL_ADMIN
        )
    )
    ```


## Set service account permission

API examples of how to set the specific permission of a service account.

??? Abstract "Required permissions"

    Setting the permission of a service account requires at least ADMIN permissions on the previous resource and the specified resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to set the permission of an exisiting service account
    curl '
      {
        "permission": {
            "projectId": "",
            "collectionId": "<collection-id>",
            "datasetId": "",
            "objectId": "",
            "permissionLevel": "PERMISSION_LEVEL_WRITE"
        }
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PUT https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/permissions
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to set the permission of an exisiting service account
    let request = SetServiceAccountPermissionRequest {
        svc_account_id: "<svc-account-id>".to_string(),
        permission: Some(Permission {
            permission_level: PermissionLevel::Write as i32,
            resource_id: Some(ResourceId::CollectionId("<collection-id>".to_string())),
        }),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.set_service_account_permission(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to set the permission of an exisiting service account
    request = SetServiceAccountPermissionRequest(
        svc_account_id="<svc-account-id>",
        permission=Permission(
            collection_id="<collection-id>", # (1)
            permission_level=PermissionLevel.PERMISSION_LEVEL_WRITE
        )
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.SetServiceAccountPermission(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    1. **Empty fields of the request were omitted:**
    ```python linenums="1"
    request = CreateServiceAccountRequest(
        name="<service-account-name>",
        permission=Permission(
            project_id="<project-id>",
            collection_id="",
            dataset_id="",
            object_id="",
            permission_level=PermissionLevel.PERMISSION_LEVEL_ADMIN
        )
    )
    ```


## Create service account token

API examples of how to generate a service account token.

Service accounts can have as many tokens as they like, but are limited to resources that are registered hierarchically under the resource for which the service account has its specific permission set.

??? Abstract "Required permissions"

    Setting the permission of a service account requires at least ADMIN permissions on the previous resource and the specified resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a token with READ permissions on a dataset for a service account
    curl '
      {
        "permission": {
            "projectId": "",
            "collectionId": "",
            "datasetId": "<dataset-id>",
            "objectId": "",
            "permissionLevel": "PERMISSION_LEVEL_READ"
        },
        "name": "string",
        "expiresAt": "2030-01-01T08:00:00.000Z"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/tokens
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a token with READ permissions on a dataset for a service account
    let request = CreateServiceAccountTokenRequest {
        svc_account_id: "<svc-account-id>".to_string(),
        permission: Some(Permission {
            permission_level: PermissionLevel::Read as i32,
            resource_id: Some(ResourceId::DatasetId("<dataset-id>".to_string())),
        }),
        name: "<token-name">.to_string(),
        expires_at: Some(
            NaiveDate::from_ymd_opt(2030, 01, 01)
                .unwrap()
                .and_hms_opt(8, 0, 0)
                .unwrap()
                .into(),
        ),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.create_service_account_token(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a token with READ permissions on a dataset for a service account
    request = CreateServiceAccountTokenRequest(
        svc_account_id="<svc-account-id>",
        permission=Permission(
            collection_id="<collection-id>", # (1)
            permission_level=PermissionLevel.PERMISSION_LEVEL_WRITE
        )
        name="<token-name>",
        expires_at=Timestamp(seconds=int(datetime.datetime(2030, 1, 1).timestamp()))
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.CreateServiceAccountToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    1. **Empty fields of the request were omitted:**
    ```python linenums="1"
    request = CreateServiceAccountRequest(
        name="<service-account-name>",
        permission=Permission(
            project_id="<project-id>",
            collection_id="",
            dataset_id="",
            object_id="",
            permission_level=PermissionLevel.PERMISSION_LEVEL_ADMIN
        )
    )
    ```


## Get service account token(s)

API examples of how to fetch information of one or multiple service account tokens.

??? Abstract "Required permissions"

    Fetching information of service account tokens requires at least ADMIN permissions on the resource the service account has set its permission on.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a single specific service account token
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/tokens/{token-id}
    ```

    ```bash linenums="1"
    # Native JSON request to fetch information of all service account tokens
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/tokens
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a single specific service account token
    let request = GetServiceAccountTokenRequest {
        svc_account_id: "<svc_account_id>".to_string(),
        token_id: "<token_id>".to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.get_service_account_token(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of all service account tokens
    let request = GetServiceAccountTokensRequest {
        svc_account_id: svc_account_id.to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.get_service_account_tokens(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of a single specific service account token
    request = GetServiceAccountTokenRequest(
        svc_account_id="<svc-account-id>",
        token_id="<token-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.GetServiceAccountToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of all service account tokens
    request = GetServiceAccountTokensRequest(
        svc_account_id="<svc-account-id>",
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.GetServiceAccountTokens(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Delete service account token(s)

API examples of how to delete one or multiple service account tokens.

??? Abstract "Required permissions"

    Deletion of service account tokens requires at least ADMIN permissions on the resource the service account has set its permission on.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a single specific service account token
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/tokens/{token-id}
    ```

    ```bash linenums="1"
    # Native JSON request to delete all service account tokens
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/tokens
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a single specific service account token
    let request = DeleteServiceAccountTokenRequest {
        svc_account_id: "<svc_account_id>".to_string(),
        token_id: "<token_id>".to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.delete_service_account_token(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete all service account tokens
    let request = DeleteServiceAccountTokensRequest {
        svc_account_id: svc_account_id.to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.delete_service_account_tokens(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a single specific service account token
    request = DeleteServiceAccountTokenRequest(
        svc_account_id="<svc-account-id>",
        token_id="<token-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.DeleteServiceAccountToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete all service account tokens
    request = DeleteServiceAccountTokensRequest(
        svc_account_id="<svc-account-id>",
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.DeleteServiceAccountTokens(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get service account S3 credentials

API examples of how to get S3 credentials for a service account from a specific DataProxy.

??? Abstract "Required permissions"

    Fetching S3 credentials requires at least ADMIN permissions on the resource the service account has set its permission on.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch S3 credentials for the Aruna server instance default DataProxy
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/s3_credentials?endpointId=endpoint-id
    ```

    ```bash linenums="1"
    # Native JSON request to fetch S3 credentials for a specific DataProxy
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/s3_credentials?endpointId=endpoint-id
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request fetch S3 credentials for the Aruna server instance default DataProxy
    let request = GetS3CredentialsSvcAccountRequest {
        svc_account_id: "<svc-account-id>".to_string(),
        endpoint_id: "".to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.get_s3_credentials_svc_account(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch S3 credentials for a specific DataProxy
    let request = GetS3CredentialsSvcAccountRequest {
        svc_account_id: "<svc-account-id>".to_string(),
        endpoint_id: "<endpoint-id>".to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.get_s3_credentials_svc_account(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch S3 credentials for the Aruna server instance default DataProxy
    request = GetS3CredentialsSvcAccountRequest(
        svc_account_id="<svc-account-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.GetS3CredentialsSvcAccount(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch S3 credentials for a specific DataProxy
    request = GetS3CredentialsSvcAccountRequest(
        svc_account_id="<svc-account-id>",
        endpoint_id="<endpoint-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.GetS3CredentialsSvcAccount(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


<!--
## Get service account DataProxy token

API examples of how to generate a service account token for direct communication with a specific DataProxy.

These tokens currently can only be used to fetch S3 credentials or to intiate data replication.

??? Abstract "Required permissions"

    Creation of a token for communication with a DataProxy requires at least ADMIN permissions on the resource the service account has set its permission on.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a token which can be used to directly fetch S3 credentials from a DataProxy
    curl '
      {
        "context": {
          "s3Credentials": true,
          "copy": {}
        }
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/proxy_tokens/{endpoint-id}
    ```

        ```bash linenums="1"
    # Native JSON request to create a token which can be used to initiate data replication 
    curl '
      {
        "context": {
          "s3Credentials": false,
          "copy": {
            "resource": "<resource-id>",
             "targetEndpoint": "<endpoint-id>",
             "push": true
          }
        }
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}/proxy_tokens/{endpoint-id}
    ```
-->


## Delete service account

API examples of how to delete a service account.

??? Abstract "Required permissions"

    Service account deletion requires at least ADMIN permissions on the resource the service account has set its permission on.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a service account
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/service_accounts/{svc-account-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a service account
    let request = DeleteServiceAccountRequest {
        svc_account_id: "<svc_account_id>".to_string()
    };

    // Send the request to the AOS instance gRPC gateway
    let response = self.service_account_client.delete_service_account(request)
                                              .await
                                              .unwrap()
                                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a service account
    request = DeleteServiceAccountTokenRequest(
        svc_account_id="<svc-account-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.service_account_client.DeleteServiceAccount(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
