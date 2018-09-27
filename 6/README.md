```
shortname: 6/CSAPI
name: API to register & invoke services
type: Standard
status: Raw
editor: Mike Anderson <mike.anderson@dex.sg>
editor: Kiran Karkera <kiran.karkera@dex.sg>
contributors: 
```

<!--ts-->

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Service Invocation](#service-invocation)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Specification](#specification)
         * [Proposed Solution](#proposed-solution)
         * [Registering a Service](#registering-a-service)
         * [Retrieve metadata of an Service](#retrieve-metadata-of-a-service)
         * [Updating Service Metadata](#updating-service-metadata)
         * [Retiring a Service](#retiring-a-service)
         * [Invoking a Service](#invoking-a-service)
         * [Providing Service proof](#proving-a-service)
         * [Consuming Service results](#consuming-service-results)
         * [Targeted Release](#targeted-release)
      * [Copyright Waiver](#copyright-waiver)

      
<!--te-->

<a name="service-invocation"></a>
# Service Invocation

The Service Invocation API (**INVOKE**) is a specification for the Ocean Protocol to register and invoke services.

Compute services are defined as services available on the Ocean Network that

* Can be invoked by any Ocean Agent connected to the Ocean Network (subject to access restrictions)
* May accept one or more Input parameters (which will typically be data assets to be used or algorithms to be run)
* Typically produce one or more Outputs (which will typically be references to generated data assets)
* Support the provision of proofs by service providers upon service completion (after which tokens in escrow may be released) 

This OEP does not prescribe the exact type of services offered. It is open to service provider implementations to define there, providing that they conform with this API specification
This OEP does not cover service discovery.
The OEP is agnostic to the location of service delivery records, whether on, or off chain.

### Examples of services

- *Data cleaning service* which removes outliers and empty instances from a dataset
- *Model training service* which trains a model on a dataset
- *prediction service*, which given a trained model, returns predictions on a new instances
- *Data retrieval services* (e.g. Google Bigquery)

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper), [3/ARCH](../3/README.md), [4/KEEPER](../4/README.md) and [5/AGENT](../5/README.md).


<a name="change-process"></a>
## Change Process
This document is governed by the [2/COSS](../2/README.md) (COSS).

<a name="language"></a>
## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

<a name="motivation"></a>
## Motivation

Ocean network aims to power marketplaces for relevant AI-related data services.
There is a need for a standardised **interface** for the invocation of services so that different implementations can be provided and invoked by users of the Ocean Protocol.

Requirements are:

* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER
* SERVICES are compute services according to the protocol defined in this OEP
* PROVIDERS register and offer SERVICES 
* PROVIDERS may publish SERVICE METADATA relating to the service offered
* CONSUMER can identify services with a unique ID on the Ocean Network 
* CONSUMER can invoke services via any OCEAN AGENT (subject to contract and access requirements) or directly on the service endpoint
* SERICES may require INPUTS 
* SERICES may produce OUTPUTS
* INPUT ASSETS must be available for the service provider to consume
* SERVICES may fail, in which case failure should be reported with relevant information to the CONSUMER
* PROVIDER provides SERVICE and optionally, a PROOF 
* VERIFIER validates PROOF
* SERVICE CONTRACT must be settled and any tokens transfered after receipt of valid PROOF
* OUTPUT ASSETS, if created, must be identified and communicated to the CONSUMER. These assets may be registered as Ocean Assets.
  
<a name="specification"></a>
## Specification 

The **Service Metadata** information should be managed using an API on the Ocean. 

This API exposes the following capabilities in the Ocean Agent related to different functionality groups:

### Service management 

* Registering a new Service (the same as registering an asset)
* Retrieve metadata information of an Service
* Update the metadata of an existing Service 
* Retire a Service

### Service invocation

* Invoke the service
* Get the result of a service invocation
* Get the status of a service invocation

### Proof of service delivery

* Get the proof of service delivery completion

---

The following restrictions apply during the design/implementation of this OEP:

* The SERVICES registered in the system MUST be associated to the PROVIDER registering the services
* AGENT MUST NOT store or rely on any other information about the Services during this process

## Types

An invocation service can be categorized on these dimensions:

### Invocation endpoint

| Examples              | Notes                                                                      |
|-----------------------|----------------------------------------------------------------------------|
| Ocean Keeper node     | Invoke service may be invoked via a Squid API                          |
| Service provider node | The service may be invoked on a service URL (e.g. HTTP GET on http://service-endpoint.com/ ) after getting the access token |

### Inputs 

| Examples                    | Notes                                                                  |
|-----------------------------|------------------------------------------------------------------------|
| Ocean registered assets     | The invoke payload may provide a list of Ocean Asset_IDs               |
| Non-ocean registered assets | The invoke payload may reference data assets that are unknown to Ocean |

### Outputs

| Examples                    | Notes                                                                                                  |
|-----------------------------|--------------------------------------------------------------------------------------------------------|
| Ocean registered assets     | The result of a service invocation may include a list of Ocean Asset IDs.                             |
| Non-ocean registered assets | The result of a service invocation may return a data payload that is not an Ocean registered Asset IDs |

### Sync vs Async

The invocation API could choose to return result of invocation synchronously(i.e. returning the result in the same call) or asynchronously .

# Proposed Solution

## Parties

Invokable services involve transations with 3 parties

* Ocean (Keepers)
* (Invokable) service provider
* Service consumer

## High level use case description

### Use case 1: paid service

Consumption of a service involves 4 phases:

1. Registration

- Service provider registers service with Ocean

2. Purchase

- Consumer searches Ocean for service providers
- Ocean returns list of service providers
- Consumer chooses a service provider, and buys a particular service (invocation type)

3. Invocation

- Consumer invokes the job on the service provider 
- Consumer checks if job completed successfully. 

4. Consumption

- Consumer retrieves new asset from Ocean to examine results.

5. Service delivery verification

- Service provider provides proof of service delivery
- Write service agreement proofs to chain.

### Use case 2: free service

1. Registration

- Service provider registers service with Ocean

2. Purchase

- Consumer searches Ocean for service providers
- Ocean returns list of service providers

3. Invocation

- Consumer invokes the job on the service provider 
- Consumer checks if job completed successfully. 


# API definition


## Invoking a service

### Python

- *asset.invoke(args,payload)*

| Args           | Input? | Type | Mandatory?                                                             | Notes |
|----------------|----|---|------------------------------------------------------------------------|-------|
| purchase token | input| String | yes  |The purchase token that is provided by Ocean on purchase of this Asset       |
| asset id      | input| String |  yes  | The service id |
| job id         | output | String   | no    | The job id used to track execution of the service
|                |        |                                                                        |       |
|                |        |                                                                        |       |

### HTTP

- POST request to https://endpoint-url/invoke?asset-id=0xcafa&purchase-token=0xdada, with a json payload.

### Payload format

| JSON keys    | Notes                                                                                                       | Mandatory? |
| -            | -                                                                                                           | -          |
| webhook URL  | URL where the consumer can get notifications about service completion                                       | no         |
| Inputs       | A map of arguments where keys are argument names and values are strings. Key spec is in service metadata    | no         |
| Ocean Inputs | A map of Ocean asset ids required for the service. Keys are argument names as specified in service metadata | no         |
| Job config   | A map of configuration options (e.g. version of Python to use)                                              | no         |



## Get Status

### Python

- *job.get_status()*

### HTTP

- GET request to https://endpoint-url/invoke?job-id=<yourjobid>

### Returns 

| Args   | Type   | Mandatory? | Notes                                                              |
|--------|--------|------------|--------------------------------------------------------------------|
| status | String | yes        | Returns an enum with values Started, Completed, Failed, Inprogress |


## Get Result

### Python

- *job.get_result()*

### HTTP

- GET request to https://endpoint-url/result?job-id=<jobid>

### Returns 

| Args    | Type | Mandatory? | Notes                                                                    |
|---------|------|------------|--------------------------------------------------------------------------|
| payload | json | yes        | Returns a result payload in the format specified in the service metadata |
|         |      |            |                                                                          |

## Get Proof

### Python

- *job.get_proof()*

### HTTP

- GET request to https://endpoint-url/proof?job-id=<jobid>

### Returns 

| Args    | Type | Mandatory? | Notes                                                                                                                                        |
|---------|------|------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| payload | json | yes        | The API returns a payload object which contains the proof that the service completed. The payload format is described by the sevice metadata |
|         |      |            |                                                                                                                                              |
