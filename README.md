# bba-did-method-specification

A DID method based on the Ardor blockchain.

This repository contains the `bba` DID method specification. It uses the Ardor blockchain as a decentralized public-key infrastructure (DPKI) for creating and managing decentralized identifiers (DIDs). A DID is a self-controlled persistent identifier and can be resolved into a DID Document. A DID document contains publicly available information (usually in the form of public keys and service endpoints for further interaction) associated with a DID-representing entity. Because this information is cryptographically linked to a DID, a DID controller may use it for authentication, assertion and/or proof of personally identifiable information (PII). PIIs are generally not stored in DID documents but are embedded in Verifiable Credentials (VCs). VCs and DIDs are the pillars of Self-Sovereign Identity (SSI).

The `bba` DID method is listed in the official DID method registry.


## Resources

### BBA Related

- [DID Method Specification](docs/markdown/spec.md)
- [Reference Implementation](https://github.com/blobaa/bba-did-method-handler-ts)
- [Web UI for CRUD Operations](https://wubco.blobaa.dev)
- [DID Driver](https://github.com/blobaa/bba-did-driver) for the [Universal Resolver](https://dev.uniresolver.io)


### General

- [DID Method Registry](https://w3c.github.io/did-spec-registries/#did-methods)
- [DID Core Specification](https://www.w3.org/TR/did-core/)
- [VC Specification](https://www.w3.org/TR/vc-data-model/)
- [SSI Book](https://www.manning.com/books/self-sovereign-identity)
- [Trust over IP Stack](https://trustoverip.org/wp-content/uploads/sites/98/2020/05/toip_introduction_050520.pdf)
- [DPKI Whitepaper](https://www.weboftrust.info/downloads/dpki.pdf)


## Contributing

PRs accepted.

If editing the Readme, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.


## License

[MIT](./LICENSE) © Attila Aldemir
