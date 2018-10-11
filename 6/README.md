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
      * [Questions](#questions)
      * [API Groups](#api-groups)
      * [Categorization](#categorization-of-invocation-services)
      * [Parties](#parties)
      * [Use case description](#high-level-use-case-description)
      * [API  definition](#api-definition)
         * [Registering a Service](#registering-a-service)
         * [Retrieve metadata of an Service](#retrieve-metadata-of-a-service)
         * [Updating Service Metadata](#updating-service-metadata)
         * [Retiring a Service](#retiring-a-service)
         * [Invoking a Service](#invoking-a-service)
         * [Get status of invocation](#get-status)
         * [Get result of invocation](#get-result)
         * [Get proof of service delivery](#get-proof)
         * [Targeted Release](#targeted-release)
      * [Copyright Waiver](#copyright-waiver)

      
<!--te-->

<a name="service-invocation"></a>

# Introduction

Ocean protocol aims to build marketplaces that enable exchange of services that are meaningful in the data economy. Storage, compute, data cleaning, prediction, segmentation and other AIs are some examples of services that would be useful on the Ocean network. This document proposes APIs on service delivery, brokerage and consumption.

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

## What is a service

For the sake of simplicity, we can assume that services are of two types. 

A local service can be invoked via a library call, using an object oriented language:

`Service.doSomethingUseful(dataset, other arguments..)`

A remote service could be invoked via an HTTP call. 

`GET http://weatherservice/my/city`

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
 
## Questions

 - What is the difference between Invoke and Compute service
 
 - Let's use a series of examples to demonstrate this:
 
 | Service                       | Accepts                                                         | Parametrizable?      | Returns                                          |
 |-------------------------------|-----------------------------------------------------------------|----------------------|--------------------------------------------------|
 | Addition service              | 2 numbers, a and b                                              | no                   | a+b                                              |
 | Numerical computation service | a list of numbers and a function to run                         | yes, with a function | result of function applied with a list of inputs |
 | Compute service               | an environment (e.g. Docker image) and a script (e.g script.py) | yes,with a script    | the result of script execution                   |

It can be observed that a generic compute service is simply a service that is parameterizable, one that allows the user to specify a host environment
(e.g. java or Python version as a Docker image) as well as the script to run (e.g. a .jar file or .py script)
Therefore, for the rest of this discussion, we will assume that a *Compute Service* is simply a special case of an *Invokable service*

<a name="specification"></a>
## API Groups 

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

## Restrictions 

The following restrictions apply during the design/implementation of this OEP:

* The SERVICES registered in the system MUST be associated to the PROVIDER registering the services
* AGENT MUST NOT store or rely on any other information about the Services during this process

## Categorization of invocation services 

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
| Non-ocean registered assets | The invoke payload may reference data assets (e.g. URIs)that are unknown to Ocean |
| Raw payload                 | The invoke payload contains the data required to perform the service   |

### Outputs

| Examples                    | Notes                                                                                                  |
|-----------------------------|--------------------------------------------------------------------------------------------------------|
| Ocean registered assets     | The result of a service invocation may include a list of Ocean Asset IDs.                             |
| Non-ocean registered assets | The result of a service invocation may return a data payload that is not an Ocean registered Asset IDs |

### Sync vs Async

The invocation API could choose to return result of invocation synchronously(i.e. returning the result in the same call) or asynchronously .

### Level of Ocean integration

- Level 0: This is a service that is agnostic to Ocean. Example: consider a prediction service which can predict handwritten digits. It accepts a URL input, and the service takes the image at the URL and returns a prediction. 

The service may be registered on Ocean by a third party, and users pay for this service using fiat currency.
An Ocean user may want to use this service to indicate provenance (e.g prove that this prediction was indeed returned by this service.) It may be possible to create an Ocean wrapper that wraps invocations to this service (for better evidence of the provenance trail).

- Level 1: This is a service that implements the invoke service API, but does not partake in service agreements (i.e, its "verification" is a no-op). It may offer the service for free (in terms of Ocean tokens), but may demand a fiat (or other token) payment for the service. Since the service is free, there is no need for escrow.

- Level 2: Similar to Level 2, except that it is a paid service, and therefore may be subject to service agreement fulfilment/verification checks.

- Level 3: Similar to Level 3, except  that it provides an on-chain (e.g. parachain) proof of fulfillment.

| Level | Price (in ocn tokens) | service fulfilment | Implements Invoke API |
|-------|-----------------------|--------------------|-----------------------|
|     0 |                     0 | No                 | no                    |
|     1 |                     0 | no                 | yes                   |
|     2 |                    >0 | Yes(no-op)         | yes                   |
|     3 |                >0     | Yes                | yes                   |

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

# Provenance

For the perspective of provenance, a service is an entity, which interacts with other entities (such as Ocean assets) and is involved in activities. 

Specifically, a service 

* uses certain entities (e.g. takes Ocean assets as inputs)
* generates entities (e.g. on output)
* has a start and end time
* might be informed by external communication (such as events)



# API definition

- the API description provided here is a guide, and could be tailored to the idioms of the implementing language. 
- For example, object oriented implementation might allow a `job.get_result()` ,where an API with no job objects might specify `service_api.get_result(job_id)`.

## Invoking a service

### Python

- *asset.invoke(args,payload)*

| Args           | Input? | Type   | Mandatory? | Notes                                                                  |
|----------------|--------|--------|------------|------------------------------------------------------------------------|
| purchase token | input  | String | yes        | The purchase token that is provided by Ocean on purchase of this Asset |
| asset id       | input  | String | yes        | The service id                                                         |
| payload        | input  | json | yes        |   A json formatted documented conformant with a schema as defined by the service provider |
| job id         | output | String | no         | The job id used to track execution of the service                      |

### HTTP

- POST request to https://endpoint-url/asset-id?purchase-token=0xdada, with a json payload.

### Payload format

| JSON keys    | Notes                                                                                                       | Mandatory? |
| -            | -                                                                                                           | -          |
| webhook URL  | URL where the consumer can get notifications about service completion                                       | no         |
| Inputs       | A map of arguments where keys are argument names and values are strings. Key spec is in service metadata    | no         |
| Ocean Inputs | A map of Ocean asset ids required for the service. Keys are argument names as specified in service metadata | no         |
| Job config   | A map of configuration options (e.g. version of Python to use)                                              | no         |

Schemas for the payload will be provided as a json schema (not described in this doc).

## Get Status

### Python

- *job.get_status()*

### HTTP

- GET request to https://endpoint-url/job-id/status

### Returns 

| Args   | Type   | Mandatory? | Notes                                                              |
|--------|--------|------------|--------------------------------------------------------------------|
| status | String | yes        | Returns an enum with values Started, Completed, Failed, Inprogress |

## Get Result

### Python

- *job.get_result()*

Returns a dictionary with the keys/values similar to the payload schema

### HTTP

- GET request to https://endpoint-url/job-id/result

### Returns 

| Args    | Type | Mandatory? | Notes                                                                    |
|---------|------|------------|--------------------------------------------------------------------------|
| payload | json/string | yes        | Returns a result payload in the format specified in the service metadata |

## Get Proof

### Python

- *job.get_proof()*

Returns a dictionary with the keys/values similar to the payload schema

### HTTP

- GET request to https://endpoint-url/proof?job-id=<jobid>

### Returns 

| Args    | Type        | Mandatory? | Notes                                                                                                                                        |
|         |             |            |                                                                                                                                              |
|---------|-------------|------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| payload | json/string | yes        | The API returns a payload object which contains the proof that the service completed. The payload format is described by the sevice metadata |

<TODO> incorporate this:
https://github.com/DEX-Company/catamaran/wiki/API-for-DSE-interactions

## Payload definition

- Service registration
`
{ "inputs" : [{"argname" : "arg1", "argtype" : "" , "mandatory" : "true"}]"}}
`
