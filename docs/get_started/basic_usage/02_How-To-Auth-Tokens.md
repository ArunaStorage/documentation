
# How to deal with tokens and permissions

## Introduction

Every action inside the AOS is authenticated and authorized via tokens which have to be provided in the request header: `Authorization: Bearer <token>`.

This can be a either a generated API token or your OIDC token which you received after authentication against the OIDC provider supported by AOS. API tokens have the advantage that they can not only be used to authorise user-specific permissions, but can also be explicitly generated with permissions for a specific resource. In addition, the exact expiry date of an API token can be defined.

For data protection reasons, a user must also register with each Dataproxy with which they wish to interact. By registering once, the DataProxy receives permission to synchronise information about the user. If a user wants to communicate directly with a dataproxy, he or she must request S3 credentials from/for the dataproxy in advance, which also counts as registration.


## API tokens

An API token can be created with different scopes and/or different permissions.

### Available token permissions

* `ADMIN` ("PERMISSION_LEVEL_ADMIN"): Can create new resources, modify existing and additionally delete
* `WRITE` ("PERMISSION_LEVEL_WRITE"): Can create new resources and modify existing
* `APPEND` ("PERMISSION_LEVEL_APPEND"): Can create new resources but cannot modify existing
* `READ` ("PERMISSION_LEVEL_READ"): Read only access
* `NONE` ("PERMISSION_LEVEL_NONE"): No permissions granted

So when we talk about minimum requirements for authorization, we get the following order:
`ADMIN > WRITE > APPEND > READ > NONE`

### Available API token scopes

**Global/Personal**

: The field `permission` is empty on creation. 
This token is valid with nearly every request against the ArunaServer and inherits the permissions which are set user-specific on resources. 
For example, when a user is added to a Project with READ permission, 
this token "inherits and enforces" the user's READ permission with every request regarding the specific Project and its underlying resources. 

    !!! Warning

        These tokens should only be used by the user itself as they are bound to the permissions of the user who created it!

**Resource**: 

: The field `permission` is filled with a valid ULID of an existing AOS resource and the corresponding permission level.
This token is valid for the specific resource and all the resources which are registered beneath it. 
For example, these tokens can be used to give external users general but time limited access to a Project and all resources registered under it. However, these tokens should also not be distributed carelessly.


### Generate API tokens

An API token can be created with different scopes and/or different permissions.

