
# Get Storage Access

For the general access a registered account within the official NFDI/NFDI4Biodiversity/GFBio AAI is needed to obtain a valid OIDC token from one of those.

After login, you can use your OIDC token to register yourself at an AOS instance. 
To create/modify resources within the given scope/permissions you have to generate API token(s) after you have been activated by an administrator.

Please note that these following tutorials cover only the most basic operations. 
For a complete list of the public API endpoints, their matching requests and responses, please refer to our [Aruna Object Storage REST API Swagger-UI](https://api.aruna.nfdi-dev.gi.denbi.de/swagger-ui/).

Let's get started!


## Client Creation

If you plan to send your requests through an application the first step is to open a connection to the server you want to send the requests to.

**Note:** This is a minimal reproducible example which should not be used 'as-is' in a production environment...

### Rust:
```rust
use aruna_rust_api::api::aruna::api::storage::services::v1::user_service_client;
use aruna_rust_api::api::aruna::api::storage::services::v1::project_service_client;
use aruna_rust_api::api::aruna::api::storage::services::v1::collection_service_client;
use aruna_rust_api::api::aruna::api::storage::services::v1::object_service_client;
use aruna_rust_api::api::aruna::api::storage::services::v1::object_group_service_client;

use std::sync::Arc;
use tonic::codegen::InterceptedService;
use tonic::metadata::{AsciiMetadataKey, AsciiMetadataValue};
use tonic::transport::{Channel, ClientTlsConfig};

// Create a client interceptor which always adds the specified api token to the request header
#[derive(Clone)]
pub struct ClientInterceptor {
    api_token: String,
}
// Implement a request interceptor which always adds the authorization header with a specific API token to all requests
impl tonic::service::Interceptor for ClientInterceptor {
    fn call(&mut self, request: tonic::Request<()>) -> Result<tonic::Request<()>, tonic::Status> {
        let mut mut_req: tonic::Request<()> = request;
        let metadata = mut_req.metadata_mut();
        metadata.append(
            AsciiMetadataKey::from_bytes("Authorization".as_bytes()).unwrap(),
            AsciiMetadataValue::try_from(format!("Bearer {}", self.api_token.as_str())).unwrap(),
        );

        return Ok(mut_req);
    }
}

fn main() {
    // Create connection to AOS instance gRPC gateway
    let api_token   = "MySecretArunaApiToken".to_string();
    let tls_config  = ClientTlsConfig::new();
    let endpoint    = Channel::from_shared("https://<URL-To-AOS-Instance-gRPC-Gateway>").unwrap().tls_config(tls_config).unwrap();
    let channel     = endpoint.connect().await.unwrap();
    let interceptor = ClientInterceptor { api_token: api_token.clone() };

    // Create the individual client services
    let mut user_client         = user_service_client::UserServiceClient::with_interceptor(channel.clone(), interceptor.clone());
    let mut project_client      = project_service_client::ProjectServiceClient::with_interceptor(channel.clone(), interceptor.clone());
    let mut collection_client   = collection_service_client::CollectionServiceClient::with_interceptor(channel.clone(), interceptor.clone());
    let mut object_client       = object_service_client::ObjectServiceClient::with_interceptor(channel.clone(), interceptor.clone());
    let mut object_group_client = object_group_service_client::ObjectGroupServiceClient::with_interceptor(channel.clone(), interceptor.clone());
    
    // Do something with the client services ...
}
```

### Go:
```go
```

### Python:
```python
```

### Java:
```java
```

The presence of a client connection to the specific resource service is required for all further requests in this tutorial if the requests are send via gRPC.


## User registration

Users can register themselves in an AOS instance with their valid OIDC token received from the AAI login.

### Bash:
```bash
# Native JSON request to register OIDC user
curl -d '
  {
    "display_name": "Forename Surname"
  }' \
     -H "Authorization: Bearer <OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X POST https://<URL-to-AOS-instance-API-gateway>/v1/auth/register
```

### Rust:
```rust
// Create tonic/ArunaAPI request to register OIDC user
let register_request = RegisterUserRequest {
    display_name: "John Doe".to_string(),
};

// Send the request to the AOS instance gRPC gateway
let response = user_client.register_user(register_request)
                          .await
                          .unwrap()
                          .into_inner();

// Do something with the response
println!("Registered user: {:#?}", response.user_id)
```


## User activation

After registration users additionally have to be activated in a second step.
Activation can only be performed by an AOS instance administrator.

### Bash:
```bash
# For convenience, administrators can request info on all unactivated users at once
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user/not_activated
```

```bash
# Native JSON request to activate registered user
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X PATCH https://<URL-to-AOS-instance-API-gateway>/v1/user/<user-id>/activate
```


### Rust:
```rust
// Create tonic/ArunaAPI request to fetch all not activated users
let get_request = GetNotActivatedUsersRequest {};

// Send the request to the AOS instance gRPC gateway
let unactivated = user_client.get_not_activated_users(request)
                             .await
                             .unwrap()
                             .into_inner();

// Do something with the response
println!("{:#?}", unactivated);
```

```rust
// Create tonic/ArunaAPI request for user activation
let user_id = uuid::Uuid::parse("12345678-1234-1234-1234-123456789999").unwrap();

let activate_request = ActivateUserRequest {
    user_id: user_id.to_string()
};

// Send the request to the AOS instance gRPC gateway
let activate_response = user_client.activate_user(activate_request)
                                   .await
                                   .unwrap()
                                   .into_inner();

// Do something with the response
println!("Activated user: {:#?}", activate_response.user_id)
```
<!--
### Rust:
    ```rust
    // Create tonic/ArunaAPI request for user registration
    let get_request = tonic::Request::new(
        GetUserRequest {
            user_id: "user_oidc_external_id".to_string(),
        }
    );
    
    // Send the request to the server
    let get_response = user_client.get_user(get_request)
                                  .await
                                  .unwrap()
                                  .into_inner();
    
    // Do something with the response
    match get_response.user {
        None => panic!("Could not find registered user."),
        Some(user) => {
            // Activate registered user
            let activate_request = ActivateUserRequest {
                user_id: user.id,
            };
    
            let activate_response = user_client.activate_user(activate_request)
                                               .await
                                               .unwrap()
                                               .into_inner();
    
            println!("Activated user: {}", activate_response.user_id)
        }
    }
    ```
-->


## Who Am I / What Am I

To check which user a token is associated with or get information about the current users permissions, you can use the UserService API.

Only AOS instance administrators can request user information of other users.

### Bash:
```bash
# Native JSON request to fetch user information associated with authorization token
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user
```

```bash
# Native JSON request to fetch user information associated with the provided user id
curl -H "Authorization: Bearer <API-Or-OIDC_TOKEN>" \
     -H "Content-Type: application/json" \
     -X GET https://<URL-to-AOS-instance-API-gateway>/v1/user?userId=<user-id>
```

### Rust:
```rust
// Create tonic/ArunaAPI request to fetch user info of current user
let get_request = GetUserRequest {
    user_id: "".to_string()
};

// Send the request to the AOS instance gRPC gateway
let response: GetUserResponse = user_client.get_user(get_request)
                                           .await
                                           .unwrap()
                                           .into_inner();

// Do something with the response
println!("Received permission info for user: {:#?}", response.user);
println!("Received project permissions:");
for permission in response.project_permissions {
    println!("{:#?}", permission);
}
```
