
# How to use the Project API / ProjectServiceClient

## Introduction

Here you get a quick rundown how to basically create, read, update and delete Projects within the AOS.

This should be the first step after gaining access to the storage and creating an API token which is described in the previous chapters.


## Create Project

API example for creating a new Project.

!!! Note

    Currently, to create a new Project you have to be an AOS instance administrator.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a project
    curl -d '
      {
        "name": "curl-api-test-project", 
        "description": "This project was created with a cURL request."
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a project
    let create_request = CreateProjectRequest {
        name: "rust-api-test-project".to_string(),
        description: "This project was created with the gRPC Rust API client.".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.create_project(create_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a project
    request = CreateProjectRequest(
        name="python-api-test-project",
        description="This project was created with the gRPC Python API client."
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.CreateProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get Project

API example for fetching info of an existing Project.

!!! Info

    This request needs at least READ permissions on the specific Project.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of a project
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of a project
    let get_request = GetProjectRequest { 
        project_id: "<project-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.get_project(get_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of a project
    request = GetProjectRequest(
        project_id="<project-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.GetProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get all Projects of User

API example for fetching all registered Projects a specific user is associated with.

!!! Note

    You have to be an AOS instance administrator to fetch all registered Projects associated with other users than yourself.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch information of all projects a specific user is member of
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user/{user-id}/projects
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch information of all projects a specific user is member of
    let get_request = GetUserProjectsRequest {
        user_id: "".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_user_projects(get_request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}\n", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch information of all projects a specific user is member of
    request = GetUserProjectsRequest(
        user_id=""  # Parameter can be omitted if empty
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetUserProjects(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Update Project

API example for updating an existing Project.

!!! Info

    This request needs at least ADMIN permissions on the specific Project.

!!! Warning 

    **A project update overwrites all the fields in the request, even if they're empty. 
    If you want to retain a field you have to explicitly set the old value.**

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to update the metadata of a project
    curl -d '
      {
        "name": "curl-api-test-project", 
        "description": "This project was updated with a cURL request."
      }' \
         -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PUT https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to update the metadata of a project
    let update_request = UpdateProjectRequest {
        project_id: "<project-id>".to_string(),
        name: "rust-api-test-project".to_string(),
        description: "This project was updated with the gRPC Rust API client.".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.update_project(update_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to update the metadata of a project
    request = UpdateProjectRequest(
        project_id="<project-id>",
        name="python-api-test-project",
        description="This project was updated with the gRPC Python API client."
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.UpdateProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Delete Project

API examples for deleting a Project. 

!!! Note

    Before a project can be deleted, all collections within the project must have been deleted or moved to other projects. 
    In other words, the project must be empty.

!!! Info

    This request needs at least ADMIN permissions on the specific Project.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a project
    curl -H 'Authorization: Bearer <API_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a project
    let delete_request = DestroyProjectRequest {
        project_id: "<project-id>".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = project_client.destroy_project(delete_request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a project
    request = DestroyProjectRequest(
        project_id="<project-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.project_client.DestroyProject(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
