
# Introduction

## Event notifications

The AOS includes an event notification service which can be used to _subscribe_ to specific resources including their subresources. 
This means that the created event notification consumer can be used to fetch notifications regarding all actions which affected the resources in the scope of the consumer. 

The event notification system is based on the integration of the cloud native, open source messaging technology [:simple-natsdotio: NATS.io](https://nats.io){target=_blank} 

These notifications have a shelf life, so they are not persistent forever. 
The exact concept of how many notifications are retained over what period of time is not yet final. 


### Create event notification consumer

API examples of how to create an event notification consumer.

??? Abstract "Required permissions"

    This request requires at least READ permissions on all resources in the scope of the consumer.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create an event notification consumer
    let request = CreateStreamConsumerRequest {
        target: Some(Target::Resource(ResourceTarget {
            resource_id: "<project-id>".to_string(),
            resource_variant: ResourceVariant::Project as i32,
        })),
        include_subresources: true,
        stream_type: Some(StreamType::StreamAll(StreamAll{})),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = notification_client.create_stream_consumer(request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create an event notification consumer
    request = CreateStreamConsumerRequest(
        resource=ResourceTarget(
            resource_id="<resource-id>",
            resource_variant=ResourceVariant.RESOURCE_VARIANT_PROJECT
        ),
        user="<user-id>",
        anouncements=False,
        all=False,
        include_subresources=True,
        stream_all=StreamAll(),
        stream_from_date=None,
        stream_from_sequence=None
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.notification_client.CreateStreamConsumer(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Fetch messages via batch

API examples of how to fetch an event notification message batch.

??? Abstract "Required permissions"

    This request requires at least READ permissions on all resources in the scope of the consumer.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch an event message batch
    let request = GetEventMessageBatchRequest { 
        stream_consumer: "<stream-consumer-id>".to_string(), 
        batch_size: 25 
    };

    // Send the request to the AOS instance gRPC gateway
    let response = notification_client.get_event_message_batch(request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch an event message batch
    request = GetEventMessageBatchRequest(
        stream_consumer="<stream-consumer-id>", 
        batch_size=25 
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.notification_client.GetEventMessageBatch(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Fetch messages via stream

API examples of how to fetch event notification messages via stream.

??? Abstract "Required permissions"

    This request requires at least READ permissions on all resources in the scope of the consumer.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create an event notification stream
    let request = GetEventMessageStreamRequest { 
        stream_consumer: "<stream-consumer-id>".to_string()
    };

    // Send the request to the AOS instance gRPC gateway
    let response = notification_client.get_event_message_stream(request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create an event notification stream
    request = GetEventMessageStreamRequest ( 
        stream_consumer: "<stream-consumer-id>"
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.notification_client.GetEventMessageStream(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Acknowledge messages

Event notification messages have to be acknowledged in order that they won't be delivered again to the consumer after a set period of time.

??? Abstract "Required permissions"

    As the complete `Reply` (including a secure HMAC) is also delivered with the message individually for the consumer there are no special permissions required.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to acknowledge a batch of event notifications
    let request = AcknowledgeMessageBatchRequest {
        replies: vec![Reply {
            reply: "<reply-subject>".to_string(),
            salt: "<hmac-salt>",
            hmac: "<message-hmac>".to_string(),
        }],
    };

    // Send the request to the AOS instance gRPC gateway
    let response = notification_client.acknowledge_message_batch(request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to acknowledge a batch of event notifications
    request = AcknowledgeMessageBatchRequest(
        replies: [Reply {
            reply="<reply-subject>",
            salt="<hmac-salt>",
            hmac="<message-hmac>"
        }]
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.notification_client.AcknowledgeMessageBatch(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


### Delete event notification consumer

API examples of how to delete an event notification consumer.

??? Abstract "Required permissions"

    This request requires at least WRITE permissions.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete an event notification consumer
    let request = DeleteStreamConsumerRequest {
        stream_consumer: "<stream-consumer-id>".to_string(),
    };

    // Send the request to the AOS instance gRPC gateway
    let response = notification_client.delete_stream_consumer(request)
                                      .await
                                      .unwrap()
                                      .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete an event notification consumer
    request = DeleteStreamConsumerRequest(
        stream_consumer="<stream-consumer-id>",
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.notification_client.DeleteStreamConsumer(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

## Personal notifications

Some notifications are not based on actions in the AOS that affected specific resources but are addressed directly to a specific user.
These notifications have to be persistent and therefore are retained separately from the event notification system until explicitly acknowledged.

Examples for personal notifications would be permission changes on resources or access requests on resources from other users.


### Get personal notifications

API examples of how to fetch persistent personal notifications.

??? Abstract "Required permissions"

    This request only requires a personal token which authenticates and provides the user id.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch personal notifications
    let request = GetPersonalNotificationsRequest {};

    // Send the request to the AOS instance gRPC gateway
    let response = user_client.get_personal_notifications(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch personal notifications
    request = GetPersonalNotificationsRequest()
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.GetPersonalNotifications(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```

### Acknowledge personal notifications

API examples of how to acknowledge persistent personal notifications.

??? Abstract "Required permissions"

    This request only requires a personal token which authenticates the user.

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to acknowledge personal notifications
    let request = AcknowledgePersonalNotificationsRequest {
        notification_ids: vec![
            "<notification-id-01>".to_string(),
            "<notification-id-02>".to_string(),
            "<...>".to_string()
        ],
    };

    // Send the request to the AOS instance gRPC gateway
    let response = user_client.acknowledge_personal_notifications(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to acknowledge personal notifications
    request = AcknowledgePersonalNotificationsRequest(
        notification_ids=[
            "<notification-id-01>".to_string(),
            "<notification-id-02>".to_string(),
            "<...>".to_string()
        ]
    )
    
    # Send the request to the AOS instance gRPC gateway
    response = client.user_client.AcknowledgePersonalNotifications(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```