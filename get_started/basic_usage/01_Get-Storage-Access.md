
# Get Storage Access

For the general access a registered account within the official NFDI/NFDI4Biodiversity/GFBio AAI is needed to obtain a valid OIDC token from one of those.

After login, you can use your OIDC token to register yourself at an AOS instance. 
To create/modify resources within the given scope/permissions you have to generate API token(s) after you have been activated by an administrator.

Please note that these following tutorials cover only the most basic operations. 
For a complete list of the public API endpoints, their matching requests and responses, please refer to our [Aruna Object Storage REST API Swagger-UI](https://api.aruna.nfdi-dev.gi.denbi.de/swagger-ui/).

Let's get started!


## User registration

Users can register themselves in an AOS instance with their valid OIDC token.

### Bash:
```bash
# Native JSON request to register OIDC user
curl -d '{"display_name": "Forename Surname"}' \
     -H "Authorization: Bearer <OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/auth/register
```


## User activation

After registration users additionally have to be activated in a second step.
Activation can only be performed by an AOS instance administrator.

### Bash:
```bash
# Native JSON request to activate registered user
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/user/<user-id>/activate

# For convenience, administrators can request info on all unactivated users at once
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user/unregistered
```


## Who Am I / What Am I

To check which user a token is associated with or get information about the current users permissions, you can use the UserService API.

Keep in mind that only administrators can request user information of other users.

### Bash:
```bash
# Native JSON request to fetch user information associated with authorization token
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user

# Native JSON request to fetch user information associated with provided user id
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X GET 'https://api.aruna.nfdi-dev.gi.denbi.de/v1/user?userId=<user-id>
```
