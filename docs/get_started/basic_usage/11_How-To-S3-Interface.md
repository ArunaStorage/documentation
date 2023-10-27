
# How to use the DataProxy S3 compatible interface

## Introduction

The AOS DataProxy implements a subset of the AWS S3 API specification. By providing an S3 compatible interface, 
the AOS DataProxy is able to interact with a wide variety of modern data analysis tools which natively support S3 as storage solution.

Here we will give you examples for the currently supported and most common S3 functionality which can be used directly with an AOS DataProxy. 
For the best accessibility all the examples are written for the freely available tool 
[S3cmd](https://s3tools.org/s3cmd){:target="_blank"} which is a command line client for uploading, 
retrieving and managing data in Amazon S3 and other cloud storage service providers that use/support the S3 protocol.

!!! Info "S3cmd configuration"
    The examples assume only a minimalist configuration file containing the Access Key ID, Secret Access Key and AOS DataProxy host. The remaining parameters can be set with the default values.

    ```toml title=".s3cfg"
    [default]
    access_key = <access-key-id>
    secret_key = <access-secret>
    host_base = <aos-dataproxy-endpoint>
    host_bucket = %(bucket)s.<aos-dataproxy-endpoint>
      ...
    ```

    An alternative to the configuration file would be to use the CLI options of S3cmd: 
    
    * `--host=<aos-dataproxy-endpoint>`
    * `--host-bucket=%(bucket)s.<aos-dataproxy-endpoint>`, 
    * `--access_key=<access-key-id>`
    * `--secret-key=<access-secret>`

    **But be aware that with the usage of the `--secret-key` parameter your private secret key gets logged in clear text at the least in your terminal history. This should only be used in a testing environment!**

!!! Tip "DataProxy access"

    * Before you can directly communicate with or upload data to a DataProxy you have to register your user first by requesting credentials for the specific endpoint.


### Object paths in AOS

In order for the AOS to provide an S3 compatible interface, it is necessary that Objects can be accessed via a unique path instead of their id.

Currently, this path complies with the standardized specifications of AWS S3 and are represented in the format  
`<project-name>/<collection-name>/<dataset-name>/<object-filename>` which resembles the S3 path-style `s3://bucket/key` where:

* bucket: `<project-name>`
* key: `<collection-name>/<dataset-name>/<object-filename>`

This also applies when the Object has a relation to multiple parents which means that in this case the Object is available through multiple paths.

??? Example "Example for an Object path inside an existing Collection"

    * **Project name:** `dummy-project`
    * **Collection name:** `sample-collection`
    * **Object filename:** `example.file`

    This would correspond to the path: `s3://dummy-project/sample-collection/example.file`

??? Example "Example for an Object path inside an existing Dataset"

    * **Project name:** `dummy-project`
    * **Collection name:** `sample-collection`
    * **Dataset name:** `my-dataset`
    * **Object filename:** `example.file`

    This would correspond to the path: `s3://dummy-project/sample-collection/my-dataset/example.file`

<!--
??? Example "Example for an Object with a custom path"

    **Custom paths are still experimental, so while custom paths may work as documented, any such usage is at your own risk.**

    Custom paths are only 
-->

!!! Warning

    **The fully qualified paths of objects are unique, which implies some conditions that must be met:**

    * Project names are unique globally
    * Project names are restricted to the following characters: [a-z0-9\-] (i.e. alphanumeric lowercase and hyphens)
    * Collection and Dataset names are restricted to the safe characters specified in the [S3 object key naming guidelines](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html#object-key-guidelines){target=_blank}
    * Names are unique within each hierarchy (e.g. you cannot create Objects with the same name inside the same Collection)


## Create bucket

This operation is analog to the [Create Project](04_How-To-Project.md){:target="_blank"} API request with a specific endpoint.
This means that the Project is also registered in the central ArunaServer catalog with the DataProxy as the main data location.

[:material-file-cog: **Native S3 Create Bucket Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateBucket.html){:target="_blank"}

??? Abstract "Required permissions"

    To create a new Project you only have to be a registered AOS user.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Create a Project with a create bucket request
    s3cmd mb s3://mybucket
    ```


## Put Object

This operation is analog to the combination of [Create Object](07_How-To-Objects.md#initialize-object){:target="_blank"}, [Upload data](07_How-To-Objects.md#upload-data-to-a-staging-object){:target="_blank"} and [Finish Object](07_How-To-Objects.md#finish-object){:target="_blank"}. 
That also means that on usccess the Object is directly available. 
Optionally the specified parent resources are also created if they do not exist at the time of the execution.

If an object already exists under the provided path, this object will be updated and a new revision will be created which takes the uploaded data.

[:material-file-cog: **Native S3 Put Request Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least APPEND permission on the parent resource in which the Collection/Dataset/Object is to be created.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Upload a file as single part upload in a Project
    s3cmd put /tmp/example.file s3://mybucket/
    ```

    ```bash linenums="1"
    # Upload a file as single part upload in a Dataset that does not exist
    s3cmd put /tmp/example.file s3://mybucket/my-dataset/
    ```

    ```bash linenums="1"
    # Upload a file as single part upload in a Collection if it already exists
    s3cmd put /tmp/example.file s3://mybucket/my-collection/
    ```

    ```bash linenums="1"
    # Upload a file as single part upload in a Dataset inside a Collection
    s3cmd put /tmp/example.file s3://mybucket/my-collection/my-dataset/
    ```


## Multipart Upload

S3cmd also supports Amazon S3 multipart uploads. Multipart uploads are automatically used 
when a file to upload is larger than 15MB if the `--multipart-chunk-size-mb` parameter is not set.

Natively a multipart upload has to be `created` for each part upload and `completed` after the upload of all parts has been finished.

- [:material-file-cog: **Native S3 Create Multipart Upload Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html){:target="_blank"}
- [:material-file-cog: **Native S3 Upload Part Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html){:target="_blank"}
- [:material-file-cog: **Native S3 Complete Multipart Upload Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least APPEND permission on the parent resource in which the Collection/Dataset/Object is to be created.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Upload a file in 500MB chunks
    s3cmd put --multipart-chunk-size-mb=500 /tmp/example.file s3://mybucket/
    ```


## Head Object

The HEAD action retrieves metadata from an object without returning the object itself. 
This action is useful if you're only interested in an object's metadata. 

[:material-file-cog: **Native S3 Head Object Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadObject.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Object or one of its parent resources.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Get info of a specific Object
    s3cmd info s3://mybucket/example.file
    ```


## Get Object

Retrieves the data associated with the Object.

[:material-file-cog: **Native S3 Get Object Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Object or one of its parent resources.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Download the data associated with the Object to a local file
    s3cmd get s3://mybucket/example.file /tmp/example.file
    ```


## List bucket objects

Returns all (up to 1.000 for each request) Objects of a specific bucket i.e. AOS Project.

Unfortunately, S3cmd does only support the revised *ListObjects*, which is why for now we just can refer to the [**Native S3 ListObjectsV2 specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListObjectsV2.html){:target="_blank"}

??? Abstract "Required permissions"

    This request requires at least READ permissions on the Project
