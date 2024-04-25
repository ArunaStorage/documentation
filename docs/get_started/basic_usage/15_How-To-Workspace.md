
# How to use the Workspace API / WorkspaceServiceClient


## Introduction

Workspaces are a dedicate class of Projects that can be used to provide unregistered users with an anonymous, encapsulated space in Aruna.
When a workspace is created, an internal service account is automatically created for the administration. 
Also token is generated for the service account that only has permissions for the workspace and cannot be changed.

After a user has registered in Aruna, he/she is free to use this token to claim the entire Workspace and associate it with his or her user like a normal Project. 

Templates for Workspaces can be created in advance. 
Through a template, the basic conditions for the creation of a *Workspace* are defined and automated in order to simplify the administration of Workspaces. 
The following attributes can be defined by a template:

- **owner:** The user id of the owner of the data, as long as it is located in the anonymous workspace
- **prefix:** Prefix that is placed in front of the name when resources in the Workspace are created: `<prefix>-<resource-name>`
- **hook_ids:** Ids of the hooks to which the workspace is assigned
- **endpoint_ids:** Ids of the endpoints that are allowed for the data storage of the Workspace

As long as data is located within an anonymous Workspace, it is not included in the search index. 
However, as soon as the data is claimed by a user and it is available with the data class Public/Private, it is also subsequently entered in the search index.


## Create Workspace

API examples of how to create a new Workspace.

??? Abstract "Required permissions"

    To create a new Workspace you only have to be a registered Aruna user.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new Workspace
    curl '
      {
        "workspaceTemplate": "<workspace-template-id>",
        "description": "<workspace-description>"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a new Workspace
    let request = CreateWorkspaceRequest {
        workspace_template: "<workspace-template-id>".to_string(),
        description: "<workspace-description>".to_string(),
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.create_workspace(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

## Create Workspace template

API examples of how to create a new Workspace.

??? Abstract "Required permissions"

    To create a new Workspace template you do not need special permissions but have to use a personal token.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new Workspace template
    curl '
      {
        "ownerId": "<user-id>",
        "prefix": "<name-prefix>",
        "name": "<template-name>",
        "hookIds": [
            "<hook-id-01>",
            "<hook-id-02>",
            "<...>"
        ],
        "description": "<template-description>",
        "endpointIds": [
            "<endpoint-id-01>",
            "<endpoint-id-02>",
            "<...>"
        ]
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces/templates
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a new Workspace template
    let request = CreateWorkspaceTemplateRequest {
        owner_id: "<user-id>".to_string(),
        prefix: "<name-prefix>".to_string(),
        name: "<template-name>".to_string(),
        hook_ids: vec![
            "<hook-id-01>".to_string(),
            "<hook-id-02>".to_string(),
            "<...>".to_string()
        ],
        description: "<template-description>".to_string(),
        endpoint_ids: vec![
            "<endpoint-id-01>".to_string(),
            "<endpoint-id-02>".to_string(),
            "<...>".to_string()
        ],
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.create_workspace_template(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```


## Get Workspace template

API examples of how to get a Workspace template.

??? Abstract "Required permissions"

    To fetch a Workspace template you do not need special permissions but have to use a personal token.

    Only global Aruna administrators can fetch information of Workspace templates created by other users.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a specific Workspace template
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces/templates/{template-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a specific Workspace template
    let request = GetWorkspaceTemplateRequest { 
        template_id: "<template-id>".to_string() 
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.get_workspace_template(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```


## List owned Workspace templates

API examples of how to get all Workspace templates owned by yourself.

??? Abstract "Required permissions"

    To fetch all your Workspace templates you do not need special permissions but have to use a personal token.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of all your Workspace templates
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces/templates
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of all your Workspace templates
    let request = ListOwnedWorkspaceTemplatesRequest {};

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.list_owned_workspace_templates(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```


## Claim Workspace

API examples of how to claim an anonymous Workspace for your user.

??? Abstract "Required permissions"

    To claim a workspace you do not need special permissions but you have to use the specific Workspace token.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to claim a Workspace for your user
    curl '
      {
        "token": "<workspace-token>"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces/{workspace-id}/claim
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI to claim a Workspace for your user
    let request = ClaimWorkspaceRequest { 
        workspace_id: "<workspace-id>".to_string(), 
        token: "<workspace-token>".to_string(),
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.claim_workspace(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```


## Delete Workspace template

API examples of how to delete a Workspace template.

??? Abstract "Required permissions"

    To delete a Workspace template you do not need special permissions but have to use a personal token.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request delete a Workspace template
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces/template/{template-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request delete a Workspace template
    let request = DeleteWorkspaceTemplateRequest { 
        template_id: "<template-id>".to_string() 
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.delete_workspace_template(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

## Delete Workspace

API examples of how to delete a Workspace.

??? Abstract "Required permissions"

    This request requires at least APPEND permissions on the specific Workspace.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request delete a Workspace
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-Aruna-instance-API-endpoint>/v2/workspaces/{workspace-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request delete a Workspace
    let request = DeleteWorkspaceRequest { 
        workspace_id: "<workspace-id>".to_string() 
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = workspace_client.delete_workspace(request)
                                   .await
                                   .unwrap()
                                   .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```
