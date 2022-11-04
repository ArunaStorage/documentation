
# How to deal with tokens and permissions

## Introduction

The only actions that can be executed with your OIDC token are the user registration, fetching user info and API token creation/deletion.

For every action afterwards inside the AOS the user needs a generated API token with sufficient permissions.
Permissions can be granted either through scoped API tokens for projects or collections themselves or through user specific permissions which will be enforced by a global/personal token.


## Generate API token

An API token can be created with different scopes and/or different permissions.

### Available token permissions

* `NONE` ("PERMISSION_NONE"): No permissions granted
* `READ` ("PERMISSION_READ"): Read only access
* `APPEND` ("PERMISSION_APPEND"): Can create new resources but cannot modify existing
* `MODIFY` ("PERMISSION_MODIFY"): Can create new resources and modify existing
* `ADMIN` ("PERMISSION_ADMIN"): Can create new resources, modify existing and additionally delete

So when we talk about minimum requirements for authorization, we get the following order:
`ADMIN > MODIFY > APPEND > READ`

### Available API token scopes

**Global/Personal**

: The fields `projectId` and `collectionId` are empty on creation. 
This token is valid with nearly every request and inherits the permissions which are set user-specific on projects. 
For example, when a user is added to a project with READ permission, 
this token "inherits and enforces" the user's READ permission with every request regarding the project or its resources.

**Project**: 

: The field `projectId` is filled and the field `collectionId` is empty. 
This token is valid for the specific Project and all the resources which are associated with it. 
These tokens can be used to give general access to a Project and all resources registered under it, however, should not be distributed carelessly.

**Collection**: 

: The field `collectionId` is filled and the field `projectId` is empty.
This token is valid only for the specific Collection and its containing Objects/ObjectGroups. 
These tokens can be used to give users access to a more specific selection of Collections.


## Add users to Project

Users can be granted specific permissions for projects, which are inherited and enforced by their global/personal tokens. 
This makes it easy to add users to projects without them having to create an additional token per project or even collection. 
It also makes it easy to restrict or extend a user's permissions for a project without having to revoke, re-generate and/or re-distribute tokens.

!!! Info

    This request needs at least ADMIN permissions on the specific Project.

=== ":simple-curl: cURL"

    ```bash
    # Native JSON request to add user with admin permissions to a project
    curl -d '
      {
        "userPermission": {
          "userId": "<user-id>",
          "projectId": "<project-id>",
          "permission": "PERMISSION_ADMIN"
        }
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project/<project-id>/add_user
    ```
    
    ```bash
    # Native JSON request to add user with read only permissions to a project
    curl -d '
      {
        "userPermission": {
          "userId": "<user-id>",
          "projectId": "<project-id>",
          "permission": "PERMISSION_READ"
        }
      }
    ' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project/<project-id>/add_user
    ```

