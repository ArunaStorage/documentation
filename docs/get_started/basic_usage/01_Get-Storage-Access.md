
# Get Storage Access

## Introduction

For the general access a registered account within the official NFDI/NFDI4Biodiversity/GFBio AAI is needed to obtain a valid OIDC token from one of those.

After login, you can use your OIDC token to register yourself at an AOS instance. 
To create/modify resources within the given scope/permissions you have to generate API token(s) after you have been activated by an AOS instance administrator.

Please note that these following tutorials cover only the most basic operations. 
For a complete list of the public API endpoints, their matching requests and responses, please refer to our [Aruna Object Storage REST API Swagger-UI](https://api.aruna.nfdi-dev.gi.denbi.de/swagger-ui/){:target="_blank"}.


## Client Creation

If you plan to send your requests through gRPC the first step is to open a connection to the gRPC gateway of the server you want to send the requests to.

The presence of a client connection to the specific resource service is required for all further requests in this tutorial if the requests are send via gRPC.

!!! Danger

    These are minimal reproducible examples only for demonstration purposes which should not be used 'as-is' in a production environment!

=== ":simple-rust: Rust"

    To use the Rust API library you have to set it as dependency `aruna-rust-api = "<Aruna-Rust-API-Version>"` in the `cargo.toml` of your project.

    ```rust
    use aruna_rust_api::api::aruna::api::storage::services::v1::{
        user_service_client,
        project_service_client,
        collection_service_client,
        object_service_client,
        object_group_service_client,
    }
    use std::sync::Arc;
    use tonic::codegen::InterceptedService;
    use tonic::metadata::{AsciiMetadataKey, AsciiMetadataValue};
    use tonic::transport::{Channel, ClientTlsConfig};
    
    // Create a client interceptor which always adds the specified api token to the request header
    #[derive(Clone)]
    pub struct ClientInterceptor {
        api_token: String,
    }
    // Implement a request interceptor which always adds 
    //  the authorization header with a specific API token to all requests
    impl tonic::service::Interceptor for ClientInterceptor {
        fn call(&mut self, request: tonic::Request<()>) -> Result<tonic::Request<()>, tonic::Status> {
            let mut mut_req: tonic::Request<()> = request;
            let metadata = mut_req.metadata_mut();
            metadata.append(
                AsciiMetadataKey::from_bytes("authorization".as_bytes()).unwrap(),
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

=== ":simple-python: Python"

    To use the Python API library in your Python project you have to install the PyPI package: `pip install Aruna-Python-API`.

    !!! note
    
        This example additionally implements an interceptor which adds the authorization token automatically to each request send by the service clients.

        It is a little more complicated and extra work up front but offers the advantage of having to worry less about the correct API token request metadata later.

        **All Python examples in this documentation assume that the client services have been initialized with an interceptor.**

    ```python
    import collections
    import grpc
    
    from aruna.api.storage.services.v1.collection_service_pb2_grpc import CollectionServiceStub
    from aruna.api.storage.services.v1.object_service_pb2_grpc import ObjectServiceStub
    from aruna.api.storage.services.v1.objectgroup_service_pb2_grpc import ObjectGroupServiceStub
    from aruna.api.storage.services.v1.project_service_pb2_grpc import ProjectServiceStub
    from aruna.api.storage.services.v1.user_service_pb2_grpc import UserServiceStub
    
    # Valid Aruna API token
    #   In a production environment this should be stored in a more secure location ...
    API_TOKEN = 'MySecretArunaApiToken'
    
    # AOS instance gRPC gateway endpoint
    AOS_HOST = '<URL-To-AOS-Instance-gRPC-Gateway>' # Protocol (e.g. https://) has to be omitted
    AOS_PORT = '443'


    class _MyAuthInterceptor(grpc.UnaryUnaryClientInterceptor):
        """
        Implement abstract class grpc.UnaryUnaryClientInterceptor to extend request metadata.
        """
        def intercept_unary_unary(self, continuation, client_call_details, request):
            # Append authorization token to request metadata
            metadata = []
            if client_call_details.metadata is not None:
               metadata = list(client_call_details.metadata)
            metadata.append(('authorization', f'Bearer {API_TOKEN}'))
    
            # Continue with new client call details
            request_iterator = iter((request,))
            updated_details = _ClientCallDetails(
                client_call_details.method, client_call_details.timeout, 
                metadata, client_call_details.credentials
            )
    
            return continuation(updated_details, next(request_iterator))
    
    
    class _ClientCallDetails(
            collections.namedtuple(
                '_ClientCallDetails',
                ('method', 'timeout', 'metadata', 'credentials')),
            grpc.ClientCallDetails):
        """
        Implement grpc.ClientCallDetails to pass modified request details in interceptor.
        """
        pass
    
    
    class AosClient(object):
        """
         Class to contain the AOS gRPC client service stubs for easier usage.
        """
        def __init__(self, ):
            ssl_credentials = grpc.ssl_channel_credentials()
            self.secure_channel = grpc.secure_channel("{}:{}".format(AOS_HOST, AOS_PORT), ssl_credentials)
            self.intercept_channel = grpc.intercept_channel(self.secure_channel, _MyAuthInterceptor())
    
            self.user_client = UserServiceStub(self.intercept_channel)
            self.project_client = ProjectServiceStub(self.intercept_channel)
            self.collection_client = CollectionServiceStub(self.intercept_channel)
            self.object_client = ObjectServiceStub(self.intercept_channel)
            self.object_group_client = ObjectGroupServiceStub(self.intercept_channel)
    
    
    # Entry point of the script
    if __name__ == '__main__':
        # Instantiate AosClient
        client = AosClient()
        
        # Do something with the client services ...
    ```

=== ":simple-python: Python (simple)"

    To use the Python API library in your Python project you have to install the PyPI package: `pip install Aruna-Python-API`.

    !!! note
    
        This example does not consider adding the authorization token metadata to every request. 

        In this case you have to manually add the authorization token header by using the `with_call(...)` extension of the client service methods.

        **All Python examples in this documentation assume that the client services have been initialized with an interceptor.**

    ```python
    import grpc

    from aruna.api.storage.services.v1.collection_service_pb2_grpc import CollectionServiceStub
    from aruna.api.storage.services.v1.object_service_pb2_grpc import ObjectServiceStub
    from aruna.api.storage.services.v1.objectgroup_service_pb2_grpc import ObjectGroupServiceStub
    from aruna.api.storage.services.v1.project_service_pb2_grpc import ProjectServiceStub
    from aruna.api.storage.services.v1.user_service_pb2_grpc import UserServiceStub
    
    # Valid Aruna API token
    #   In a production environment this should be stored in a more secure location ...
    API_TOKEN = 'MySecretArunaApiToken'
    
    # AOS instance gRPC gateway endpoint
    AOS_HOST = '<URL-To-AOS-Instance-gRPC-Gateway>' # Protocol (e.g. https://) has to be omitted
    AOS_PORT = '443'
    

    class AosClient(object):
        """
        Class to contain the AOS gRPC client service stubs for easier usage.
        """
        def __init__(self):
            # Read TLS credentials from local trusted certificates and instantiate a channel
            ssl_credentials = grpc.ssl_channel_credentials()
            self.channel    = grpc.secure_channel("{}:{}".format(AOS_HOST, AOS_PORT), ssl_credentials)
    
            self.user_client = UserServiceStub(self.channel)
            self.project_client = ProjectServiceStub(self.channel)
            self.collection_client = CollectionServiceStub(self.channel)
            self.object_client = ObjectServiceStub(self.channel)
            self.object_group_client = ObjectGroupServiceStub(self.channel)
    

    # Entry point of the Python script
    if __name__ == '__main__':
        # Instantiate AosClient
        client = AosClient()
        
        # Do something with the client services ...
    ```


## User registration

Users can register themselves with an individual display name in an AOS instance with their valid OIDC token received from the AAI login.

=== ":simple-curl: cURL"

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

=== ":simple-rust: Rust"

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

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to register OIDC user
    request = RegisterUserRequest(
        display_name="John Doe"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.RegisterUser(
        request=request,
        metadata=(('authorization', f'Bearer {OIDC_TOKEN}'),)
    )
    
    # Do something with the response
    print(f'{response}')
    ```


## User activation

!!! Note

    Users can only be activated by AOS instance administrators.

After registration users additionally have to be activated in a second step.

=== ":simple-curl: cURL"

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


=== ":simple-rust: Rust"

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

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request for user activation
    request = ActivateUserRequest(
        user_id="<user-id>"  # Has to be a valid UUID v4 of a registered user
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.ActivateUser(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Who Am I / What Am I

To check which user a token is associated with or get information about the current users permissions, you can use the UserService API.

!!! Note

    Only AOS instance administrators can request user information of other users.

=== ":simple-curl: cURL"

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

=== ":simple-rust: Rust"

    ```rust
    // Create tonic/ArunaAPI request to fetch user info of current user
    let get_request = GetUserRequest {
        user_id: "".to_string()
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_user(get_request)
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

=== ":simple-python: Python"

    ```python
    # Create tonic/ArunaAPI request to fetch user info of current user
    request = GetUserRequest()
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetUser(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
