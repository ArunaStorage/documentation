
# How to deal with tokens and permissions

## Introduction

The only actions that can be executed with your OIDC token are the user registration, fetching user info and API token creation/deletion.

For every action afterwards inside the AOS the user needs a generated API token with sufficient permissions.


## Create token

A token can be created with different scopes and/or different permissions.

### Available token scopes:
* **Global/Personal**:

The fields `projectId` and `collectionId` are empty. 
This token is valid with nearly every request if the permissions are sufficient. 
Therefore, these tokens should only be created for regular AOS users with a wide range of administrative duties.

* **Project**: 

The field `projectId` is filled and the field `collectionId` is empty.
This token is valid for the specific Project and all the resources which are associated with it.
These tokens can be used to give users general access to a Project, however, should not be distributed carelessly.

* **Collection**: 

The field `collectionId` is filled and the field `projectId` is empty.
This token is valid only for the specific Collection and its containing Objects/ObjectGroups. 
These tokens can be used to give users access to a more specific selection of Collections.


### Available token permissions:
* **NONE** ("PERMISSION_NONE"): No permissions granted
* **READ** ("PERMISSION_READ"): Read only access
* **APPEND** ("PERMISSION_APPEND"): Can create new resources but cannot modify existing
* **MODIFY** ("PERMISSION_MODIFY"): Can create new resources and modify existing
* **ADMIN** ("PERMISSION_ADMIN"): Can create new resources, modify existing and additionally delete


## Add users to Project

Users can be granted specific permissions for projects, which are inherited and enforced by their personalized tokens. 
This makes it easy to add users to projects without having to create an additional token per project for each user. 
It also makes it easy to restrict or extend a user's permissions for a project without having to revoke and regenerate tokens.

**Only project administrators can add other users to a project.** 

### Bash
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
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project/<project-id>/adduser

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
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/project/<project-id>/adduser
```


## Generate Tokens

### Bash:

Here are some API examples on generating API tokens with individual scopes and permissions.

**The token secret is only available once in the response and cannot be re-generated!**

Store the received token secret in a secure location for further usage.
If the token secret is lost or compromised delete the old token and generate a new one.

```bash
# Native JSON request to create a global/personal token
#  This token inherits the permissions from the projects the user is a member
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
# Native JSON request to create a project scoped token with modify permissions
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
# Native JSON request to create a collection scoped token with read only permissions
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


## Get token(s)

API examples to fetch info of a specific token or all tokens of the current user.

### Bash:
```bash
# Native JSON request to get info on a specific API token by its id
curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/auth/token/{token-id}
```

```bash
# Native JSON request to get info on a specific API token by its user defined name
curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/auth/token?name=<token-name>
```

```bash
# Native JSON request to get info on all tokens associated with the current user
curl -H 'Authorization: Bearer <OIDC-Or-API_TOKEN>' \
     -H 'Content-Type: application/json' \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/auth/tokens
```


## Revoke token(s)

API examples to revoke/delete a specific API token or all tokens of the current user.

### Bash:
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

