# NEA Project plan

Vayu will be an encrypted delay-tolerant amnesic messenger. Where the server, will have no access to content of messages, and will not store any message history, with the potential for a distributed p2p network. The project will be written in Rust.

The intended client, is a spy/journalist.

### Milestone 1: Core encrypted messenger

- Phase 1: Define the initial project architecture.
- Phase 2: Decide on how keys, messages, and configuration will be stored.
- Phase 3: Set up the Rust project, command-line client and local relay.
- Phase 4: Send and receive encrypted messages between two clients. <sub>(likely on the same device)</sub>

### Milestone 2: Server-side relay

- Phase 1: Separate the local relay from the client.
- Phase 2: Add client registration and public-key lookup.
- Phase 3: Accept and forward encrypted messages.
- Phase 4: Test online delivery and invalid request, through multiple networks.

### Milestone 3: Delay-tolerence

- Phase 1: Queue messages while a node is offline.
- Phase 2: Add polling and delivery acknowledgements.<sub>(likely utilising long polling)</sub>
- Phase 3: Handle retries, duplicates and expired messages.
- Phase 4: Test disconnections and relay restarts.

### Milestone 4: Usable messenger

- Phase 1: Add contacts and clear send and inbox commands.
- Phase 2: Store message history and keys safely.
- Phase 3: Improve configuration.
- Phase 4: Try the messenger with the user.

Milestone 5: Portable storage (optional)

Milestone 6: Direct peer messaging (optional)

Milestone 7: De-reliance on librarys (optional)