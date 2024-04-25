
# How to use the Hooks API / HooksServiceClient

## Introduction

Hooks are the way to automate internal processes in Aruna and/or to integrate external services to extend functionality. 
Once created, they're available globally in Aruna, and Projects must be associated with them to be included in their trigger cycle. 
The action that triggers the specific hook is defined by its trigger type.

The individual trigger types currently include:

* **HookAdded:** Triggers, when a Hook key-value gets added to a resource. Can be limited to specific values.
* **ResourceCreated:** Triggers for all hierarchical resources on creation.
* **LabelAdded:** Triggers, when a Label key-value gets added to a resource. Can be limited to specific values.
* **ObjectFinished:** Triggers only for Objects on finish.
* **StaticLabelAdded:** Triggers, when an immutable Label key-value gets added to a resource. Can be limited to specific values.
* **HookStatusChanged:** Triggers, if a hook status change occurs on a resource.

A distinction is made between external hooks and internal hooks, which have slightly different use cases. 
External hooks address an external service that can use the transferred information to perform its formatting/validation/processing/... and return a result and/or upload the result data itself. 
The transferred information contains the metadata of a resource, which can also contain a link to the uploaded data of an Object in order to make it available to the service.
Internal hooks are responsible for the automatic binding of specific labels/hooks on resources and/or the creation of relationships. 
In other words, internal hooks can be used to automate certain Aruna internal processes.

The information sent to the external service can be customized by overwriting with the `custom_template` parameter.
The format of the additional information is absolutely free as long as it contains the mandatory placeholder from the following list:

| Placeholder         | Necessity | Description |
|---------------------|-----------|-------------|
| *{{secret}}*        | mandatory | Token secret that contains WRITE permissions on the resource that triggered the Hook. |
| *{{object_id}}*     | mandatory | Id of the resource that triggered the Hook. |
| *{{hook_id}}*       | mandatory | Id of the Hook that triggered the external service. |
| *{{pubkey_serial}}* | mandatory | Id of the public key that shall be used to verify the token signature. |
| *{{name}}*          | optional  | Name of the resource that triggered the Hook. |
| *{{description}}*   | optional  | Description of the resource that triggered the Hook. |
| *{{size}}*          | optional  | Size in bytes of the resource that triggered the Hook. |
| *{{key_values}}*    | optional  | Key-value pairs of the resource that triggered the Hook. |
| *{{status}}*        | optional  | Status of the resource that triggered the Hook. |
| *{{class}}*         | optional  | Data class of the resource that triggered the Hook. |
| *{{endpoints}}*     | optional  | Endpoints associated with the resource that triggered the Hook. |
| *{{download_url}}*  | optional  | Presigned URL that can used to download the data uploaded to the Object. |
| *{{access_key}}*    | optional  | Access Key Id part of the S3 credentials that can be used to upload data. |
| *{{secret_key}}*    | optional  | Secret Key part of the S3 credentials that can be used to upload data. |

The default template that gets send if no custom template is defined as JSON:

<div class="annotate" markdown>

```json linenums="1"
{
  "hook_id": "{{hook_id}}",
  "object": {
    "resource_id": "{{object_id}}",
    "persistent_resource_id": true/false, // (1)
    "checksum": "<resource-checksum>",
    "resource_variant": 1-4 // (2)
  },
  "secret": "{{secret}}",
  "download": "{{download_url}}",
  "pubkey_serial": "{{pubkey_serial}}",
  "access_key": "{{access_key}}",
  "secret_key": "{{secret_key}}"
}
```

</div>

1.  Depending if the resource is already archived/snapshot.
2.  **Available resource variants:**  
    Project = 1  
    Collection = 2  
    Dataset = 3  
    Object = 4


## Create Hook

API examples of how to create a global Hook which can be referenced by any Project.

