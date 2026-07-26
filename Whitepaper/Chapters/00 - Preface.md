# Preface

OpenFiat began with a simple observation: despite more than a decade of innovation in blockchain technology, exchanging digital assets for local fiat currency still requires users to place significant trust in centralized organizations.

Today, most peer-to-peer cryptocurrency marketplaces operate through centralized servers, centralized databases, centralized moderation teams, and centralized infrastructure. While blockchain technology has successfully decentralized the custody and transfer of digital assets, the marketplace connecting buyers and sellers often remains under the control of a single company.

This creates an important contradiction. Users may own their cryptocurrency without relying on an intermediary, yet they still depend on that intermediary to discover trading partners, publish advertisements, resolve disputes, maintain reputation systems, operate communication channels, and keep the marketplace online.

History has repeatedly shown that centralized services can fail. Companies can close, become insolvent, suffer prolonged outages, change their policies, censor users, restrict access in certain regions, or become targets of cyberattacks. When this happens, an entire marketplace can disappear even though the underlying blockchain continues operating normally.

OpenFiat was created to address this problem.

Rather than building another centralized exchange, OpenFiat is designed as an open protocol. Anyone can develop compatible software, operate network infrastructure, build user interfaces, provide supporting services, or participate in governance without requiring permission from AllenHark or any other organization.

The protocol separates asset settlement from marketplace coordination. Solana provides secure, transparent, and highly performant on-chain settlement through audited smart contracts, while OpenFiat provides a decentralized peer-to-peer coordination network responsible for advertisements, trade discovery, encrypted communication, reputation, governance, notifications, and other marketplace services.

This separation allows each layer to focus on what it does best. Solana secures assets and executes immutable smart contracts. OpenFiat coordinates human interaction efficiently without unnecessarily placing every marketplace event on-chain.

Throughout this document, every architectural decision follows several fundamental principles:

* Users should always control their own assets.
* Participation should remain permissionless.
* Every important action should be cryptographically verifiable.
* Reputation should be earned through observable behavior rather than assigned by administrators.
* Protocol rules should replace centralized moderation wherever practical.
* The network should continue operating even if its original creators cease participating.

Although AllenHark is leading the initial development of OpenFiat and intends to fund the protocol's early growth, the long-term objective is progressive decentralization. Responsibility for infrastructure, governance, ecosystem development, and protocol evolution is expected to transition gradually to a diverse global community of participants.

This whitepaper serves as the first formal specification of OpenFiat Version 1.0. It explains the motivation behind the protocol, describes its architecture, defines its economic model, and introduces the principles that guide its future evolution. Additional companion documents—including the Protocol Specification, Developer Handbook, Architecture Guide, and OpenFiat Improvement Proposal (OFIP) series—provide the detailed technical information necessary to build compatible implementations.

The intended audience includes cryptocurrency newcomers, merchants, developers, infrastructure operators, researchers, investors, auditors, and organizations evaluating OpenFiat for adoption. While no prior blockchain experience is assumed, the document aims to remain technically rigorous enough to serve as a reliable reference for future protocol development.

OpenFiat is not simply another cryptocurrency project. It is an attempt to define an open standard for decentralized peer-to-peer digital commerce—one that can be implemented, improved, and governed by the broader community for many years to come.
