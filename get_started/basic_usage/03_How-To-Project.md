
# How to use the Project API / ProjectServiceClient

## Introduction

Here you get a quick rundown how to basically create, read, update and delete Projects within the AOS.
This should be the first step after gaining access to the storage and creating an API token which is described in the previous chapters: 

* [**Get Storage Access**](01_Get-Storage-Access.md)
* [**How to Authorize**](02_How-To-Auth-Tokens.md)


## Create Project

API example for creating a new Project:

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


## Get Project

API examples for fetching info fo an existing Project:

### Bash:
```bash
# Native JSON request to fetch information of a project
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
```


## Get all Projects of User

API examples for fetching all registered Projects a specific user is associated with:

### Bash:
```bash
# Native JSON request to fetch information of all projects a specific user is member of
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user/{user-id}/projects
```


## Update Project

API examples for updating an existing Project.

> :warning: **A project update overwrites all the fields in the request, even if they're empty.**

### Bash:
```bash
# Native JSON request to update a projects metadata
curl -d '
  {
    "name": "Updated Dummy Project", 
    "description": "This is an updated dummy project."
  }' \
     -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X PUT https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
```


## Delete Project

API examples for deleting a Project. 

The following conditions have to be met before a Project can be deleted:
* The Project has to be empty (all Collections have to be deleted/moved)
* You need admin permissions on the Project or have to be an AOS administrator

### Bash:
```bash
# Native JSON request to delete a project
curl -H 'Authorization: Bearer <API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X DELETE https://<URL-to-AOS-instance-API-gateway>/v1/project/{project-id}
```
