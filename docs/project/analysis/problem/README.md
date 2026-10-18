# Problem description

I am building Vayu as an encrypted, delay-tolerant messenger, with journalists, whistleblowers and "spies" in mind. I want people to communicate securely, and privately, without giving the service their name, phone number or email address, or trusting it with their messages.

## Historical background

The need to protect a message has been a staple of human communication for centuries, the issues of privacy and security have existed long before the concept of computers, and the internet, and have only become more complex and prevelant with the rise of digital communication. 

The ideas of the "right to privacy", and the continued development of encryption and secure communication methods have been a constant struggle between those who wish to protect their communications, and those who witsh to intercept.

The earliest known use of encryption was in the 600BC. It worked by reversing the alphabet, so the first letter swaps with the last, the second with the second to last, and so on. In Hebrew, this pairs Aleph (A), and Beth (B), with Shin (Sh), giving the cipher its name "Atbash". Historically it is most famous for its use in the biblical book of jeremiah, where it was used to disguise the names of Babylonian kings. 

100 years later, the famous Caesar cipher, which is a simple letter substitution cipher, was used by Julius Caesar to protect his private letters. 

By the Second World War, the Enigma machine was used by the Germans to encrypt their communications. The machine used a series of rotors to create a complex substitution cipher, which at the time was thought to be unbreakable. in 1940, Alan Turing and his team at Bletchley Park were able to rbeak the enigma code, by using a guessed piece of the original message, known as a *crib*, to rule out impossible rotor settings. 

In 1977, Ron Rivest, Adi Shamir and Leonard Adleman described the system known as RSA. It allowed someone to publish an encryption key, while keeping the corresponding decryption key private. This addressed the difficulty of giving a sender a way to encrypt a message without first sharing a secret key through a secure channel. 

