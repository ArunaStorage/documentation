
# How to use the Project API / ProjectServiceClient

## Introduction

Here you get a quick rundown how to basically create, read, update and delete Projects within the AOS.
This should be the first step after gaining access to the storage and creating an API token which is described in the previous chapters: 

* [**Get Storage Access**](01_Get-Storage-Access.md)
* [**How to Authorize**](02_How-To-Auth-Tokens.md)


## Create Project

API example for creating a new Project.

Currently, to create a new Project you have to be an AOS instance administrator.

### Bash:
```bash
# Native JSON request to create a project
curl -d '
  {
    "name": "DummyProject", 
    "description": "This is a new dummy project."
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project
```

### Rust:
```rust
// Create tonic/ArunaAPI request to create a project
let create_request = CreateProjectRequest {
    name: "Rust-API-Test-Project".to_string(),
    description: "This project was created through the Rust API.".to_string(),
};

// Send the request to the AOS instance gRPC gateway
let response = project_client.create_project(create_request)
                             .await
                             .unwrap()
                             .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Get Project

API example for fetching info of an existing Project.

This request needs at least READ permissions on the specific Project.

### Bash:
```bash
# Native JSON request to fetch information of a project
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
```

### Rust:
```rust
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


## Get all Projects of User

API example for fetching all registered Projects a specific user is associated with.

You have to be an AOS instance administrator to fetch all registered Projects associated with other users than yourself.

### Bash:
```bash
# Native JSON request to fetch information of all projects a specific user is member of
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user/{user-id}/projects
```

### Rust:
```rust
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


## Update Project

API example for updating an existing Project.

This request needs at least ADMIN permissions on the specific Project.

> :warning: **A project update overwrites all the fields in the request, even if they're empty.**

### Bash:
```bash
# Native JSON request to update the metadata of a project
curl -d '
  {
    "name": "Updated Dummy Project", 
    "description": "This is an updated dummy project."
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X PUT https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
```

### Rust:
```rust
// Create tonic/ArunaAPI request to update the metadata of a project
let update_request = UpdateProjectRequest {
    project_id: "<project-id>".to_string(),
    name: "Rust-API-Test-Project".to_string(),
    description: "This project was updated through the Rust API.".to_string(),
};

// Send the request to the AOS instance gRPC gateway
let response = project_client.update_project(update_request)
                             .await
                             .unwrap()
                             .into_inner();

// Do something with the response
println!("{:#?}", response);
```


## Delete Project

API examples for deleting a Project. 

The following conditions have to be met before a Project can be deleted:
* The Project has to be empty (all Collections have to be deleted/moved)

This request needs at least ADMIN permissions on the specific Project.

### Bash:
```bash
# Native JSON request to delete a project
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
```

### Rust:
```rust
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