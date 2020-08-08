# bba-did-method-specification

A DID method based on the Ardor blockchain.

This repository contains the `bba` DID method specification. It uses the Ardor blockchain as a decentralized public key infrastructure (DPKI) to create and manage decentralized identifiers (DIDs). A DID is a self controlled persistent identifier and is resolvable to a DID Document. A DID Document contains public available informations (usually in the form of public keys and service endpoints for further interaction) associated to a DID representing entity. Because these informations are cryptographically bound to a DID, a DID controller can use these informations for authentication, assertions and/or proofs of personal identifiable informations (PII). PIIs are usually not stored inside DID Documents, but embedded into Verifiable Credentials (VCs). VCs and DIDs are the pillars of self-sovereign identity (SSI).

This repository contains the `bba` DID method specification. It uses the Ardor blockchain as a decentralized public-key infrastructure (DPKI) for creating and managing decentralized identifiers (DIDs). A DID is a self-controlled persistent identifier and can be resolved into a DID document. A DID document contains publicly available information (usually in the form of public keys and service endpoints for further interaction) associated with a DID-representing entity. Because this information is cryptographically tied to a DID, a DID controller can use this information for authentication, assertion and/or proof of personally identifiable informations (PIIs). PIIs are typically not stored in DID Documents but embedded into Verifiable Credentials (VCs). VCs and DIDs are the pillars of Self-Sovereign Identity (SSI).


The `bba` DID method was added to the official DID method register.


## Resources

### BBA Related

- [BBA Specification](docs/markdown/spec.md)
- [Reference Implementation](https://github.com/blobaa/bba-did-method-handler-ts)
- [Web UI for CRUD Operations](https://dev.uniresolver.io)
<!-- - [Universal Resolver](https://dev.uniresolver.io) -->


### General

- [DID Core Specification](https://www.w3.org/TR/did-core/)
- [DID Method Registry](https://w3c.github.io/did-spec-registries/#did-methods)
- [VC Specification](https://www.w3.org/TR/vc-data-model/)
- [SSI Book](https://www.manning.com/books/self-sovereign-identity)
- [Trust over IP Stack](https://trustoverip.org/wp-content/uploads/sites/98/2020/05/toip_introduction_050520.pdf)
- [DPKI Whitepaper](https://www.weboftrust.info/downloads/dpki.pdf)

## Contributing

PRs accepted.

If editing the Readme, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.


## License

[MIT](./LICENSE) © Attila Aldemir