Phil Zimmermann brought these ideas to personal email with the release of Pretty Good Privacy (PGP) in 1991. In [*Why I Wrote PGP*](https://www.philzimmermann.com/EN/essays/WhyIWrotePGP.html), he argued for ordinary people's ability to protect private conversations, including communication with political dissidents. His comparison between an encrypted email and a letter inside an envolope makes the purpous clear: wanting privacy does not make a conversation suspicious. 

For Vayu, this history connects the mathematics of encryption with the practical problem of communicating privately. I want to use established cryptography, while also addressing contact verification, delivery and usablity.

## Intended users

I have journalists and confidential sources in mind. For example a journalist and a source might connect briefly at different times of day. A message sent during one person's connection should remain avaliable for the other person to collect later. This is an intended use case which i will use to guide the design and testing. 

The aspect of pseudonymity is also important. I want to allow people to communicate without giving the service their name, phone number or email address.

These users also need a way to establish who they are communicating wivth. Encrypting a message is only useful if it is encrypted for the intended persons's key. I therefore need to consider contact verification alongside protecting message contents.

Choosing this intended user group allows me to focus on a specific set of requirements, which will be easy to incrementally implement. This is outlined in [plan.md](/docs/project/plan.md).

## Communication and delivery

The main challange is combining provate communication with delayed delivery. The rela needs enough information to route a message, and eventually to hold it while the recipient is offline, without needing access to its contents. I will need to distinguish temporary storage for delivery from a permanent  record of conversations. 

Interrupted connections also make delivery uncertain. A sender might rery because a connection closes before a recipient arrives, or a recipient might receive the same message more than once. I want Vayu to be able to handle these situations without silently losing accepted messages or showing repeated copies as new messages. he user should be able to distinguish a message accepted by the relay from one acknowledged by the recipient. 

## Architecture and identity

The **client** will manage keys, encrypt messages and read them. The **relay** will route encrypted envelopes. I will start with two command-line clients and a local relay, using separate Rust modules.

```mermaid
flowchart LR
    subgraph sender["Sender's device"]
        compose["Write a message"] --> encrypt["Encrypt and authenticate"]
        senderKeys["Private keys stay here"] --> encrypt
    end
    subgraph server["Relay: not trusted with message contents"]
        route["Route encrypted envelopes"]
        queue["Temporary encrypted queue<br/>Added in Milestone 3"]
        route --> queue
    end
    subgraph recipient["Recipient's device"]
        decrypt["Verify and decrypt"] --> read["Read the message"]
        recipientKeys["Private keys stay here"] --> decrypt
    end
    encrypt -->|"Encrypted envelope"| route
    route -->|"Online delivery"| decrypt
    queue -->|"Delivery after reconnecting"| decrypt
```

Envelopes will carry ciphertext, routing details, delivery identifiers and expiry. Relevant fields will be authenticated against tampering. Private keys and readable messages will stay on clients.

Clients will create keys locally and register an opaque identifier with proof of key ownership. Contact names will stay local. Key replacement, collection and acknowledgement will require authentication and replay protection.

I will verify contact key fingerprints outside the relay through a trusted channel. Later key lookup must still warn about changed keys.

This gives pseudonymity, but does not hide IP addresses, connection times or relationships between identifiers. I will minimise retained metadata and avoid logging secrets or contact details. Network anonymity, such as Tor, remains optional.

## Encryption

Messages will be encrypted and decrypted by clients. Remote connections will use TLS with certificate verification. I will use established libraries, secure randomness and authenticated encryption, rejecting invalid messages without falling back to readable text.

My initial choice is **MLS through [OpenMLS](https://docs.rs/openmls/0.9.0/openmls/)** for two-client conversations.

Identities will use signing keys and credentials containing opaque identifiers. Contacts will pin verified keys. Public [key packages](https://book.openmls.tech/user_manual/create_key_package.html) allow offline conversation setup; their private parts (ha funny) stay on the recipient's device.

MLS key-state changes need consistent ordering. Delayed messages need bounded retention of old keys, trading some [forward secrecy](https://book.openmls.tech/forward_secrecy.html) for delayed delivery. I will define limits alongside expiry. Relay access control and receipts remain Vayu's responsibility; I will not claim cryptographic deniability.

## Offline delivery

In Milestone 3, the relay will queue encrypted messages until expiry or authenticated recipient acknowledgement, then remove them. Duplicate records will have bounded lifetimes. The queue will be temporary.

```mermaid
sequenceDiagram
    participant A as Sender
    participant R as Relay
    participant B as Recipient
    A->>A: Assign a message identifier and encrypt once
    A->>R: Submit encrypted envelope
    R-->>A: Confirm queue acceptance
    Note over R,B: Recipient is offline#59; relay retains ciphertext until expiry
    B->>R: Reconnect and authenticate a poll
    R-->>B: Return queued envelope
    B->>B: Verify, decrypt and accept#59; suppress duplicates
    B->>R: Authenticate acknowledgement
    R->>R: Remove queued copy
    Note over A,B: A relay receipt alone does not prove recipient acceptance
```

## Client storage

Each client will use a selected profile directory. Keys, contacts and MLS state will go in **SQLite encrypted with [SQLCipher](https://www.zetetic.net/sqlcipher/design/)**, unlocked through a hidden passphrase prompt. I will reject unencrypted SQLite and keep passphrases out of configuration, arguments and logs.

I plan to use OpenMLS's SQLite provider with SQLCipher, using Cargo-locked versions. Non-secret settings will go in `config.toml`. The relay will initially hold state in memory and lose it on restart.

Old keys must be removed when no longer needed. Encryption does not prevent restoring an old database, so restored profiles will need fresh conversations. Secure deletion remains a limitation. Portable storage and memory-only operation remain optional and will need checks for logs, temporary files, swap and crash dumps.

## Milestones and checks

- **Milestone 1:** local encrypted exchange; check tampering, wrong recipient keys, profile reload and wrong passphrases.
- **Milestone 2:** separate network relay, TLS and access control.
- **Milestone 3:** offline delivery, expiry, retries, duplicates, delayed key-state changes and restart recovery.
- **Milestone 4:** usable contacts, configuration and storage, including persistent history.
