# Investigation

## Pre-Existing software

There are multiple services and software packages that achieve a similar goal to Vayu, each with their own drawbacks and limitations. I will use these examples as references to industry standard techniques, methods, and practices.

| Name | Description | Licenseed? | Encryption | Platform | Language | Phone-number required? | Email required? | Issues |
|------|-------------|---------|------|------------|----------|----------|-----------------------|----------------|
| [Signal](https://signal.org/) | General-purpose private messenger with end-to-end encrypted messages and calls. | Open source (AGPL-3.0) | End-to-end encrypted by default | Android, iOS, Windows, macOS, Linux | Java, Kotlin, Swift, TypeScript, C++ | Yes | No | Requires a phone number, so it does not provide Vayu's intended pseudonymous identity. |
| [Briar](https://briarproject.org/) | Messenger designed for activists, journalists and others who need communication resilient to surveillance or internet outages. | Open source (GPL-3.0) | End-to-end encrypted | Android | Java | No | No | Android only; direct communication depends on peers being able to sync over available routes. |
| [Session](https://getsession.org/) | Private messenger using account IDs, onion-routed requests and decentralised storage for offline messages. | Open source | End-to-end encrypted | Android, iOS, Windows, macOS, Linux | C++, Kotlin, Swift, TypeScript | No | No | Uses its own decentralised network and protocol. |
| [SecureDrop](https://securedrop.org/) | Whistleblower submission system for news organisations to receive documents and communicate with anonymous sources. | Open source (AGPL-3.0) | Encrypts submissions in transit and at rest | Web; self-hosted by news organisations | Python | No | No | Designed for submitting material to a newsroom, rather than general-purpose messaging. |
| [GlobaLeaks](https://globaleaks.org/) | Software for organisations to run secure whistleblowing and reporting platforms. | Open source (AGPL-3.0) | Encrypted submissions and protected access | Web; self-hosted | Python | No | No | Focuses on reporting cases to an organisation, not ongoing private conversations. |
| [WhatsApp](https://www.whatsapp.com/) | Widely used general-purpose messaging service with private messages encrypted by default. | Proprietary | End-to-end encrypted by default for personal messages | Android, iOS, Windows, macOS, Web | C++, Java, Kotlin, Swift, TypeScript | Yes | No | Requires a phone number and is a centralised commercial service. |

I will use these examples to examine identity requirements, contact verification, offline delivery, encryption standards, and retained information, in detail. This makes the presence of many open-source applications increasingly useful, as I am able to examine their source, and understand how Vayu will tackle the challanges they have already solved. 


## Signal

Signal is a useful comparison because it combines encrypted messageing with clear information about contac verification and message delivery. It [requires a phone number to register](https://support.signal.org/hc/en-us/articles/360007318691-Register-a-phone-number), although users can control whether people see that number. This is different from my aim for Vayu, as I do not want a phone number, or email to be used as a requirement to register to the service. 

Signal lets users compare a  [safety number](https://support.signal.org/hc/en-us/articles/360007060632-What-is-a-safety-number-and-why-do-I-see-that-it-changed) with a contact, ideally in person or through another trusted channel. It warns when the number changes, and a previously verified number must be approved again before sending a new message. A matching number helps check the connection to a contact, but it does not by itself prove that the person is who they claim to be. For Vayu, I will use this as a reason to show contact fingerprints, record whether they have been verified and warn about a changed key. The trusted channel still needs to be chosen by the users. 

Signal also distinguishes [sent, delivered and read](https://support.signal.org/hc/en-us/articles/360007320751-How-do-I-know-if-my-message-was-delivered-or-read) messages. A sent message has reached the service, whearas delivery means it has reached the recipients device; A read receipt depends on the users settings. Signal temporarily [queues encrypted messages](https://support.signal.org/hc/en-us/articles/5532268300186-Disappearing-Messages-with-a-Linked-Device) for offline devices. This supports my plan to distinguish relay acceptance from recipient acknowledgement in Vayu. 

## Briar 

Briar is relevant to my intended users as it is designed for private communication without a central messaging server. Its [manual](https://briarproject.org/manual/) describe direct synchronisation over Bluetooth or WiFi when the internet is enavaliable, and over Tor when it is avaliable. For ordinary direct synchronisation, a private message is delivered when both can connect. 

Briars approach reduces rliance on a central service and can work without an internet connection. In contrast, my initial Vayu design uses a relay, which is simpler for me to develop and will be able to hold messages temporarily for an offline recipient. This introduces a dependency on that relay: if it is unabvaliable, users cannot rely on it to route or collect messages. Direct peer messaging remains an optional extension of Vayu. 

Briar allows contacts to exchange links remotely or scan one another's codes in person. Its [explanation of remote contacts](https://briarproject.org/news/2019-briar-1.2-released-remote-contacts/) makes clear that users must check who suppplied a link: an attacker can impersonate a contact if both sides accept the wrong links. This reinforces the need for Vayu to explain verification rather than treating an exchanged identifier as proof of identity. Briar uses  Tor to help protect contact relationships; Vayu's initial relay design does no provide the same networkj anonymity, and the relay may observe identifiers, connection times and traffic patterns. 

## Session

Session is a clser comparison for pseudoanonymous, delayed messaging. Its [litepaper](https://getsession.org/litepaper/pdf) describes an account ID generated from a public/private key pair without a requiring personal information at registration. The public ID is used to contact the user. This shows that a service can route contact requests without asking for a phone number or email address, although an ID alone does not tell a user who controls it. Vayu will similiarly use an opaque identifier and seperate contact verification, rather than assuming that an identifier proves a real-world identity. 

Session uses a ditributed network to store ad route messages when recipients are offline. Its [network description](https://www.getsession.org/blog/introducing-the-session-network-page) explains that a group of nodes, called a swarm, stores messages for an account. Copies across several nodes provide resiliience if one is unavaliable. Session's litepaper also describes onion routing to limit what individual nodes learn about a message's route. 

# Findings for Vayu

These comparisons suggest that encrypting message contents is only one part of the problem. The project will need to consider how users verify contacts, what each delivery status actually means, how long undelivered messages are retained, and what happens if the relay stops. Signal's receipts and verification warnings are useful models; Briar highlights the value and complexity of direct communication; and Session shows both the benefits and added complexity of distributed offline delivery. 