??? Abstract "Required permissions"

    To create a new Hook you only have to be a registered Aruna user.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new external Hook which triggers 
    #    - on object finish
    #    - if the object is tagged with the hook key 'fasta-validation'
    curl -d '
      {
        "name": "Fasta-Validation",
        "trigger": {
          "triggerType": "TRIGGER_TYPE_OBJECT_FINISHED",
          "filters": [
            {
              "name": "hook-filter",
              "keyValue": {
                "key": "fasta-validation",
                "value": "",
                "variant": "KEY_VALUE_VARIANT_HOOK"
              }
            }
          ]
        },
        "hook": {
          "externalHook": {
            "url": "https://validation-demonstrator.org/fasta",
            "credentials": {
              "token": "SecretHookAuthToken"
            },
            "customTemplate": "",
            "method": "METHOD_POST"
          },
          "internalHook": {}
        },
        "timeout": "",
        "projectIds": [],
        "description": "Fasta file format validation demonstrator."
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/hook
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a new external Hook which triggers 
    //    - on object finish
    //    - if the object is tagged with the hook key 'fasta-validation'
    let request = CreateHookRequest {
        name: "Fasta-Validation".to_string(),
        trigger: Some(Trigger {
            trigger_type: TriggerType::ObjectFinished as i32,
            filters: vec![Filter {
                filter_variant: Some(FilterVariant::KeyValue(KeyValue {
                    key: "fasta-validation".to_string(),
                    value: "".to_string(),
                    variant: KeyValueVariant::Hook as i32,
                })),
            }],
        }),
        hook: Some(Hook {
            hook_type: Some(HookType::ExternalHook(ExternalHook {
                url: "https://validation-demonstrator.org/fasta".to_string(),
                credentials: Some(Credentials {
                    token: "SecretHookAuthToken".to_string(),
                }),
                custom_template: None,
                method: Method::Post as i32,
            })),
        }),
        timeout: 604800, // One week in seconds
        project_ids: vec![],
        description: "Fasta file format validation demonstrator.".to_string(),
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = hook_client.create_hook(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new external Hook which triggers 
    #    - on object finish
    #    - if the object is tagged with the hook key 'fasta-validation'    
    request = CreateHookRequest(
        name="Fasta-Validation",
        trigger=Trigger(
            trigger_type=TriggerType.TRIGGER_TYPE_OBJECT_FINISHED,
            filters=[],
        ),
        hook=Hook(
            external_hook=ExternalHook(
                url="https://validation-demonstrator.org/fasta",
                credentials=Credentials(
                    token="SecretHookAuthToken"
                ),
                custom_template="",
                method=Method.METHOD_POST
            )
        ),
        timeout=604800,
        project_ids=[],
        description="Fasta file format validation demonstrator."
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.hook_client.CreateHook(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Add Projects to Hook

Projects have to be added to the globally available hooks to be included in their trigger cycle.

??? Abstract "Required permissions"

    To add a Hook to a Project you require ADMIN permissions on the Project.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to add Projects to a Hook
    curl '
      {
        "projectIds": [
            "<project-id-01>",
            "<project-id-02>",
            "<...>",
        ]
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-Aruna-instance-API-endpoint>/v2/hook/{hook-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to add Projects to a Hook
    let request = AddProjectsToHookRequest {
        hook_id: "<hook-id>".to_string(),
        project_ids: vec![
            "<project-id-01>".to_string(),
            "<project-id-02>".to_string(),
            "<...>".to_string(),
        ],
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = hook_client.add_projects_to_hook(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to add Projects to a Hook
    request = AddProjectsToHookRequest(
        hook_id="<hook-id>",
        project_ids=[
            "<project-id-01>",
            "<project-id-02>",
            "<...>",
        ]
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.hook_client.AddProjectsToHook(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get Hooks of Project

API examples of how to list all Hooks a specific Project is associated with.

??? Abstract "Required permissions"

    This request requires at least ADMIN permissions on the specific Project.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new external Hook which triggers on object creation
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/hooks/project/{project-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to list all active Hooks of a Project
    let request = ListProjectHooksRequest { 
        project_id: "<project-id>".to_string()
     };

    // Send the request to the Aruna instance gRPC endpoint
    let response = hook_client.list_hooks_of_project(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to list all active Hooks of a Project
    request = ListProjectHooksRequest(
        project_id="<project-id>"
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.hook_client.ListProjectHooks(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Get Hooks of creator

API examples of how to list all Hooks created by a specific user.

??? Abstract "Required permissions"

    To list your created Hooks you only have to be a registered Aruna user.

    Global Aruna administrators can list the created Hooks of other users.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new external Hook which triggers on object creation
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-Aruna-instance-API-endpoint>/v2/hooks/owner/{user-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to list all Hooks created by a specific user
    let request = ListOwnedHooksRequest {
        user_id: "<user-id>".to_string(),
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = hook_client.list_hooks_of_project(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to list all Hooks created by a specific user
    request = ListOwnedHooksRequest(
        user_id="<user-id>"
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.hook_client.ListOwnedHooks(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Hook callback

This is the API endpoint which has to be used from an external Hook service to return its execution result. 
The result can be either an error message or on success optionally trigger the addition and/or removal of 
key-value pairs to a specific Object.

??? Abstract "Required permissions"

    This request does not require specific permissions as they're handled with the provided token secret.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fuzzy search for a keyword
    curl '
      {
        "secret": "SecretHookAuthToken"
        "hookId": "<hook-id>",
        "objectId": "<object-id>",
        "pubkeySerial": 1337,
        "error": "<error-message>",
        "addKeyValues": [],
        "removeKeyValues": []
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X PATCH https://<URL-to-Aruna-instance-API-endpoint>/v2/hook/callback
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to return a successful external Hook service result
    let request = HookCallbackRequest {
        secret: "SecretHookAuthToken".to_string(),
        hook_id: "<hook-id>".to_string(),
        object_id: "<object-id>>".to_string(),
        pubkey_serial: 1337,
        status: Some(Status::Finished(Finished {
            add_key_values: vec![KeyValue {
                key: "fasta-validated".to_string(),
                value: "true".to_string(),
                variant: KeyValueVariant::Label as i32,
            }],
            remove_key_values: vec![],
        })),
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = hook_client.hook_callback(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to return a successful external Hook service result
    request = HookCallbackRequest(
        secret="SecretHookAuthToken",
        hook_id="<hook-id>",
        object_id="<object-id>",
        pubkey_serial=1337,
        finished=Finished(
            add_key_values=[
                KeyValue(
                    key="fasta-validated",
                    value="true",
                    variant=KeyValueVariant.KEY_VALUE_VARIANT_LABEL
                )
            ],
            remove_key_values=[]
        ),
        error=None
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.hook_client.HookCallback(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```


## Delete Hook

API examples of how to delete a Hook.

??? Abstract "Required permissions"

    This request requires ADMIN permissions on all Projects associted with the Hook.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to delete a Hook
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X DELETE https://<URL-to-Aruna-instance-API-endpoint>/v2/hook/{hook-id}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to delete a Hook
    let request = DeleteHookRequest { 
        hook_id: "<hook-id>".to_string()
    };

    // Send the request to the Aruna instance gRPC endpoint
    let response = hook_client.delete_hook(request)
                              .await
                              .unwrap()
                              .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to delete a Hook
    request = DeleteHookRequest(
        hook_id="<hook-id>"
    )
    
    # Send the request to the Aruna instance gRPC endpoint
    response = client.hook_client.HookCallback(request=request)
    
    # Do something with the response
    print(f'{response}')
    ```
