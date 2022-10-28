# Documentation

Welcome to the general documentation for the Aruna Object Storage (AOS).

This documentation includes theoretical concepts, the internal data structure, the entity-relationship model, a basic usage guide, some generic user story playbooks, and much more in the future.

Deeper technical documentation can be found in the implementation repositories for the [API](#aos-api), [Server](#aos-server) or [CLI](#aos-cli).
Details on the individual structures can be found in the API documentation and/or the [Data Structure](internal_data_structure/internal_data_structure.md) part of this documentation.

<!--
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./internal_data_structure/internal_data_structure.dark.png">
  <img alt="UML diagram of the Aruna Object Storage internal data structure." src="./internal_data_structure/internal_data_structure.png">
</picture>
-->

<figure class="pull-right" markdown>
  ![UML diagram of the Aruna Object Storage data structure](internal_data_structure/internal_data_structure.png){ align=right }
  <figcaption markdown>UML diagram of the Aruna Object Storage data structure. A more detailed description of the individual parts can be found [here](internal_data_structure/internal_data_structure.md).</figcaption>
</figure>


## AOS Components

### **AOS API** 

[**Main Aruna API repository**](https://github.com/ArunaStorage/ArunaAPI){:target="_blank"}

The API is designed generally with Google Protocol Buffers. With the release of a new API version, the client libraries are automatically compiled and updated to the latest version.
The API is fundamentally backwards compatible, which means that users' applications will continue to work as usual before they also decide to move to the new version.

* Rust API stubs: [GitHub](https://github.com/ArunaStorage/rust-api){:target="_blank"} or [crates.io](https://crates.io/crates/aruna-rust-api){:target="_blank"}
* Go API stubs: [GitHub](https://github.com/ArunaStorage/go-api){:target="_blank"}
* Python API stubs: [GitHub](https://github.com/ArunaStorage/python-api){:target="_blank"} or [PyPI](https://pypi.org/project/Aruna-Python-API){:target="_blank"}
* Java API stubs: [GitHub](https://github.com/ArunaStorage/java-api){:target="_blank"}

### **AOS Server**

[**Main Aruna Server repository**](https://github.com/ArunaStorage/ArunaServer){:target="_blank"}

The implementation of the API that handles the incoming requests.

> Aruna Object Storage is data lake application with API that manages scientific data objects according to FAIR principles.
It interacts with multiple data storage backends (e.g. S3, File ...) via the DataProxy application and stores a rich and queryable set of metadata about stored objects in a
postgres compatible database (e.g. CockroachDB).
Aruna is conceptually geo-redundant with multiple dataset and database locations. Users can choose where and how to store their data.

### **AOS Data Proxy**

[**Main Aruna Data Proxy repository**](https://github.com/ArunaStorage/DataProxy){:target="_blank"}

This is the internal server implementation handling the communication between the data storage backend used for the specific AOS instance.

> The DataProxy service is designed to handle data transfers AOS. It starts two server: One server implements the internal proxy api from the official AOS gRPC API. 
> The other server handles the transfer of the actual data. All data access is handled through presigned URLs.

### **AOS CLI**

[**Main Aruna CLI repository**](https://github.com/ArunaStorage/ArunaCLI){:target="_blank"}

> This is a simple CLI application for the ScienceObjectsDB API. 
> Its currently work in progress and will be developed along with the API. Neither concept nor implementation are final.

### **Notification system**

The storage system has a notification system that can be used to receive change notification on specific resources.

The notification system is currently still in the implementation phase but will be available soon.

<!-- An example can be found here: [Notification Stream Example](#) -->

## Implementation Design Trivia

- An RDBMS will be used as database backend for the AOS Server
- The AOS Server, Data Proxy nd CLI will be implemented in [Rust](https://www.rust-lang.org/){:target="_blank"}
- The base API interface will be defined using [Protocol Buffers](https://developers.google.com/protocol-buffers){:target="_blank"}
- All endpoints work with JSON over HTTP just as they do with requests made via gRPC from individual clients
- [Clients stubs](#aos-api) will be generated for major programming languages on every API release
- A [basic CLI client](https://github.com/ArunaStorage/ArunaCLI){:target="_blank"} will be offered to simplify the usage entry barrier
- A [web UI](https://web.aruna.nfdi-dev.gi.denbi.de/ui/){:target="_blank"} is available for demonstration purposes

