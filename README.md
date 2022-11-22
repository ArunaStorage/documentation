# Documentation

This repository contains the general documentation for the Aruna Object Storage (AOS).

This includes a basic usage guide, the internal data structure, the entity-relationship model, some generic user story playbooks, theoretical concepts, and much more in the future.

Deeper technical documentation can be found in the implementation repositories.
Details on the individual structures can be found in the API documentation and/or the [Internal data structure](https://ArunaStorage.github.io/Documentation/internal_data_structure/internal_data_structure/) part of this repository.

[🌐 **The complete AOS documentation is available via GitHub Pages** 🌐](https://ArunaStorage.github.io/Documentation)

## Design overview

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./docs/internal_data_structure/internal_data_structure.dark.png">
  <img alt="Diagram of the Aruna Object Storage " src="./docs/internal_data_structure/internal_data_structure.png">
</picture>

A more detailed description of the individual resources can be found in the [complete documentation](https://ArunaStorage.github.io/Documentation/internal_data_structure/internal_data_structure/).

## AOS Components

### **AOS API**

[**Main Aruna API repository**](https://github.com/ArunaStorage/ArunaAPI)

The API is designed generally with Google Protocol Buffers. With the release of a new API version, the client libraries are automatically compiled and updated to the latest version.
The API is fundamentally backwards compatible, which means that users' applications will continue to work as usual before they also decide to move to the new version.
The API currently contains part that are not fully implemented in the backend yet. For testing please use those components that are marked with "STABLE"

- Rust API stubs: [GitHub](https://github.com/ArunaStorage/rust-api) or [crates.io](https://crates.io/crates/aruna-rust-api)
- Go API stubs: [GitHub](https://github.com/ArunaStorage/go-api)
- Python API stubs: [GitHub](https://github.com/ArunaStorage/python-api) or [PyPI](https://pypi.org/project/Aruna-Python-API)
- Java API stubs: [GitHub](https://github.com/ArunaStorage/java-api)

### **AOS Server**

[**Main Aruna Server repository**](https://github.com/ArunaStorage/ArunaServer)

The implementation of the API that handles the incoming requests.

> Aruna Object Storage is data lake application with API that manages scientific data objects according to FAIR principles.
> It interacts with multiple data storage backends (e.g. S3, File ...) via the DataProxy application and stores a rich and queryable set of metadata about stored objects in a
> postgres compatible database (e.g. CockroachDB).
> Aruna is conceptually geo-redundant with multiple dataset and database locations. Users can choose where and how to store their data.

### **AOS Data Proxy**

[**Main Aruna Data Proxy repository**](https://github.com/ArunaStorage/DataProxy)

This is the internal server implementation handling the communication between the data storage backend which is used and the specific AOS instance.

> The DataProxy service is designed to handle data transfers AOS. It starts two server: One server implements the internal proxy api from the official AOS gRPC API.
> The other server handles the transfer of the actual data. All data access is handled through presigned URLs.

### **AOS CLI**

[**Main Aruna CLI repository**](https://github.com/ArunaStorage/ArunaCLI)

> This is a simple CLI application for the ScienceObjectsDB API.
> Its currently work in progress and will be developed along with the API. Neither concept nor implementation are final.

- **Notification system**

The storage system has a notification system that can be used to receive change notification on specific resources.

The notification system is currently still in the implementation phase but will be available soon.

<!-- An example can be found here: [Notification Stream Example](#) -->

## Implementation design trivia

- An RDBMS will be used as database backend for the AOS Server
- The AOS Server, Data Proxy nd CLI will be implemented in [Rust](https://www.rust-lang.org/)
- The base API interface will be defined using [Protocol Buffers](https://developers.google.com/protocol-buffers)
- All endpoints work with JSON over HTTP just as they do with requests made via gRPC from individual clients
- [Client stubs](#aos-api) will be generated for major programming languages on every API release
- A [basic CLI client](https://github.com/ArunaStorage/ArunaCLI) will be offered to simplify the usage entry barrier
- A [web UI](https://web.aruna.nfdi-dev.gi.denbi.de/ui/) is available for demonstration purposes
