
# How to use the DataProxy S3 compatible interface

## Introduction

The AOS DataProxy implements a subset of the AWS S3 API. By providing an S3 compatible interface, 
the AOS DataProxy is able to interact with a wide variety of modern data analysis tools which natively support S3 as storage solution.

Here we will give you examples for the currently supported and most common S3 functionality which can be used directly with an AOS DataProxy. 
For the best accessibility all the examples are written for the freely available tool 
[S3cmd](https://s3tools.org/s3cmd){:target="_blank"} which is a free command line tool and client for uploading, 
retrieving and managing data in Amazon S3 and other cloud storage service providers that use the S3 protocol.

!!! Info "S3cmd configuration file"
    The examples assume only a minimalist configuration file containing the Access Key ID, Secret Access Key and AOS Data Proxy host. The remaining parameters can be set with the default values.

    ```toml title=".s3cfg"
    [default]
    access_key = <api-token-id>
    secret_key = <api-token-s3-secret>
    host_base = <aos-dataproxy-endpoint>
    host_bucket = %(bucket)s.<aos-dataproxy-endpoint>
    ...
    ```

If you're unfamiliar with the S3 path format you should first take a look at the [:material-book-education: **Object Paths Introduction**](06_How-To-Object-Paths.md#introduction).

To communicate directly with an AOS Data Proxy via the S3 interface, the path used must also be extended to include the specific Data Proxy endpoint. 
This means that the path format changes to: `s3://bucket.location/key`

!!! Example "Examples for valid S3 interface paths"

    * **Project name:** `dummy-project`
    * **Collection name:** `sample-collection`
    * **Collection version:** `None`
    * **DataProxy URL:** `https://data.aos-endpoint.gi.de`
    * **Object filename:** `example.file`

    This would correspond to the path: `s3://latest.sample-collection.dummy-project.data.aos-endpoint.gi.de/example.file`

    Custom sub-paths are supported here as well:

    * **Custom object path:** `/my-subdir`

    This would correspond to the path: `s3://latest.sample-collection.dummy-project.data.aos-endpoint.gi.de/my-subdir/example.file`



## Put Object

Adds an Object in a Collection, uploads the specified data and finishes the Object to make it available.

If an object already exists under the provided path, this object will be updated and a new revision will be created.

[:material-file-cog: **Native S3 Put Request Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html){:target="_blank"}

!!! Info

    This request needs at least APPEND permissions on the Object's Collection or the Project under which the Collection is registered.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Upload a file as single part upload
    s3cmd put /tmp/example.file s3://latest.sample-collection.dummy-project.data.aos-endpoint.gi.org/example.file
    ```


## Multipart Upload

S3cmd also supports Amazon S3 multipart uploads. Multipart uploads are automatically used 
when a file to upload is larger than 15MB if the `--multipart-chunk-size-mb` parameter is not set.

Natively a multipart upload has to be `created` for each part upload and `completed` after the upload of all parts has been finished.

- [:material-file-cog: **Native S3 Create Multipart Upload Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html){:target="_blank"}
- [:material-file-cog: **Native S3 Upload Part Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html){:target="_blank"}
- [:material-file-cog: **Native S3 Complete Multipart Upload Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html){:target="_blank"}

!!! Info

    This request needs at least APPEND permissions on the Object's Collection or the Project under which the Collection is registered.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Upload a file in 500MB chunks
    s3cmd put --multipart-chunk-size-mb=500 /tmp/example.file s3://latest.sample-collection.dummy-project.data.aos-endpoint.gi.org/example.file
    ```


## Head Object

The HEAD action retrieves metadata from an object without returning the object itself. 
This action is useful if you're only interested in an object's metadata. 

[:material-file-cog: **Native S3 Head Object Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_HeadObject.html){:target="_blank"}

!!! Info

    This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Get info of specific object
    s3cmd info s3://latest.sample-collection.dummy-project.data.aos-endpoint.gi.org/example.file
    ```


## Get Object

Retrieves the data associated with the Object.

[:material-file-cog: **Native S3 Get Object Specification**](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html){:target="_blank"}

!!! Info

    This request needs at least READ permissions on the Object's Collection or the Project under which the Collection is registered.

=== ":simple-amazons3: S3cmd"

    ```bash linenums="1"
    # Download the data associated with the object to a local file
    s3cmd get s3://latest.sample-collection.dummy-project.data.aos-endpoint.gi.org/example.file /tmp/example.file
    ```
