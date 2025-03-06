# Name of your Project

> [!NOTE]
> This document will be part of the terms and conditions of your agreement and, therefore, needs to contain all the required information about the project. Don't remove any of the mandatory parts presented in bold letters or as headlines (except for the title)! Lines starting with a `>` (such as this one) should be removed. Please use markdown instead of HTML (e.g., `![](image.png)` instead of `<img>`).
>
> See the [Grants Program Process](https://grants.web3.foundation/docs/process) on how to submit a proposal.

- **Team Name:** Ajuna Network
- **Payment Details:**
  - **DOT**: 16AjunUasoBZKWkDnHvNEALGUgGuzC92j7LJoLu9qBSUJB2e
  - **Payment**: 16AjunUasoBZKWkDnHvNEALGUgGuzC92j7LJoLu9qBSUJB2e (USDC).
- **[Level](https://grants.web3.foundation/docs/Introduction/levels):** 2

> [!IMPORTANT]
> *The combination of your GitHub account submitting the application and the payment address above will be your unique identifier during the program. Please keep them safe.*

## Project Overview :page_facing_up:

This application is a **follow-up grant** to the previous **Ajuna Network grants**, including work on the Substrate Unity SDK and gaming framework (referencing the previous Open Grants Program application).
- https://github.com/w3f/Grants-Program/pull/265
- https://github.com/w3f/Grants-Program/pull/433

**Project Name:** Substrate Asset Game Engine (SAGE)

**Tagline:** A modular, instanced Substrate framework for building blockchain-based games with reusable logic, asset handling, and developer-friendly tooling.

**Description:**
The **SAGE Game Framework** is a **modular and extensible game engine** designed to manage **on-chain game state** through an **asset-based, state-transition model**. Rather than maintaining complex game states in traditional off-chain databases or fragmented Substrate storage items, **SAGE embeds the game state directly within the assets themselves** — creating a self-contained, portable state system that lives entirely on-chain.

SAGE is designed to reduce the complexity of building blockchain games on Substrate by providing a flexible, reusable **state machine framework** with key features:
- Asset creation, mutation, trading, and nft wrapping (compatible with pallet-assets/NFT standards).
- Generic transitions for game state progression.
- Season, tournament, and leaderboard management.
- Support for modular gameplay logic, with **multiple entry points** for developers to plug in game-specific logic.
- Free to play mechanic, and Affiliation hooks for referral programs.
- Manager/admin roles to configure features dynamically.
- Account farming protection, with an achievement unlock feature system.
- Terminal UI and developer playground for quick hack & play.

SAGE evolved from the **Awesome Ajuna Avatars** project, that evolved from Bajun Network, into Big Ballz on **Ajuna Network** and available as a mobile game ([bbb.ajuna.io](https://bbb.ajuna.io)) ([Google Play](https://play.google.com/store/apps/details?id=io.ajuna.bbb)). After experiencing high complexity onboarding other projects onto AAA, we redesigned and modularized SAGE to be a **all-in one framework for future Substrate-based games**. Allowing gamedevs, to spin up a full enviroment, including a Unity template, to start develope games in seconds.

### Project Details

#### SAGE Game Framework

The SAGE Game Framework is a modular engine designed for game state management using an asset-based, state-transition model. Instead of using a traditional database, SAGE stores state directly on "assets" (e.g., game cards), and state changes are performed by executing transitions that consume and produce assets. This architecture allows for a flexible, decentralized, and extensible game design. Maintainging the full utility on the Asset it self, and allowing devs to not care for the on-chain storage, but work on the actual state transitions of the assets. SAGE Framework is not limited to games, features like assets holding balances, might open for other products, that can be created which we have to figure out. Game Devs, don't need to care for a specific and game cetric datamodel, assets can hold utilities far beyond, other benefits

##### Core Concepts

Each **asset** (such as a game character, item, or piece of equipment) is not just an object with a static set of properties; it carries its **current state**, such as its health, level, evolution status, or any other gameplay-relevant data. State changes (like upgrading an item, progressing a character, or resolving a battle) are processed through **state transitions** — pre-defined logical transformations that **consume and produce assets**.

- **Account:**
  Represents a player or system account. Each account is identified by a unique ID and holds a balance.

- **Asset:**
  The fundamental game object. Assets have an owner (`OwnerId`), a collection identifier, a score, a genesis timestamp, and a data array (often referred to as DNA) that encodes various properties using compact, bit-level operations.

- **Balance:**
  Tracks funds for both accounts and assets. Asset-specific balances are managed separately from overall account balances.

- **Transition:**
  A state change operation that is defined by:
  - A unique **Identifier** (encapsulated by an `ITransitionIdentifier`)
  - A set of **Rules** (implementing `ITransitionRule`)
  - An optional **Fee**
  - A **Transition Function** that computes new assets (the updated state) from the provided inputs

- **Engine:**
  The central component that manages accounts, assets, balances, and transitions. It relies on external providers (like a blockchain info provider) for block numbers and randomness.

##### Architecture Overview

![Sage Architecture WIP](https://raw.githubusercontent.com/ajuna-network/ajuna-pallets/refs/heads/develop/docs/sage-architecture-wip.svg)

The following Entity-Relationship (ER) diagram shows the relationships between the key entities:

![image](https://github.com/user-attachments/assets/1fc850c2-bdc7-4674-9030-02c316b0b14c)

```plantuml
@startuml ER Diagram
entity "Account" as Account {
  * Id : uint
  * Balance : uint
}

entity "Asset" as Asset {
  * Id : uint
  * OwnerId : uint
  * CollectionId : byte
  * Score : uint
  * Genesis : uint
  * Data : byte[]
}

entity "Balance" as Balance {
  * AssetId : uint
  * BalanceValue : uint
}

entity "Transition" as Transition {
  * Identifier : ITransitionIdentifier
  * Rules : ITransitionRule[]
  * Fee : ITransitioFee?
}

entity "Engine" as Engine {
  * BlockchainInfo : IBlockchainInfoProvider
}

Account ||--o{ Asset : "owns"
Asset ||--|{ Balance : "has"
Engine o-- Account : "manages"
Engine o-- Asset : "manages"
Engine o-- Transition : "registers"
@enduml
```

##### Transition Execution Flow

The following UML activity diagram illustrates the high-level flow of a state transition within the engine:

![image](https://github.com/user-attachments/assets/5827a96b-1ef5-49a0-87ba-fb766550aef8)

```plantuml
@startuml Activity Diagram for Transition Execution
start

:Account initiates transition;
:Call Engine.Transition(identifier, assets);

if (assets contain duplicates or lockable assets?) then (yes)
  :Abort transition;
  stop
else (no)
  :Verify rules via VerifyFunction;
  if (rules NOT satisfied?) then (yes)
    :Abort transition;
    stop
  else (no)
    :Check fee requirements;
    if (insufficient balance for fee?) then (yes)
      :Abort transition;
      stop
    else (no)
      :Withdraw fee from account balance;
      :Execute transition function;
      :Obtain output assets;
      :Apply state changes (create, update, delete assets);
      :Update balances accordingly;
    endif
  endif
endif

:Return success with output assets;
stop
@enduml
```

##### Summary

By separating accounts, assets, balances, and transitions, the SAGE Game Framework provides a robust foundation for developing complex, asset-based games. Its modular design and use of state transitions make it highly adaptable for various game genres and mechanics.


- Mockups/designs of any UI components
- Data models / API specifications of the core functionality
- An overview of the technology stack to be used
- Documentation of core components, protocols, architecture, etc. to be deployed
- PoC/MVP or other relevant prior work or research on the topic
- What your project is *not* or will *not* provide or implement
  - This is a place for you to manage expectations and clarify any limitations that might not be obvious


Things that shouldn’t be part of the application (see also our [FAQ](../docs/faq.md)):

- The (future) tokenomics of your project
- For non-infrastructure projects—deployment and hosting costs, maintenance or audits
- Business-oriented activities (marketing, business planning), events or outreach

### Ecosystem Fit

#### Where and how does your project fit into the ecosystem?

SAGE (State Asset Game Engine) fits directly into the **Polkadot/Substrate/Kusama ecosystem** by providing a **modular, asset-first framework** for building **fully on-chain games** and **game-like experiences**. It integrates seamlessly into the existing Substrate framework and the broader Polkadot application landscape, including the **Polkadot SDK for Unity**, creating an **easy and intuitive entry point for game developers** unfamiliar with blockchain development.

The **SAGE Playground repository** offers a **hack & play environment** that can be spun up locally in seconds, complete with **a Unity project template and a browser-based template**, allowing developers to experiment immediately without complex setup. This dramatically lowers the barrier to entry for developers interested in **on-chain gaming** on Polkadot.

#### Who is your target audience?

The primary target audiences are:

- **Parachains looking to offer gaming or asset-centric applications**.
- **Traditional game developers** (Unity, Unreal, browser-based devs) interested in integrating on-chain asset management and progression into their games.
- **dApp developers building applications where asset evolution, ownership, and composability are critical (beyond just gaming)**.

#### What need(s) does your project meet?

SAGE addresses a long-standing gap in the ecosystem: the **lack of accessible tooling for game developers** to create **on-chain games** without needing extensive Substrate, Rust, or smart contract expertise.

- **Simplified Onboarding**: By providing clear examples, templates, and ready-to-use assets, game developers can get started with **on-chain game development** without needing to understand **substrate pallets** or **complex storage handling**.
- **Asset-Centric Game Logic**: Instead of writing game logic directly in Rust or in smart contracts, developers define **how assets can transition** from state to state. This abstracts away much of the blockchain complexity.
- **Full Lifecycle Asset Management**: Developers can **track asset states, player progression, inventories**, and **in-game economies** via on-chain transitions, all without having to manually design low-level storage solutions.
- **Cross-Game Potential**: Assets created in one game can be **extended, evolved, or even reused across other games**, fostering interoperability within the ecosystem.

#### How did you identify these needs? Please provide evidence.

Our insights stem from **over three years of direct experience onboarding game developers into the Polkadot ecosystem**. Despite our own technical expertise, we encountered recurring challenges when working with **traditional game studios**. The primary obstacles included:

- **Understanding Substrate’s architecture**.
- **The need to write custom Rust code for every game mechanic**.
- **The disconnect between game design logic (in Unity/Unreal) and blockchain state management**.
- **The steep learning curve of blockchain-specific game logic (state transitions, storage, asset ownership, and evolution)**.

Multiple projects were delayed or abandoned because **the gap between traditional game development workflows and the on-chain asset lifecycle was too wide**. This feedback, combined with internal post-mortem reviews, highlighted the need for an **abstraction layer** that allows game developers to focus on **defining asset rules** instead of wrestling with blockchain complexity.

#### Are there any other projects similar to yours in the Substrate / Polkadot / Kusama ecosystem?

##### Similar Projects

The **Awesome Ajuna Avatars** project built on some of the **core SAGE principles**, demonstrating **on-chain asset evolution**, **player progression tied to asset states**, and **embedded game mechanics on-chain**. However, this project was **highly tailored to a specific game** and lacked the flexibility to **onboard external developers** easily. The onboarding process for new teams was complicated, often requiring **direct involvement from our core technical team**.

##### Key Differentiation

SAGE directly addresses these limitations by:

- Providing **a generalized, reusable asset transition framework** applicable across **different games and asset types**.
- Including **pre-built templates, Unity/Browser integration**, and **a simplified local dev environment (Playground)**.
- Focusing on **developer autonomy**, allowing **traditional game developers** to design **on-chain asset logic** without needing direct blockchain expertise.
- Emphasizing **composability and modularity**, enabling assets to **evolve across multiple games** or **applications**.

#### If not, why might such a project not have been attempted?

The lack of similar projects likely stems from:

- **High complexity in aligning game logic with blockchain state management**.
- **The relative immaturity of gaming-specific tooling in the Polkadot ecosystem**.
- **A focus on DeFi and infrastructure projects rather than user-facing game development frameworks**.
- **A lack of structured onboarding tools for game developers** who are unfamiliar with blockchain paradigms.

#### Are there any projects similar to yours in related ecosystems?

At this point, we are not aware of any **comparable projects in other blockchain ecosystems** that offer:

- A **fully on-chain game engine focused on asset state transitions**.
- Direct **Unity integration** with a focus on **traditional game developer workflows**.
- A **local playground environment for rapid prototyping**.

Some ecosystems (like Flow and Immutable X) have focused on **NFT-based game assets**, but these efforts typically **focus on asset minting/trading** rather than **full asset lifecycle management with built-in state progression and game logic integration**.

SAGE represents a **novel approach**, bridging **traditional game design concepts** with **on-chain asset management and game progression**, tailored specifically for Polkadot’s strengths (scalability, composability, and cross-chain compatibility).

## Team :busts_in_silhouette:

- Cedric Decoster (Solution Architect / Game Dev)
  A software engineer and solution architect for over 20 years and worked on enterprise applications in the Swiss banking sector. In his spare time, he has worked passionately on various blockchain and gaming projects for years. Cédric holds a bachelor’s degree in computer science. Lead Contributor of the SubstrateGaming Organisation and its flagship product the Substrate .NET API, and creative mind behind the products BattleMogs, Awesome Ajuna Avatars, Big Ballz of Bajun, DOT 4 Gravity, Hexalem (1st Polkadot Winter Hackathon ‘23), and more.
- Christian Langenbacher (Lead Developer, Rust)
  Major Contributor and Developer at Encointer and Integritee, building on the TEE stack, for a few years.
- Didac (Rust Developer)
- Niklas (Frontend & Rust Developer)

### Contact

- **Contact Name:** Cedric Decoster
- **Contact Email:** cedric.decoster@gmail.com
- **Website:** https://ajuna.io

### Legal Structure

- **Registered Address:** Stetten SH, Switzerland
- **Registered Legal Entity:** Bloga Tech AG

### Team's experience

The team consists of People that have been active in the Polkadot Developer Community for multiple Years, working in projects as Ajuna NEtwork, Encointer, Integritee, Polkadot Play, providing mutliple grants, but also infrastructure tool, like SDKs and APIs. We have PBA absolvent and Fellowship member, Parachain Founder.

### Team Code Repos

- https://github.com/ajuna-network
- https://github.com/SubstrateGaming
- https://github.com/PolkadotPlay

Please also provide the GitHub accounts of all team members. If they contain no activity, references to projects hosted elsewhere or live are also fine.

- https://github.com/darkfriend77
- https://github.com/clangenb
- https://github.com/DidacSF
- https://github.com/niklasp

### Team LinkedIn Profiles (if available)

- https://www.linkedin.com/in/cedric-decoster-3a004510b/
- https://www.linkedin.com/in/christian-langenbacher-baa629182
- https://www.linkedin.com/in/d%C3%ADdac-sement%C3%A9
- https://www.linkedin.com/{person_4}


## Development Status :open_book:

We already started with most of the work, and have been working on it for the past six months, obviously we had to see first if the concept works, and how it plays out. We are at the stage were we think it is a very usfull framework to be used to hack and play, with the playground to attrackt future game devs into the ecosystem, the repository will be moved to Polkadot PLay, or SubstrateGaming organisation, once we are ready.

- **Existing repo:**
  - (wip) C# Reference Implementation https://github.com/ajuna-network/Ajuna.SAGE
  - (wip) Playground: https://github.com/ajuna-network/sage-playground
  - (wip) Substrate: https://github.com/ajuna-network/ajuna-pallets/
    - (wip) https://github.com/ajuna-network/ajuna-pallets/tree/develop/sage
- **Documentation:**
  - (wip) https://wiki.ajuna.io/build-with-us/sage-substrate-asset-game-engine
- **Previous work:**
  - Precursor of SAGE: https://github.com/ajuna-network/ajuna-pallets/tree/develop/pallets/ajuna-awesome-avatars

## Development Roadmap :nut_and_bolt:

This section should break the development roadmap down into milestones and deliverables. To assist you in defining it, we have created a document with examples for some grant categories [here](../docs/Support%20Docs/grant_guidelines_per_category.md). Since these will be part of the agreement, it helps to describe *the functionality we should expect in as much detail as possible*, plus how we can verify and test that functionality. Whenever milestones are delivered, we refer to this document to ensure that everything has been delivered as expected.

Below we provide an **example roadmap**. In the descriptions, it should be clear how your project is related to Substrate, Kusama or Polkadot. We *recommend* that teams structure their roadmap as 1 milestone ≈ 1 month.

> [!CAUTION]
> If any of your deliverables are based on somebody else's work, make sure you work and publish *under the terms of the license* of the respective project and that you **highlight this fact in your milestone documentation** and in the source code if applicable! **Projects that submit other people's work without proper attribution will be immediately terminated.**

### Overview

- **Total Estimated Duration:** Duration of the whole project (e.g. 2 months)
- **Full-Time Equivalent (FTE):**  Average number of full-time employees working on the project throughout its duration (see [Wikipedia](https://en.wikipedia.org/wiki/Full-time_equivalent), e.g. 2 FTE)
- **Total Costs:** Requested amount in USD for the whole project (e.g. 12,000 USD). Note that the acceptance criteria and additional benefits vary depending on the [level](../README.md#level_slider-levels) of funding requested.
- **DOT %:** Percentage of Total Costs to be paid in (vested) DOT (≥ 50%)

### Milestone 1 Reference Implementation, Solution Architecture & Modularity

- **Estimated duration:** 1 month
- **FTE:**  2
- **Costs:** 25,000 USD

> [!NOTE]
> **The default deliverables 0a-0d below are mandatory for all milestones**, and deliverable 0e at least for the last one.

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 / GPLv3 / MIT / Unlicense. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#license) for details. |
| **0b.** | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can (for example) spin up one of our Substrate nodes and send test transactions, which will show how the new functionality works. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#documentation) for details. |
| **0c.** | Testing and Testing Guide | Core functions will be fully covered by comprehensive unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#testing-guide) for details. |
| **0d.** | Docker | We will provide a Dockerfile(s) that can be used to test all the functionality delivered with this milestone. |
| 0e. | Article | We will publish an **article**/workshop that explains [...] (what was done/achieved as part of the grant). (Content, language, and medium should reflect your target audience described above.) |
| 1. | Reference Implementation: Asset Transition Engine | Initial implementation of a reference transition engine, demonstrating minting, mutation, and asset validation in a modular manner, decoupled from any specific game logic. |
| 2. | Feature Evaluation Report | A comprehensive evaluation of features required for a generic Substrate game engine, comparing different approaches from existing pallets (e.g., pallet-assets, pallet-uniques) and explaining which features will be included, extended, or excluded in SAGE. |
| 3. | Modularity Design | Full design documentation covering how individual components (assets, transitions, seasons, roles, etc.) will be isolated into extensible modules, with clear extension points for custom game logic. |
| 4. | Transition Logic PoC | 	Initial implementation of generic transition logic, demonstrating how asset minting, mutation, and seasonal changes can be triggered and validated independently of game-specific rules. |
| 5. | API Design | Design and initial documentation for the SageApi, which will expose critical storage access and management functions for developers to plug into. |
| 6. | Developer Playground Setup | Basic setup for a developer playground, where developers can simulate asset transitions and test core SAGE concepts via CLI or simple web UI. |

### Milestone 2 Rust Implementation & Core Framework Development

- **Estimated Duration:** 1 month
- **FTE:**  2
- **Costs:** 25,000 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 / GPLv3 / MIT / Unlicense. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#license) for details. |
| **0b.** | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can (for example) spin up one of our Substrate nodes and send test transactions, which will show how the new functionality works. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#documentation) for details. |
| **0c.** | Testing and Testing Guide | Core functions will be fully covered by comprehensive unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#testing-guide) for details. |
| **0d.** | Docker | We will provide a Dockerfile(s) that can be used to test all the functionality delivered with this milestone. |
| 0e. | Article | We will publish an **article**/workshop that explains [...] (what was done/achieved as part of the grant). (Content, language, and medium should reflect your target audience described above.) |
| 1. | Substrate Module: Core SAGE Pallet | Implementation of the pallet-sage, including asset management, asset transitions (mint/mutate/trade), seasons, and role-based management. |
| 2. | Substrate Module: Transition Handler | Implementation of a transition handler module, capable of processing modular transition logic (pluggable by game developers). |
| 3. | Substrate Module: SageAPI Hooks | Implementation of the SageApi, exposing player configuration, asset lookup, seasonal data, and global game configuration methods. |
| 4. | Storage Schema | Full storage schema, defining how assets, players, seasons, and transitions are stored and queried efficiently within the Substrate runtime. |
| 5. | Generic Transition Library | Reusable library of common transitions (minting, merging, mutating, splitting), easily extensible for game-specific logic. |
| 6. | Runtime Integration Example | Example Substrate runtime demonstrating how to include pallet-sage alongside other game-related pallets, with configuration examples. |
| 7. | Playground Runtime | Deployment-ready playground runtime for testing SAGE concepts locally, preloaded with assets, seasons, and sample transitions. |

### Milestone 3 UI, JavaScript SDK, Unity Integration & Developer Playground

- **Estimated Duration:** 1 month
- **FTE:**  1,5
- **Costs:** 15'000 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 / GPLv3 / MIT / Unlicense. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#license) for details. |
| **0b.** | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can (for example) spin up one of our Substrate nodes and send test transactions, which will show how the new functionality works. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#documentation) for details. |
| **0c.** | Testing and Testing Guide | Core functions will be fully covered by comprehensive unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. See the [delivery guidelines](https://grants.web3.foundation/docs/Support%20Docs/milestone-deliverables-guidelines#testing-guide) for details. |
| **0d.** | Docker | We will provide a Dockerfile(s) that can be used to test all the functionality delivered with this milestone. |
| 0e. | Article | We will publish an **article**/workshop that explains [...] (what was done/achieved as part of the grant). (Content, language, and medium should reflect your target audience described above.) |
| 1. | JavaScript SDK | Creation of a JavaScript SDK that wraps calls to SageAPI, providing easy-to-use functions for asset lookup, transitions, and player management. |
| 2. | Unity Integration | Sample Unity project demonstrating wallet creation, asset minting, mutation, and trading, using the JavaScript SDK to interact with the Substrate node. |
| 3. | Developer Playground UI | Web-based developer playground where developers can visually create assets, trigger transitions, and inspect game state — backed by SageAPI and running on a local Substrate node. |
| 4. | Playground Backend | Node.js backend for proxying Substrate RPC calls to SageAPI, with simplified endpoints for asset management and transitions (developer-friendly). |
| 5. | CLI Tool | Simple CLI tool to simulate transitions and query asset state directly from the terminal, using the JavaScript SDK. |
| 6. | Sample Game Template | Basic game template (example: breeding game) demonstrating how developers can build fully on-chain gameplay using SAGE transitions. |
| 7. | Workshop / Demo Video | Full recorded demo showing how to set up a Substrate node with SAGE, deploy assets, run transitions, and connect both Unity and web UIs to interact with the game state. |

## Future Plans

Please include here

- how you intend to finance the project's long-term maintenance and development,
- how you intend to use, enhance, and promote your project in the short term, and
- the team's long-term plans and intentions in relation to it.

## Referral Program (optional) :moneybag:

You can find more information about the program [here](https://grants.web3.foundation/docs/referral-program).

- **Referrer:** Name of the Polkadot Ambassador or GitHub account of the Web3 Foundation grantee
- **Payment Address:** Polkadot/Kusama (USDC) payment address. Please also specify the currency. (e.g. 15oF4... (USDC))

## Additional Information :heavy_plus_sign:

**How did you hear about the Grants Program?** Web3 Foundation Website / Medium / Twitter / Element / Announcement by another team / personal recommendation / etc.

Here you can also add any additional information that you think is relevant to this application but isn't part of it already, such as:

- Work you have already done.
- If there are any other teams who have already contributed (financially) to the project.
- Previous grants you may have applied for.
