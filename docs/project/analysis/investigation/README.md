# Investigation

## Pre-Existing software

There are multiple services and software packages that achieve a similar goal to Vayu, each with their own drawbacks and limitations. I will use these examples as references to industry standard techniques, methods, and practices.

| Name | Description | Licenseed? | Encryption | Platform | Language | Phone-number required? | Email required? | Issues |
|------|-------------|---------|------|------------|----------|----------|-----------------------|----------------|
| [Signal](https://signal.org/) | General-purpose private messenger with end-to-end encrypted messages and calls. | Open source (AGPL-3.0) | End-to-end encrypted by default | Android, iOS, Windows, macOS, Linux | Java, Kotlin, Swift, TypeScript, C++ | Yes | No | Requires a phone number, so it does not provide Vayu's intended pseudonymous identity. |
| [Briar](https://briarproject.org/) | Messenger designed for activists, journalists and others who need communication resilient to surveillance or internet outages. | Open source (GPL-3.0) | End-to-end encrypted | Android | Java | No | No | Android only; communication depends on peers being able to sync over available routes. |
| [Session](https://getsession.org/) | Private messenger using account IDs, onion-routed requests and decentralised storage for offline messages. | Open source | End-to-end encrypted | Android, iOS, Windows, macOS, Linux | C++, Kotlin, Swift, TypeScript | No | No | Uses its own decentralised network and protocol. |
| [SecureDrop](https://securedrop.org/) | Whistleblower submission system for news organisations to receive documents and communicate with anonymous sources. | Open source (AGPL-3.0) | Encrypts submissions in transit and at rest | Web; self-hosted by news organisations | Python | No | No | Designed for submitting material to a newsroom, rather than general-purpose messaging. |
| [GlobaLeaks](https://globaleaks.org/) | Software for organisations to run secure whistleblowing and reporting platforms. | Open source (AGPL-3.0) | Encrypted submissions and protected access | Web; self-hosted | Python | No | No | Focuses on reporting cases to an organisation, not ongoing private conversations. |
| [WhatsApp](https://www.whatsapp.com/) | Widely used general-purpose messaging service with private messages encrypted by default. | Proprietary | End-to-end encrypted by default for personal messages | Android, iOS, Windows, macOS, Web | C++, Java, Kotlin, Swift, TypeScript | Yes | No | Requires a phone number and is a centralised commercial service. |

I will use these examples to examine identity requirements, contact verification, offline delivery, encryption standards, and retained information, in detail. This makes the presence of many open-source applications increasingly useful, as I am able to examine their source, and understand how Vayu will tackle the challanges they have already solved. 

