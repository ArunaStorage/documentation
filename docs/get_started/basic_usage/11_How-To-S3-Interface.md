
# How to use the DataProxy S3 compatible interface

## Introduction

The Aruna DataProxy implements a subset of the AWS S3 API specification. By providing an S3 compatible interface, 
the DataProxy is able to interact with a wide variety of modern data analysis tools which natively support S3 as storage solution.

Here we will give you examples for the currently supported and most common S3 functionality which can be used directly with an Aruna DataProxy. 
For the best accessibility all the examples are written for the freely available tool 
[s5cmd](https://github.com/peak/s5cmd){:target="_blank"} which is an extremely fast command line client for uploading, 
retrieving and managing data in Amazon S3 and other cloud storage service providers that use/support the S3 protocol.

!!! Tip "DataProxy access"

    Before you can directly communicate with or upload data to a DataProxy you have to "register" your user first by requesting credentials for the specific endpoint.

    This is for data protection reasons, so that each DataProxy only receives and stores the data that is relevant to it.

!!! Info "s5cmd Configuration"
    s5cmd makes use of the [:material-file-cog: standard S3 configuration and credentials file](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html){:target="_blank"}.
    The examples in this section assume only a minimalist configuration containing the Access Key ID, Secret Access Key and DataProxy host. 
    The remaining parameters are optional and can be set with defaults.

    The configuration files are automatically picked up by *s5cmd* when they exist in the following default locations: 

    * `<user-home>/.aws/config` and `<user-home>/.aws/credentials`

    ```toml title=".aws/config"
    [default]
    endpoint_url = <data-proxy-host-url>
    region = <bucket-region>
    output = json
      ...
    ```

    ```toml title=".aws/credentials"
    [default]
    aws_access_key_id = <access-key-id>
    aws_secret_access_key = <access-secret>
      ...
    ```

    An alternative to the configuration files would be to use environment variables: 

    * **Access Key Id:** `export AWS_ACCESS_KEY_ID='<access-key-id>'`
    * **Secret Access Key:** `export AWS_SECRET_ACCESS_KEY='<access-secret>'`
    * **Endpoint URL:** `export S3_ENDPOINT_URL='<data-proxy-host-url>'`
    * **Bucket Region:** `export AWS_REGION='<bucket-region>'`

    ... or partially the global CLI options of *s5cmd*:

    * **Endpoint URL:** `--endpoint-url <aruna-dataproxy-endpoint>`
    * **Output format:** `--json`
    * **Disable SSL verification:** `--no-verify-ssl`


### Object paths in Aruna

In order for Aruna to provide an S3 compatible interface, it is necessary that Objects can be accessed via a unique path instead of their id.

Currently, this path complies with the standardized specifications of AWS S3 and are represented in the format  
`<project-name>/<collection-name>/<dataset-name>/<object-filename>` which resembles the S3 path-style `s3://bucket/key` where:

* bucket: `<project-name>`
* key: `<collection-name>/<dataset-name>/<object-filename>`

This also applies when the Object has a relation to multiple parents which means that in this case the Object is available through multiple paths.

??? Example "Example for an Object path inside a Project"

    * **Project name:** `dummy-project`
    * **Object filename:** `example.file`

    This would correspond to the path: `s3://dummy-project/example.file`

??? Example "Example for an Object path inside an existing Collection"

    * **Project name:** `dummy-project`
    * **Collection name:** `sample-collection`
    * **Object filename:** `example.file`

    This would correspond to the path: `s3://dummy-project/sample-collection/example.file`

??? Example "Example for an Object path inside an existing Collection and Dataset"

    * **Project name:** `dummy-project`
    * **Collection name:** `sample-collection`
    * **Dataset name:** `my-dataset`
    * **Object filename:** `example.file`

    This would correspond to the path: `s3://dummy-project/sample-collection/my-dataset/example.file`

!!! Warning "Path limitations"

    **The fully qualified paths of objects are unique, which implies some conditions that must be met:**

    * Project names are unique globally
    * Project names are restricted to the following characters: [a-z0-9\-] (i.e. alphanumeric lowercase and hyphens)
    * Collection and Dataset names are restricted to the safe characters specified in the [S3 object key naming guidelines](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html#object-key-guidelines){target=_blank}
    * Names are unique within each hierarchy (e.g. you cannot create Objects with the same name inside the same Collection)


## Create bucket

This operation is analog to the [Create Project](04_How-To-Project.md){:target="_blank"} API request with a specific endpoint.

This means that the Project is also registered in the central ArunaServer catalog with the specific DataProxy as its main data location.

[:material-file-cog: **Native S3 Create Bucket Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateBucket.html){:target="_blank"}

??? Abstract "Required permissions"

    To create a new Project you only have to be a registered Aruna user.

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # Create a Project with a create bucket request
    s5cmd mb s3://mybucket
    ```


## Put Object

[:material-file-cog: **Native S3 Put Request Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html){:target="_blank"}

This operation is analog to the combination of [Create Object](07_How-To-Objects.md#initialize-object){:target="_blank"}, [Upload data](07_How-To-Objects.md#upload-data-to-a-staging-object){:target="_blank"} and [Finish Object](07_How-To-Objects.md#finish-object){:target="_blank"}. 
That also means that on success the Object is directly available. 

**If an object already exists under the provided path, this object will be updated and a new revision will be created which takes the uploaded data.**

!!! Tip "Optional parent creation"

    Optionally, the non-existent parent resources of the path will also be created if they do not exist at the time of execution, except for Projects.
    As the middle part of a path in an Aruna hierarchy can be ambiguous, the Dataset is always favoured during creation.
    This means you can't create a Collection as a direct parent of an Object with an S3 request.

    === "Create a Collection, Dataset and Object"

        * **Project name:** `dummy-project`
        * **Object filename:** `example.file`

        `s3://dummy-project/new-collection/new-dataset/example.file` would also create the Collection with the name `new-collection` and the Dataset with the name `new-dataset`. The created Dataset will be the direct parent of the Object.

    === "Create a Dataset and Object"

        * **Project name:** `dummy-project`
        * **Object filename:** `example.file`

        `s3://dummy-project/new-dataset/example.file` would also create the Dataset with the name `new-collection` as the parent of the Object. The created Dataset will be the direct parent of the Object.

??? Abstract "Required permissions"

    This request requires at least APPEND permission on the parent resource in which the Collection/Dataset/Object is to be created.

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # Upload a file as single part upload in a Project
    s5cmd cp /tmp/example.file s3://dummy-project/example.file
    ```

    ```bash linenums="1"
    # Upload a file as single part upload in a Dataset that does not exist
    s5cmd cp /tmp/example.file s3://dummy-project/my-dataset/example.file
    ```

    ```bash linenums="1"
    # Upload a file as single part upload in a Collection if it already exists
    s5cmd cp /tmp/example.file s3://dummy-project/my-collection/example.file
    ```

    ```bash linenums="1"
    # Upload a file as single part upload in a Dataset inside a Collection
    s5cmd cp /tmp/example.file s3://dummy-project/my-collection/my-dataset/example.file
    ```


## Multipart Upload

s5cmd also supports Amazon S3 multipart uploads. Multipart uploads are automatically used 
when a file to upload is larger than 5MB if the `multipart_chunksize = <chunk-size>` parameter is not set in the configuration file.

Natively a multipart upload has to be `created` for each part upload and `completed` after the upload of all parts has been finished.

- [:material-file-cog: **Native S3 Create Multipart Upload Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html){:target="_blank"}
- [:material-file-cog: **Native S3 Upload Part Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html){:target="_blank"}
- [:material-file-cog: **Native S3 Complete Multipart Upload Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least APPEND permission on the parent resource in which the Collection/Dataset/Object is to be created.

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # Upload a file in 5MB chunks
    s5cmd cp /tmp/large-example.file s3://dummy-project/large-example.file
    ```


## Head Object

The HEAD action retrieves metadata from an object without returning the object itself. 
This action is useful if you're only interested in an object's metadata. 

[:material-file-cog: **Native S3 Head Object Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadObject.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Object or one of its parent resources.

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # Get info of a specific Object
    s5cmd ls s3://dummy-project/example.file
    ```


## Get Object

Retrieves the data associated with the specific Object.

[:material-file-cog: **Native S3 Get Object Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Object or one of its parent resources.

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # Download the data associated with the Object to a local file
    s5cmd cp s3://dummy-project/example.file /tmp/example.file
    ```


## List buckets/Projects

Lists all the buckets (i.e. Projects) the user has specific permissions for.

[:material-file-cog:**Native S3 ListBuckets specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListBuckets.html){:target="_blank"}

??? Abstract "Required permissions"

    This request does not require any kind of specific permission but only buckets/Projects the user has permissions on will be returned.

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # List all buckets/Projects a user has permissions for 
    s5cmd ls
    ```


## List bucket/Project objects

Returns all (up to 1.000 for each request) Objects of a specific bucket i.e. Project.

[:material-file-cog:**Native S3 ListObjectsV2 specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Project

=== ":simple-amazons3: s5cmd"

    ```bash linenums="1"
    # Recursively list all objects in a Project/bucket 
    s5cmd ls s3://dummy-project/*
    ```

    ```bash linenums="1"
    # Recursively list all objects in a public Project/bucket 
    s5cmd --no-sign-request ls s3://public-project/*
    ```
