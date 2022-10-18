
# How to deal with tokens and permissions

## Introduction

The only actions that can be executed with your OIDC token are the user registration, fetching user info and API token creation/deletion.

For every action afterwards inside the AOS the user needs a generated API token with sufficient permissions.
Permissions can be granted either through scoped API tokens for projects or collections themselves or through user specific permissions which will be enforced by a global/personal token.


## Generate API token

An API token can be created with different scopes and/or different permissions.

### Available token permissions:
* `NONE` ("PERMISSION_NONE"): No permissions granted
* `READ` ("PERMISSION_READ"): Read only access
* `APPEND` ("PERMISSION_APPEND"): Can create new resources but cannot modify existing
* `MODIFY` ("PERMISSION_MODIFY"): Can create new resources and modify existing
* `ADMIN` ("PERMISSION_ADMIN"): Can create new resources, modify existing and additionally delete

So when we talk about minimum requirements for authorization, we get the following order:
`ADMIN > MODIFY > APPEND > READ`

### Available API token scopes:
* **Global/Personal**:

The fields `projectId` and `collectionId` are empty on creation.
This token is valid with nearly every request and inherits the permissions which are set user-specific on projects. 

For example, when a user is added to a project with READ permission, this token "inherits and enforces" the user's READ permission with every request regarding the project or its resources.

* **Project**: 

The field `projectId` is filled and the field `collectionId` is empty.
This token is valid for the specific Project and all the resources which are associated with it.

These tokens can be used to give general access to a Project and all resources registered under it, however, should not be distributed carelessly.

* **Collection**: 

The field `collectionId` is filled and the field `projectId` is empty.
This token is valid only for the specific Collection and its containing Objects/ObjectGroups. 

These tokens can be used to give users access to a more specific selection of Collections.


## Add users to Project

Users can be granted specific permissions for projects, which are inherited and enforced by their global/personal tokens. 
This makes it easy to add users to projects without them having to create an additional token per project or even collection. 
It also makes it easy to restrict or extend a user's permissions for a project without having to revoke, re-generate and/or re-distribute tokens.

This request needs at least ADMIN permissions on the specific Project.

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


## Generate API Tokens

### Bash:

Here are some API examples on generating API tokens with individual scopes and permissions.

**The token secret is only available once in the response and cannot be re-generated!**

Store the received token secret in a secure location for further usage.
If a token secret is lost or compromised, delete the old token and generate a new one.

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


## Get API token(s)

API examples to fetch info of a specific token or all tokens of the current user.

**This request does not re-display the generated API token secret.**

### Bash:
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


## Revoke token(s)

API examples to revoke/delete a specific API token or all tokens of the current user.

Only AOS instance administrators can revoke API tokens of other users.

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