!!! Warning

    **The token secret is only available once in the response and cannot be re-generated!**

    Store the received secret keys in a secure location for further usage.
    If a token secret is lost or compromised, delete the old token and generate a new one.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a global/personal token.
    #  This token inherits the permissions from the resources the user has been granted permissions for.
    curl -d '
      {
        "name": "<token-display-name>",
        "expiresAt": "2024-01-01T00:00:00.000Z"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/user/token
    ```

    ```bash linenums="1"
    # Native JSON request to create a project scoped token with WRITE permissions
    curl -d '
      {
        "name": "<token-display-name>",
        "permission": {
          "projectId": "<project-id>",
          "collectionId": "",
          "datasetId": "",
          "objectId": "",
          "permissionLevel": "PERMISSION_LEVEL_WRITE"
        },
        "expiresAt": "2024-01-01T00:00:00.000Z"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/user/token
    ```

    ```bash linenums="1"
    # Native JSON request to create a collection scoped token with READ permissions
    curl -d '
      {
        "name": "<token-display-name>",
        "permission": {
          "projectId": "",
          "collectionId": "<collection-id>",
          "datasetId": "",
          "objectId": "",
          "permissionLevel": "PERMISSION_LEVEL_READ"
        },
        "expiresAt": "2024-01-01T00:00:00.000Z"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/user/token
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a global/personal API token with expiration date
    let request = CreateApiTokenRequest {
        name: "<token-display-name>".to_string(),
        permission: None,
        expires_at: Some(
            NaiveDate::from_ymd_opt(2030, 01, 01)
                .unwrap()
                .and_hms_opt(0, 0, 0)
                .unwrap()
                .into(),
        ),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.create_api_token(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a project scoped token with WRITE permissions
    let request = CreateApiTokenRequest {
        name: "<token-display-name>".to_string(),
        permission: Some(Permission {
            permission_level: PermissionLevel::Write as i32,
            resource_id: Some(ResourceId::ProjectId("<project-id>".to_string())),
        }),
        expires_at: Some(
            NaiveDate::from_ymd_opt(2030, 01, 01)
                .unwrap()
                .and_hms_opt(0, 0, 0)
                .unwrap()
                .into(),
        ),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.create_api_token(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a collection scoped token with READ permissions
    let request = CreateApiTokenRequest {
        name: "<token-display-name>".to_string(),
        permission: Some(Permission {
            permission_level: PermissionLevel::Read as i32,
            resource_id: Some(ResourceId::CollectionId("<collection-id>".to_string())),
        }),
        expires_at: Some(
            NaiveDate::from_ymd_opt(2030, 01, 01)
                .unwrap()
                .and_hms_opt(0, 0, 0)
                .unwrap()
                .into(),
        ),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.create_api_token(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a global/personal API token with expiration date
    request = CreateAPITokenRequest(
        name="<token-display-name>",
        expires_at=Timestamp(seconds=int(datetime.datetime(2030, 1, 1).timestamp()))
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.CreateAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a project scoped API token with WRITE permission
    request = CreateAPITokenRequest(
        name="<token-display-name>",
        permission=Permission(
            project_id="<project-id>",
            permission_level="PERMISSION_LEVEL_WRITE"
        ),
        expires_at=Timestamp(seconds=int(datetime.datetime(2030, 1, 1).timestamp()))
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.CreateAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a collection scoped API token with READ permission
    request = CreateAPITokenRequest(
        name="<token-display-name>",
        permission=Permission(
            collection_id="<collection-id>",
            permission_level="PERMISSION_LEVEL_READ"
        ),
        expires_at=Timestamp(seconds=int(datetime.datetime(2030, 1, 1).timestamp()))
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.CreateAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Get API token(s)

Meta information of tokens can be fetched after creation e.g. to check its expiry date.

!!! Info

    This request does not re-display the generated API token secret. See [Generate API Tokens](#generate-api-tokens).

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get info on a specific API token by its id
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/auth/token/{token-id}
    ```
    
    ```bash linenums="1"
    # Native JSON request to get info on all tokens associated with the current user
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/user/tokens
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to get info on a specific API token by its id
    let request = GetApiTokenRequest { 
        token_id: "<token-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_api_token(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to get info on all tokens associated with the current user
    let request = GetApiTokensRequest {};

    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_api_tokens(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to get info on a specific API token by its id
    request = GetAPITokenRequest(
        token_id="<token-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to get info on all tokens associated with the current user
    request = GetAPITokensRequest()
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetAPITokens(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Revoke token(s)

API examples of how to revoke/delete a specific API token or all tokens of the current user.

!!! Note

    Only AOS instance administrators can revoke API tokens of other users.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to revoke the specific API token
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/user/token/{token-id}
    ```

    ```bash linenums="1"
    # Native JSON request to revoke all tokens of the current user
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/user/tokens
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to revoke the specific API token
    let request = DeleteApiTokenRequest {
        token_id: "<token-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.delete_api_token(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to to revoke all tokens of the current user
    let request = DeleteApiTokensRequest {
        user_id: "".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.delete_api_tokens(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to revoke the specific API token
    request = DeleteAPITokenRequest(
        token_id="<token-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.DeleteAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to to revoke all tokens of the current user
    request = DeleteAPITokensRequest()
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.DeleteAPITokens(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Get S3 credentials

The Aruna DataProxy implements an [S3 compatible interface](11_How-To-S3-Interface.md){target="_blank"} that implements a basic 
subset of the S3 functionality and can be used with any client that makes use of the S3 protocol. Before the interface can be 
used for uploading and downloading data, a must have fetched S3 credentials at least once for the specific DataProxy.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to request S3 credentials for the specific endpoint
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET "https://<URL-to-AOS-instance-API-gateway>/v2/user/{user-id}/s3_credentials?endpointId={endpoint-id}"
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch S3 credentials for the specific endpoint
    let request = GetS3CredentialsUserRequest {
        user_id: "<user-id>".to_string(),
        endpoint_id: "<endpoint-id>".to_string(),
    };

    // Get/Create S3 credentials to register user at DataProxy
    let response = user_client.get_s3_credentials_user(request)
                              .await
                              .unwrap()
                              .into_inner();

    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to revoke the specific API token
    request = GetS3CredentialsUserRequest(
        user_id="<user-id>",
        endpoint_id="<endpoint-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetS3CredentialsUser(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Grant permissions

For hierarchical resources users can be granted specific permissions that are inherited and enforced by their global/personal tokens. 
This makes it easy to add users e.g. to Projects without having to create additional tokens. 

**An individual user can only have one specific permission granted per resource.**

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the specific resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to grant user admin permissions to a resource
    curl -d '
      {
        "resourceId": "<resource-id>",
        "userId": "<user-id>",
        "permission": "PERMISSION_LEVEL_ADMIN",
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/auth
    ```

    ```bash linenums="1"
    # Native JSON request to add user with read only permissions to a project
    curl -d '
      {
        "resourceId": "<resource-id>",
        "userId": "<user-id>",
        "permission": "PERMISSION_LEVEL_READ",
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/auth
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to grant user admin permissions to a resource
    let request = CreateAuthorizationRequest {
        resource_id: "<resource-id>".to_string(),
        user_id: "<user-id>".to_string(),
        permission_level: PermissionLevel::Admin as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = auth_client.create_authorization(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to add user with read only permissions to a project
    let request = CreateAuthorizationRequest {
        resource_id: "<resource-id>".to_string(),
        user_id: "<user-id>".to_string(),
        permission_level: PermissionLevel::Read as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = auth_client.create_authorization(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to grant user admin permissions to a resource
    request = CreateAuthorizationRequest(
        resource_id="<resource-id>",
        user_id="<user-id>",
        permission=PermissionLevel.Value("PERMISSION_LEVEL_ADMIN")  # Needs int, therefore .Value()
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.auth_client.CreateAuthorization(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python linenums="1"
    # Create tonic/ArunaAPI request to grant user read only permissions to a resource
    request = CreateAuthorizationRequest(
        resource_id="<resource-id>",
        user_id="<user-id>",
        permission=PermissionLevel.Value("PERMISSION_LEVEL_READ")  # Needs int, therefore .Value()
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.auth_client.CreateAuthorization(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Update permissions

The assigned permissions can also be modified afterwards.

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the specific resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to set a users permission to read only for the specific resource
    curl -d '
      {
        "resourceId": "<resource-id>",
        "userId": "<user-id>",
        "permission": "PERMISSION_LEVEL_READ",
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-AOS-instance-API-gateway>/v2/auth
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to set a users permission to read only for the specific resource
    let request = UpdateAuthorizationRequest {
        resource_id: "<resource-id>".to_string(),
        user_id: "<user-id>".to_string(),
        permission_level: PermissionLevel::Read as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = auth_client.update_authorization(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to set a users permission to read only for the specific resource
    request = UpdateAuthorizationRequest(
        resource_id="<resource-id>",
        user_id="<user-id>",
        permission=PermissionLevel.Value("PERMISSION_LEVEL_READ")  # Needs int, therefore .Value()
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.auth_client.UpdateAuthorization(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Remove permissions

Users can, of course, also be completely removed from resources again, depriving them of any access with personalized tokens. 

However, access with project/collection scoped tokens is not restricted with the removal of users personal permissions!

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the specific resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to remove a users permission for a specific resource
    curl -d '
      {
        "resourceId": "<resource-id>",
        "userId": "<user-id>",
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v2/auth
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to remove a users permission for a specific resource
    let request = DeleteAuthorizationRequest {
        resource_id: "<resource-id>".to_string(),
        user_id: "<user-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = auth_client.delete_authorization(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to remove a users permission for a specific resource
    request = DeleteAuthorizationRequest(
        resource_id="<project-id>",
        user_id="<user-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.auth_client.UpdateAuthorization(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## List permissions

All permissions which are assigned for a specific resource can also be listed. 

This request additionally offers the option to recursively fetch the permissions of all underlying resources.

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the specific resource.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to list all assigned permissions for a specific resource
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/auths?resourceId=resource-id
    ```

    ```bash linenums="1"
    # Native JSON request to recursively list all assigned permissions for a specific resource
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/auths?resourceId=resource-id&recursive=true
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to list all assigned permissions for a specific resource
    let request = GetAuthorizationsRequest {
        resource_id: "<resource-id>".to_string(),
        recursive: false, // Can be set to 'true' to also fetch permissions of underlying resources
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = auth_client.get_authorizations(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to list all assigned permissions for a specific resource
    request = DeleteAuthorizationRequest(
        resource_id="<project-id>",
        recursive=False # Can be set to 'true' to also fetch permissions of underlying resources
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.auth_client.GetAuthorizations(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