=== ":simple-rust: Rust"

    ```rust
    // Create tonic/ArunaAPI request to add user with admin permissions to a project
    let add_request = AddUserToProjectRequest {
        project_id: "<project-id>".to_string(),
        user_permission: Some(ProjectPermission {
            user_id: "<user-id>".to_string(),
            display_name: "".to_string(),
            project_id: "<project-id>".to_string(),
            permission: Permission::Admin as i32,
        }),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.add_user_to_project(add_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust
    // Create tonic/ArunaAPI request to add user with read only permissions to a project
    let add_request = AddUserToProjectRequest {
        project_id: "<project-id>".to_string(),
        user_permission: Some(ProjectPermission {
            user_id: "<user-id>".to_string(),
            display_name: "".to_string(),
            project_id: "<project-id>".to_string(),
            permission: Permission::Read as i32,
        }),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.add_user_to_project(add_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to add user with admin permissions to a project
    request = AddUserToProjectRequest(
        project_id="<project-id>",
        user_permission=ProjectPermission(
            user_id="<user-id>",
            project_id="<project-id>",
            permission=Permission.Value("PERMISSION_ADMIN")  # Needs int, therefore .Value()
        )
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.AddUserToProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python
    # Create tonic/ArunaAPI request to add user with read only permissions to a project
    request = AddUserToProjectRequest(
        project_id="<project-id>",
        user_permission=ProjectPermission(
            user_id="<user-id>",
            project_id="<project-id>",
            permission=Permission.Value("PERMISSION_READ")   # Needs int, therefore .Value()
        )
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.AddUserToProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Edit Project user permission

The assigned permissions to the users can be changed by project administrators afterwards.

!!! Info

    This request needs at least ADMIN permissions on the specific Project.

=== ":simple-curl: cURL"

    ```bash
    # Native JSON request to set a users permission to read only for the specific project
    curl -d '
      {
        "userPermission": {
          "userId": "<user-id>",
          "projectId": "<project-id>",
          "permission": "PERMISSION_READ"
        }
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project/<project-id>/edit_user
    ```

=== ":simple-rust: Rust"

    ```rust
    // Create tonic/ArunaAPI request to set a users permission to read only for the specific project
    let edit_request = EditUserPermissionsForProjectRequest {
        project_id: "<project-id>".to_string(),
        user_permission: Some(ProjectPermission {
            user_id: "<user-id>".to_string(),
            display_name: "".to_string(),
            project_id: "<project-id>".to_string(),
            permission: Permission::Read as i32,
        }),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.edit_user_permissions_for_project(edit_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to set a users permission to read only for the specific project
    request = EditUserPermissionsForProjectRequest(
        project_id="<project-id>",
        user_permission=ProjectPermission(
            user_id="<user-id>",
            project_id="<project-id>",
            permission=Permission.Value("PERMISSION_READ")  # Needs int, therefore .Value()
        )
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.EditUserPermissionsForProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Remove Project user

Users can, of course, also be completely removed from projects again, depriving them of any access with personalized tokens. 

However, access with project/collection scoped tokens is not restricted with the removal of the user.

!!! Info

    This request needs at least ADMIN permissions on the specific Project.

=== ":simple-curl: cURL"

    ```bash
    # Native JSON request to remove a user from a specific project
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project/<project-id>/remove_user?userId=<user-id>
    ```

=== ":simple-rust: Rust"

    ```rust
    // Create tonic/ArunaAPI request to remove a user from a specific project
    let delete_request = RemoveUserFromProjectRequest {
        project_id: "<project-id>".to_string(),
        user_id: "<user-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.remove_user_from_project(edit_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to remove a user from a specific project
    request = RemoveUserFromProjectRequest(
        project_id="<project-id>",
        user_id="<user-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.RemoveUserFromProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Generate API Tokens

Here are some API examples on generating API tokens with individual scopes and permissions.

!!! Warning

    **The token secret is only available once in the response and cannot be re-generated!**

    Store the received token secret in a secure location for further usage.
    If a token secret is lost or compromised, delete the old token and generate a new one.

=== ":simple-curl: cURL"

    ```bash
    # Native JSON request to create a global/personal token
    #  This token inherits the permissions from the projects the user is a member of
    curl -d '
      {
        "projectId": "",
        "collectionId": "",
        "name": "MyPersonalToken",
        "expiresAt": {
          "timestamp": "2023-01-01T00:00:00.000Z"
        }
      }' \
         -H 'Authorization: Bearer <OIDC-or-API_token' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/auth/token
    ```
    
    ```bash
    # Native JSON request to create a project scoped token with MODIFY permissions
    curl -d '
      {
        "projectId": "<project-id>",
        "collectionId": "",
        "name": "Project-Modify-Token",
        "expiresAt": {
          "timestamp": "2023-01-01T00:00:00.000Z"
        },
        "permission": "PERMISSION_MODIFY"
      }' \
         -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/auth/token
    ```
    
    ```bash
    # Native JSON request to create a collection scoped token with READ permissions
    curl -d '
      {
        "projectId": "",
        "collectionId": "<collection-id>",
        "name": "Collection-ReadOnly-Token",
        "expiresAt": {
          "timestamp": "2023-01-01T00:00:00.000Z"
        },
        "permission": "PERMISSION_READ"
      }' \
         -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/auth/token
    ```

=== ":simple-rust: Rust"

    The "in-line" conversion of a NativeDateTime to a gRPC Timestamp is kind of hacky.
    
    In any case it is recommended to implement the `From<NativeDateTime>` or `TryFrom<NativeDateTime>` trait for the Timestamp Struct.

    ```rust
    // Create tonic/ArunaAPI request to create a global/personal API token with expiration date
    let expires_at = NaiveDate::from_ymd(2023, 01, 01).and_hms(0, 0, 0);
    let create_request = CreateApiTokenRequest {
        project_id: "".to_string(), 
        collection_id: "".to_string(),
        name: "MyPersonalToken".to_string(),
        expires_at: Some(ExpiresAt {
            timestamp: Some(
                Timestamp::date_time(
                    expires_at.date().year().into(),
                    expires_at.date().month() as u8,
                    expires_at.date().day() as u8,
                    expires_at.time().hour() as u8,
                    expires_at.time().minute() as u8,
                    expires_at.time().second() as u8,
                ).unwrap(), 
            ),
        }),
        permission: Permission::None as i32,
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.create_api_token(create_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to create a global/personal API token with expiration date
    request = CreateAPITokenRequest(
        project_id="",  # Parameter can also be omitted if empty
        collection_id="",  # Parameter can also be omitted if empty
        name="MyPersonalToken",
        expires_at=ExpiresAt(
            timestamp=Timestamp(seconds=int(datetime.datetime(2023, 1, 1).timestamp()))
        ),
        permission=Permission.Value("PERMISSION_NONE")  # Parameter can also be omitted for personal tokens
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.CreateAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python
    # Create tonic/ArunaAPI request to create a project scoped API token with MODIFY permission
    request = CreateAPITokenRequest(
        project_id="<project-id>",
        collection_id="",  # Parameter can also be omitted if empty
        name="ProjectReadOnly",
        expires_at=None,  # Parameter can also be omitted if None
        permission=Permission.Value("PERMISSION_MODIFY")
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.CreateAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python
    # Create tonic/ArunaAPI request to create a collection scoped API token with APPEND permission
    request = CreateAPITokenRequest(
        project_id="",  # Parameter can also be omitted if empty
        collection_id="<collection-id>",  
        name="ProjectReadOnly",
        expires_at=None,  # Parameter can also be omitted if None
        permission=Permission.Value("PERMISSION_APPEND")
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.CreateAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get API token(s)

API examples to fetch info of a specific token or all tokens of the current user.

!!! Info

    This request does not re-display the generated API token secret. See [Generate API Tokens](#generate-api-tokens).

=== ":simple-curl: cURL"

    ```bash
    # Native JSON request to get info on a specific API token by its id
    curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/auth/token/{token-id}
    ```
    
    ```bash
    # Native JSON request to get info on all tokens associated with the current user
    curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/auth/tokens
    ```

=== ":simple-rust: Rust"

    ```rust
    // Create tonic/ArunaAPI request to get info on a specific API token by its id
    let get_request = GetApiTokenRequest { 
        token_id: "<token-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_api_token(get_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust
    // Create tonic/ArunaAPI request to get info on all tokens associated with the current user
    let get_request = GetApiTokensRequest {};
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_api_tokens(get_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to get info on a specific API token by its id
    request = GetAPITokenRequest(
        token_id="<token-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python
    # Create tonic/ArunaAPI request to get info on all tokens associated with the current user
    request = GetAPITokensRequest()
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetAPITokens(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Revoke token(s)

API examples to revoke/delete a specific API token or all tokens of the current user.

!!! Note

    Only AOS instance administrators can revoke API tokens of other users.

=== ":simple-curl: cURL"

    ```bash
    # Native JSON request to revoke the specific API token
    curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/auth/token/{token-id}
    ```
    
    ```bash
    # Native JSON request to revoke all tokens of the current user
    curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/auth/tokens
    ```

=== ":simple-rust: Rust"

    ```rust
    // Create tonic/ArunaAPI request to revoke the specific API token
    let delete_request = DeleteApiTokenRequest {
        token_id: "<token-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.delete_api_token(delete_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
    
    ```rust
    // Create tonic/ArunaAPI request to to revoke all tokens of the current user
    let delete_request = DeleteApiTokensRequest {
        user_id: "".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.delete_api_tokens(delete_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to revoke the specific API token
    request = DeleteAPITokenRequest(
        token_id="<token-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.DeleteAPIToken(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

    ```python
    # Create tonic/ArunaAPI request to to revoke all tokens of the current user
    request = DeleteAPITokensRequest()
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.DeleteAPITokens(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
