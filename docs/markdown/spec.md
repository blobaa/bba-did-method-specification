# BBA DID Method Specification

This document specifies the `bba` DID method as part of the Blobaa project.

It complies with the latest version (W3C Working Draft 31 July 2020) of the generic [DID specification](https://www.w3.org/TR/2020/WD-did-core-20200731/) as specified by the [W3C Credentials Community Group](https://www.w3.org/community/credentials/).

A reference implementation handling `bba` CRUD functions is provided in form of the [bba-did-method-handler-ts](https://github.com/blobaa/bba-did-method-handler-ts) library.


## Version

Current version: 1.0.0


## Table of Contents

<details><summary><i>click to expand</i></summary>
<p><br/>

- [BBA DID Method Specification](#bba-did-method-specification)
    - [Version](#version)
    - [Table of Contents](#table-of-contents)
    - [Introduction](#introduction)
    - [Method Data Fields](#method-data-fields)
        - [DID Id](#did-id)
        - [Data Field Concatenation](#data-field-concatenation)
        - [Version](#version-1)
        - [State](#state)
        - [Redirect Account](#redirect-account)
        - [Storage Type](#storage-type)
            - [Ardor Cloud Storage](#ardor-cloud-storage)
        - [DID Document Template Reference](#did-document-template-reference)
    - [Method-specific DID syntax](#method-specific-did-syntax)
    - [CRUD](#crud)
        - [Create](#create)
        - [Read](#read)
        - [Update](#update)
        - [Deactivate](#deactivate)
        - [Example](#example)
    - [Security Requirements](#security-requirements)

</p>
</details>


## Introduction

The `bba` DID method aims to enable the [Ardor](https://ardorplatform.org) blockchain to act as a [DPKI](https://www.weboftrust.info/downloads/dpki.pdf) within the [SSI](https://www.manning.com/books/self-sovereign-identity) ecosystem. It runs on the independent [IGNIS](https://www.jelurida.com/ignis) child chain and utilizes Ardors [Account Properties](https://ardordocs.jelurida.com/Account_Properties) feature to manage DIDs and corresponding DID controllers. A DID controller is always represented in form of an Ardor account and by default separated from the public keys (if present) shown in a resolved DID Document. Think of a master key controlling the DID operations create, update, deactivate. A DID controller always corresponds to exactly one Ardor account, where an Ardor account can control multiple DIDs.

DID and DID Document handlings are decoupled so that multiple DID Document storages can be defined and integrated to store DID Document Templates (DID Documents without a DID reference). At the current state the `bba` method defines one storage type (the Ardor internal [Data Cloud](https://ardordocs.jelurida.com/Data_Cloud)).

In the following, creating and updating account properties is called attestation since doing so requires a blockchain transaction and states that the issuing account intentionally attests properties to the receiver account (which can also be the same account).


## Method Data Fields

Since the `bba` method utilizes the account property feature, most of its data fields are embedded into the *value* key/value pair of a property. In Ardor, an account property is represented as a JSON object with at least a *property* and a *value* key/value pair. An example attestation can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=0239684aef4c0d597b4ca5588f69327bed1fedfd576de35e5099c32807bb520e).


### DID Id

The only data field not embedded into the *value* key/value pair and used to determine a specific DID is the **DID Id**. It represents the value of the *property* key/value pair and is used to provide the option to control multiple DIDs with one account. To indicate that a property is part of the `bba` DID method, the property value starts with the identifier *did://* and is followed by a unique 20 character long alphanumeric case sensitive DID Id. For example `did://EZVDomDNygTr79QXKnzE`


### Data Field Concatenation

All other fields are embedded into the 160 characters long *value* key/value pair. They consist of one or more characters concatenated into a defined order and separated by a pipe character \|.

![](../draw.io/out/diagrams-bba-method-character-arrangement.svg)

*character arrangement of bba DID method data fields*


### Version

The **Version** field points to the `bba` method version. It is a three-digit number starting with 001 and needs to be incremented whenever a protocol update occurs. The current version number is 001.


### State

The **State** data field indicates the current state of an attestation. There are three state types:

An **active** state indicates that a DID is active and can be resolved.

An **inactive** state states that a DID was active in the past and is now inactive. It can no longer be resolved and is not reactivatable.

A **deprecated** state states that the DID controller is deprecated and the DID is now controlled by another controller. The new DID controller account is referenced in the redirect account data field of the corresponding attestation.


The following table shows the state characters used to represent the attestation state:

| State | State Character | Brief                                                               |
|------------|:--------------------:|---------------------------------------------------------------------|
| active     |           a          | DID is active and resolvable chain                          |
| inactive   |           i          | DID is inactive and cannot be resolved and reactivated                       |
| deprecated |           d          | DID controller is deprecated and new controller is referenced via the redirect account field |


### Redirect Account

As explained in the state type section, the **Redirect Account** data field is only used in case of a deprecated state. It points to the controller account that took over the control of a specific DID. To save character space, not the complete [Reed-Solomon](https://ardordocs.jelurida.com/RS_Address_Format) account representation is included in this data field. Only the significant 20 characters excluding the *ARDOR-* prefix are present. To use the redirecting account, one needs to reconstruct the Reed Solomon representation later on. In case the data field is not used, it has the dummy value of *0000-0000-0000-00000*.


### Storage Type

The storage type character indicates the storage mechanism used to store and retrieve a DID Document Template (DDOT). It must specify a mechanism to cryptographically proof that a DDOT is not tampered with between storing and retrieval, define methods to store and retrieve a DDOT, and define a reference that fits into the DDOT reference field.


#### Ardor Cloud Storage

At the current state there is only the Ardor cloud storage mechanism defined. It uses the Data Cloud feature to store a stringified DDOT JSON in plain text within a transaction. To be recognized as `bba` method compliant cloud data, the cloud data name field must be set to `bba-did-document-template`. The transaction full hash value is then used as the DDOT reference and can later be used to retrieve the data cloud transaction and therefore the embedded DDOT object.


The following table shows the storage type characters used to represent the storage types:

| Storage Type | Storage Type Character | Brief                                                               |
|------------|:--------------------:|---------------------------------------------------------------------|
| Ardor Cloud Storage     |           c          | DDOT is stored in Ardors Data Cloud                          |


### DID Document Template Reference 

The DDOT reference field holds the DDOT reference used to retrieve the actual DDOT. It cryptographically binds the DDOT to the DID. It can consist of any character sequence not longer than 120 character. The interpretation of these characters is defined by the storage mechanism determined by the storage type field.


## Method-specific DID syntax

The following ABNF defines the bba-specific DID scheme:

```ABNF
bba-did = "did:bba:" bba-identifier
bba-identifier = [ ardor-network ":" ] ardor-tx-hash
ardor-network = "m" / "t"
ardor-tx-hash = 64HEXDIG
```

`bba` DIDs can either point to Ardor`s testnet or mainnet. If no network is specified, the mainnet is assumed.

A `bba` DID includes the full hash of the first attestation transaction as a starting point and identifier. Due to the transaction hash as unique identifier, it can be assumed that `bba` DIDs are globally unique.

Example DIDs:

**mainnet**

`did:bba:fd8127c808552656bf3986a42884bd9ffc459fb5d71aec48e7535336a6191bf6`
</br>
or
</br>
`did:bba:m:fd8127c808552656bf3986a42884bd9ffc459fb5d71aec48e7535336a6191bf6`

**testnet**

`did:bba:t:fd8127c808552656bf3986a42884bd9ffc459fb5d71aec48e7535336a6191bf6`



## CRUD

### Create

![](../plantuml/out/did-create.svg)


### Read

![](../plantuml/out/did-read.svg)


### Update

![](../plantuml/out/did-update.svg)


### Deactivate

![](../plantuml/out/did-delete.svg)


### Example

1. Create
    1. Data Cloud
        1. d50168874504b75afa2880f62ef20c9a2b9b9d8e1dc846c6802fb857462a8dd5
    2. Account Prop
        1. 0239684aef4c0d597b4ca5588f69327bed1fedfd576de35e5099c32807bb520e
2. Update Payload
    1. 

## Security Requirements

not recommended to have multiple dids controlled by one account


https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=0239684aef4c0d597b4ca5588f69327bed1fedfd576de35e5099c32807bb520e