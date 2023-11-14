
# How to use the License API / LicenseServiceClient

## Introduction

Licences play a major role in the implementation of/compliance with the FAIR principles, especially when it comes to the conditions of data reusability. A licence should clearly describe for humans and machines which rights of use apply to the data, which also implements the legal side of interoperability.

For this purpose, a predefined set of the most common licences is provided in the AOS:

* All Rights Reserved: `AllRightsReserved`
* All Creative Commons 4.0: `CC0`, `CC-BY-4.0`, `CC-BY-SA-4.0`, `CC-BY-NC-4.0`, `CC-BY-NC-SA-4.0`, `CC-BY-ND-4.0`, `CC-BY-ND-SA-4.0`
* MIT license: `MIT`
* Apache 2.0: `Apache-2.0`
* GNU General Public License: `GPLv3`, `GPLv2`

Each object in the AOS can be assigned different licences for the metadata and "physical" data, so that a granular definition of the rights of use can be specified. The specification of licences is only mandatory when a project is created. The reason for this is that all other sub-resources can inherit the licences of their parent if no licence is specified when they are created.

!!! Warning "Default license"

    If no licences are specified when a Project is created, the default licence `AllRightsReserved` is used. 
    This is the only licence that can be replaced on a resource without creating a new revision.

    Other licences of an object cannot be modified "in-place" afterwards, but are applied to a new revision of a resource. 
    Some thought should be given in advance as to which licence is appropriate for the data.

## Create license

New licenses can be created without any limitations.

You should just check twice if the license information is correct as there is no way to edit the created license afterwards.

??? Abstract "Required permissions"

    To create a new license you only have to be a registered AOS user.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to create a new license
    curl '
      {
        "tag": "CC-BY-SA-4.0",
        "name": "Creative Commons Attribution-ShareAlike 4.0 International",
        "text": "[Full CC-BY-SA-4.0 license text]",
        "url": "https://creativecommons.org/licenses/by-sa/4.0/"
      }' \
         -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X POST https://<URL-to-AOS-instance-API-gateway>/v2/license
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to create a new license
    let request = CreateLicenseRequest {
        tag: "CC-BY-SA-4.0".to_string(),
        name: "Creative Commons Attribution-ShareAlike 4.0 International".to_string(),
        text: "[Full CC-BY-SA-4.0 license text]".to_string(),
        url: "https://creativecommons.org/licenses/by-sa/4.0/".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = license_client.create_license(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new license
    request = CreateLicenseRequest(
        tag="CC-BY-SA-4.0",
        name="Creative Commons Attribution-ShareAlike 4.0 International",
        text="[Full CC-BY-SA-4.0 license text]",
        url="https://creativecommons.org/licenses/by-sa/4.0/",
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.info_client.CreateLicense(request=request)

    # Do something with the response
    print(f'{response}')
    ```

## Get specific license

Licenses can be easily fetched via their tag name.

??? Abstract "Required permissions"

    To create a new license you only have to be a registered AOS user.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to get a specific license
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/license/{license-tag}
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to get a specific license
    let request = GetLicenseRequest {
        tag: "CC-BY-SA-4.0".to_string(),
    };
    
    // Send the request to the AOS instance gRPC gateway
    let response = license_client.get_license(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to create a new license
    request = GetLicenseRequest(
        tag="CC-BY-SA-4.0"
    )

    # Send the request to the AOS instance gRPC gateway
    response = client.info_client.GetLicense(request=request)

    # Do something with the response
    print(f'{response}')
    ```


## List all available licenses

API examples of how to fetch all available license for an overview if a fitting license already exists.

??? Abstract "Required permissions"

    To create a new license you only have to be a registered AOS user.

=== ":simple-curl: cURL"

    ```bash linenums="1"
    # Native JSON request to fetch all available licenses
    curl -H 'Authorization: Bearer <AUTH_TOKEN>' \
         -H 'Content-Type: application/json' \
         -X GET https://<URL-to-AOS-instance-API-gateway>/v2/licenses
    ```

=== ":simple-rust: Rust"

    ```rust linenums="1"
    // Create tonic/ArunaAPI request to fetch all available licenses
    let request = ListLicensesRequest {};
    
    // Send the request to the AOS instance gRPC gateway
    let response = license_client.list_licenses(request)
                                 .await
                                 .unwrap()
                                 .into_inner();
    
    // Do something with the response
    println!("{:#?}", response);
    ```

=== ":simple-python: Python"

    ```python linenums="1"
    # Create tonic/ArunaAPI request to fetch all available licenses
    request = GetLicenseRequest()

    # Send the request to the AOS instance gRPC gateway
    response = client.info_client.GetLicenses(request=request)

    # Do something with the response
    print(f'{response}')
    ```
