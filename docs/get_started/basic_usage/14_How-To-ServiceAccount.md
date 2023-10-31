
# How to use the ServiceAccount API / ServiceAccountServiceClient

## Introduction

AOS offers the possibility to create ServiceAccounts that can be used impersonally, 
e.g. by several users at the same time or by a service that communicates with the AOS via the API.

In order for a service account to be used against the API, it must be assigned a permission for a specific resource. 
For security reasons, a service account can only have this one permission at the same time, which can, however, be adjusted afterwards. 

The service account also must create at least a personal token for communication with the API, 
which takes over the specific permission of the service account in the authorisation process. 
If the service account is also to be entrusted with the upload and download of data, S3 credentials must be requested once from each DataProxy where data is to be stored or read. 

!!! Warning "Service Account Limitations"

    Service accounts behave like normal user accounts with the following limitations:

    * Only one permission can be assigned at the same time
    * If a permission is provided on token creation it also overwrites the existing permission
    * License and data class updates of resources are not allowed
    * Service accounts are not allowed to send requests against the following services:
        * EndpointService
        * AuthorizationService
        * UserService
        * LicenseService


## Create service account

API examples on how to create a service account.

> _Coming soon ..._


## Get service account

API examples on how to fetch information of a service account.

> _Coming soon ..._


## Set service account permissions 

API examples on how to set the specific permission of a service account.

> _Coming soon ..._


## Create service account token

API examples on how to generate a personal service account token.

If a permission is provided with this request it also overwrites the currently assigned permission.

> _Coming soon ..._


## Get service account token(s)

API examples on how to fetch information of one or multiple service account tokens.

> _Coming soon ..._


## Delete service account token(s)

API examples on how to delete one or multiple service account tokens.

> _Coming soon ..._


## Get service account S3 credentials

API examples on how to get S3 credentials for a service account from a specific DataProxy.

> _Coming soon ..._


## Get service account DataProxy token

API examples on how to generate a service account token for direct communication with a specific DataProxy.

> _Coming soon ..._


## Delete service account

API examples on how to delete a service account.

> _Coming soon ..._
