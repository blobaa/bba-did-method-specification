# BBA DID Method Specification

This document specifies the `bba` DID method as part of the [Blobaa](https://github.com/blobaa) project.

It complies with the latest version (W3C Working Draft 31 July 2020) of the generic [DID specification](https://www.w3.org/TR/2020/WD-did-core-20200731/) as specified by the [W3C Credentials Community Group](https://www.w3.org/community/credentials/).

A reference implementation handling the `bba` CRUD functions is provided in form of the [bba-did-method-handler-ts](https://github.com/blobaa/bba-did-method-handler-ts) project.


## Version

Current version: 1.0.0


## Status

This is a draft document and may be updated, replaced or obsoleted by other documents at any time. It is inappropriate to cite this document as other than a work in progress.


## Table of Contents

<details><summary><i>click to expand</i></summary>
<p><br/>

- [BBA DID Method Specification](#bba-did-method-specification)
    - [Version](#version)
    - [Status](#status)
    - [Table of Contents](#table-of-contents)
    - [Introduction](#introduction)
    - [DID Attestation Data Fields](#did-attestation-data-fields)
        - [DID Id](#did-id)
        - [Data Field Concatenation](#data-field-concatenation)
        - [Version](#version-1)
        - [State](#state)
        - [Redirect Account](#redirect-account)
        - [Storage Type](#storage-type)
            - [Ardor Cloud Storage](#ardor-cloud-storage)
        - [DID Document Template Reference](#did-document-template-reference)
        - [Misc](#misc)
    - [Method-specific DID syntax](#method-specific-did-syntax)
    - [CRUD Operations](#crud-operations)
        - [Create](#create)
            - [DID Document Template](#did-document-template)
            - [DID](#did)
        - [Update](#update)
            - [DID Document Template Update](#did-document-template-update)
            - [DID Controller Update](#did-controller-update)
        - [Deactivate](#deactivate)
        - [Read (Resolve)](#read-resolve)
    - [Security Requirements](#security-requirements)

</p>
</details>


## Introduction

The `bba` DID method aims to enable the [Ardor](https://ardorplatform.org) blockchain to act as a [DPKI](https://www.weboftrust.info/downloads/dpki.pdf) within the [SSI](https://www.manning.com/books/self-sovereign-identity) ecosystem. It runs on the independent [IGNIS](https://www.jelurida.com/ignis) child chain and utilizes Ardors [Account Properties](https://ardordocs.jelurida.com/Account_Properties) feature to manage DIDs and corresponding DID controllers. A DID controller is always represented in form of an Ardor account and by default separated from the public keys (if present) shown in a resolved DID Document. Think of a master key controlling the DID operations create, update and deactivate. A DID controller always corresponds to exactly one Ardor account, where an Ardor account can control multiple DIDs.

DID and DID Document handling is decoupled so that multiple DID Document storages can be defined and integrated to store DID Document Templates (DID Documents without a DID reference). At the current state the `bba` method defines one storage type (Ardor Cloud Storage).

In the following, `bba` method compliant account properties are called DID attestations. An account property is `bba` method compliant when it aligns to the data model described in the DID Attestation Data Fields section and is self-set. A self-set account property is a property where the sender and the receiver is the same account.


## DID Attestation Data Fields

Since the `bba` method utilizes the account property feature, most of its data fields are embedded into the *value* key/value pair of a property. In Ardor, an account property is represented as a JSON object with at least a *property* and a *value* key/value pair. An example attestation can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=0239684aef4c0d597b4ca5588f69327bed1fedfd576de35e5099c32807bb520e).


### DID Id

The only data field not embedded into the *value* key/value pair and used to determine a specific DID is the **DID Id**. It represents the value of the *property* key/value pair and is used to provide the capability to control multiple DIDs with one account. To indicate that a property is a DID attestation, the property value starts with the identifier *did://* and is followed by a unique 20 character long alphanumeric case sensitive DID Id. For example `did://EZVDomDNygTr79QXKnzE`


### Data Field Concatenation

All other fields are embedded into the 160 characters long *value* key/value pair. They consist of one or more characters concatenated into a defined order and separated by a pipe delimiter \| as shown below.

![](../draw.io/out/diagrams-bba-method-character-arrangement.svg)

*character arrangement of DID attestation data fields*


### Version

The **Version** field points to the `bba` method version. It is a three-digit number starting with 001 and needs to be incremented whenever a protocol update occurs. The current version number is 001.


### State

The **State** data field indicates the current state of an attestation. There are three state types:

An **active** state indicates that a DID is active and can be resolved.

An **inactive** state states that a DID was active in the past and is now inactive. It can no longer be resolved and is not reactivatable.

A **deprecated** state states that the DID controller is deprecated and the DID is now controlled by another controller. The new DID controller account is referenced in the redirect account data field.


The following table shows the state characters used to represent the attestation state:

| State | State Character | Brief                                                               |
|------------|:--------------------:|---------------------------------------------------------------------|
| active     |           a          | DID is active and resolvable                           |
| inactive   |           i          | DID is inactive and cannot be resolved and reactivated                       |
| deprecated |           d          | DID controller is deprecated and new controller is referenced via the redirect account field |


### Redirect Account

The **Redirect Account** data field is only used in case of a deprecated state. It points to the controller account that took over the control of the DID. To save character space, not the complete [Reed-Solomon](https://ardordocs.jelurida.com/RS_Address_Format) account representation is included in this data field. Only the significant 20 characters excluding the *ARDOR-* prefix are present. To use the redirecting account, one needs to reconstruct the Reed Solomon representation later on. In case the data field is not used, it has the dummy value of *0000-0000-0000-00000*.


### Storage Type

The storage type character indicates the storage mechanism used to store and retrieve a DID Document Template (DDOT). It must specify a mechanism to cryptographically proof that a DDOT is not tampered with between storing and retrieval, define methods to store and retrieve a DDOT, and define a reference that fits into the DDOT reference field.


#### Ardor Cloud Storage

At the current state there is only the Ardor Cloud Storage mechanism defined. It uses Ardors [Data Cloud](https://ardordocs.jelurida.com/Data_Cloud) feature to store a stringified DDOT JSON in plain text within a transaction. To be recognized as a `bba` method compliant cloud data, the cloud data name field must be set to `bba-did-document-template`. The transaction full hash value is then used as the DDOT reference and can later be used to retrieve the data cloud transaction and therefore the embedded DDOT object.


The following table shows the storage type characters used to represent the storage types:

| Storage Type | Storage Type Character | Brief                                                               |
|------------|:--------------------:|---------------------------------------------------------------------|
| Ardor Cloud Storage     |           c          | DDOT is stored in Ardors Data Cloud                          |


### DID Document Template Reference 

The DDOT reference field holds the DDOT reference used to retrieve the DID related DDOT. It cryptographically binds the DDOT to the DID. It can consist of any character sequence not longer than 120 character. The interpretation of these characters is defined by the storage mechanism.


### Misc

It should be mentioned that not all 160 characters are being used. Only 149 characters are occupied: 3 (version) + 1 (state) + 20 (redirect account) + 1 (storage type) + 120 (payload) + 4 (delimiter). The missing 11 characters are reserved for future method extensions.


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

`did:bba:t:11764950b0e8e69694831a9860256178a957c1064ca7c5dd8c44a20d384fe00c`


## CRUD Operations

The `bba` method  defines five operations to create, read, update (DDOT and DID controller) and deactivate a DID.


### Create

To create a DID, a DID Document Template is needed that holds the informations of the DID associated DID Document.


#### DID Document Template

 The difference between a resolved DID Document and the DID Document Template stored with a storage mechanism is the associated DID. The DDOT does not hold any DID informations about the associated DID. As an example, a valid DID Document Template is shown below:
````
{
    "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/v1"
    ],
    "id": "",
    "authentication": [
        {
            "id": "#z6MkjAySi1Ajf3cwZntbsmo53dmC8GX1Dm6PLmmBaC6SRuiy",
            "type": "Ed25519VerificationKey2018",
            "publicKeyBase58": "5iiQ7kvJKW8UTJ3uCCqECYDCJhF9osr2ekrFjv8RWgwb"
        }
    ]
}
````

The resolution operation is later responsible to link this template to the associates DID by adding the missing DID information into the resolved DID Document. The DID Document created based on the DDOT above and the `bba:t:11764950b0e8e69694831a9860256178a957c1064ca7c5dd8c44a20d384fe00c` DID is shown below.
````
{
    "@context": [
        "https://www.w3.org/ns/did/v1",
        "https://w3id.org/security/v1"
    ],
    "id": "did:bba:t:11764950b0e8e69694831a9860256178a957c1064ca7c5dd8c44a20d384fe00c",
    "authentication": [
        {
            "id": "did:bba:t:11764950b0e8e69694831a9860256178a957c1064ca7c5dd8c44a20d384fe00c#z6MkjAySi1Ajf3cwZntbsmo53dmC8GX1Dm6PLmmBaC6SRuiy",
            "type": "Ed25519VerificationKey2018",
            "publicKeyBase58": "5iiQ7kvJKW8UTJ3uCCqECYDCJhF9osr2ekrFjv8RWgwb"
        }
    ]
}
````

The mechanism to create a valid DID Document Template is out of the scope of this specification. A library assisting in the creation process is provided in form of the [did-document-ts](https://github.com/blobaa/did-document-ts) project.

Separating the DDOT creation process from the DID creation process has the benefit of being independent to the evolution of the DID data model and lets the `bba` method being compatible with future data models and custom contexts. A DDOT creator has to ensure that the related DID Document conforms to the DID specification.


#### DID

Creating and registering a `bba` DID involves multiple steps as shown in the figure below:

![](../plantuml/out/did-create.svg)

*DID creation workflow*

As a precondition it is assumed that the DID controller has an Ardor account created and access to the DDOT one wants to link to the DID.

The first step in the creation process is to store the DDOT with a supported storage mechanism. The Ardor Cloud Storage mechanism is used here. To store the DDOT, the DID controller must create a data cloud transaction as explained in the Ardor Cloud Storage section and save the transaction full hash `txh_s` to link the DDOT to the DID. An example transaction can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=d50168874504b75afa2880f62ef20c9a2b9b9d8e1dc846c6802fb857462a8dd5).

The next step is to register the DID. To do so, the DID controller creates a DID attestation to the Ardor account one has control of with a new generated DID Id and the storage transaction hash `txh_s` as the DDOT reference. An example transaction can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=0239684aef4c0d597b4ca5588f69327bed1fedfd576de35e5099c32807bb520e).

The last step involved is to create the DID string. The DID string can now be constructed by concatenating the `bba` DID prefix with the network character the DID was registered at and the attestation transaction full hash `txh_d` in the following way: `did:baa:<m or blank (without leading ':') for mainnet / t for testnet>:txh_d`. An example DID is `did:baa:t:0239684aef4c0d597b4ca5588f69327bed1fedfd576de35e5099c32807bb520e`.


### Update

There are two update operations defined. One for DID Document Template updates and one for DID controller updates.


#### DID Document Template Update

To change a DID Document, the corresponding DDOT needs to be updated. To do so, almost the same workflow is used as described in the Creating section and shown in the figure below. It is again assumed that a DID controller created / is in possetion of the new DDOT that should replace the current one.

![](../plantuml/out/did-document-template-update.svg)

*DDOT update workflow*


The DID controller stores the new DDOT with one of the support storage mechanisms and updates the current attestation with the new DDOT reference. An example transaction can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=26d3c955009090f971d862994beee8cc5afda82c5ed1fbd1849e09a14c1a001f).


#### DID Controller Update

To change the DID controller, two attestations are required, as shown below.

![](../plantuml/out/did-controller-update.svg)

*DID controller update workflow*

First, the current DID controller updates its attestation with the Ardor account of the new DID controller. To do so, one replaces the redirect account with the 20 account characters of the new DID controller account and sets the state character to `d`. An example transaction can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-S27P-EHWT-8D2L-937R7&chain=IGNIS&modal=transaction_info_modal&fullhash=b33dabe2232218bdbf38112e830f51bf32334d1691894bbdcd6012d8ea5ad932).

The second step is to *accept* the delegation of control by creating a DID attestion with the new controller account. This attestation must be equal to the pre updated DID attestation of the current DID controller account. An example transaction can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-YQ26-W5RK-6ATW-G9HRT&chain=IGNIS&modal=transaction_info_modal&fullhash=bf8a1f655d615df8f254c757cc710585cf5507448b620bacaf72291a014a456b).


### Deactivate

Deactivating a DID requires only one attestation update and is shown in the figure below.

![](../plantuml/out/did-deactivate.svg)

*DID deactivation workflow*

The DID controller only needs to set the state field of the DID attestation to `i` which sets the DID to inactive. An inactive DID cannot be reactivated and is locked in this state. An example transaction can be found [here](https://testardor.jelurida.com/index.html?account=ARDOR-YQ26-W5RK-6ATW-G9HRT&chain=IGNIS&modal=transaction_info_modal&fullhash=3149f135e6121534878dbd7ef6a17cf274c0ac07e282607621a2078dec148b46).


### Read (Resolve)

The DID resolution process to resolve a `baa` DID mainly consists of three task as shown below.

![](../plantuml/out/did-resolve.svg)

*DID resolution workflow*

The first task is to **retrieve the current DID attestation**. It consists of the following 9 steps.

1. retrieve the tranaction identified by the transaction full hash `txh_d` as part of the DID string
2. check if the returned transaction represents a DID attestation
3. continue with step 6.
4. get the *delegation accepted* DID attestation that was created in the DID controller update process
5. treat the new account as the current account
6. check if the DID attestation state is active
7. retrieve the DID attestations that were set after the current DID attestation
8. loop over the DID attestations found in step 7.
    1. treat the found attestation as the current attestation
    2. check if the DID attestation state is deprecated. If so, stop the loop.
    3. check if the DID attestation state is inactive. If so, stop the whole resolution process.
9.  check if the DID attestation state is active. If so, continue with the resolution process. Otherwise, continue with step 4.


The second task is to **retrieve the DID Document Template**. This is done by determining the storage mechanism indicated by the storage type field and retrieving the DDOT with the help of the DDOT reference string.


The last task is to **convert the DDOT into a DID Document** by injecting the DID string information.


After successfully performing these three task, a `bba` DID resolver is able to return a verified DID Document to the requester.


## Security Requirements

not recommended to have multiple dids controlled by one account

DID id collusion