---
title: "Web bot auth Glossary"
abbrev: "Web bot auth Glossary"
category: info

docname: draft-meunier-web-bot-auth-glossary-somehow-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - bot authentication
 - web security
 - glossary
 - automated agents
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "thibmeu/draft-meunier-glossary-somehow"
  latest: "https://thibmeu.github.io/draft-meunier-glossary-somehow/draft-meunier-web-bot-auth-glossary-somehow.html"

author:
 -
    fullname: Thibault Meunier
    organization: Cloudflare
    email: ot-ietf@thibault.uk

normative:
  RFC6973:
  RFC8890:
  RFC9518:

# I use names instead of quoting RFC because it's more readable for me at the moment
informative:
  A2A-AUTH:
    title: A2A protocol Authentication
    target: https://google.github.io/A2A/#/documentation?id=authentication-and-authorization
  A2A-DISCOVERY:
    title: A2A protocol Agent discovery
    target: https://google.github.io/A2A/#/topics/agent_discovery
  CERTIFICATE-TRANSPARENCY-RFC: RFC6962
  CONCEALED-AUTH-RFC: RFC9729
  DPOP-AUTH-RFC: RFC9449
  HTTP-AUTHSCHEME:
    title: IANA HTTP Authentication Scheme Registry
    target: https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml
  HTTP-MESSAGE-SIGNATURE-FOR-BOTS: I-D.draft-meunier-web-bot-auth-architecture
  KEY-TRANSPARENCY-ARCHITECTURE: I-D.draft-ietf-keytrans-architecture
  LOX:
    title: "Lox: Protecting the Social Graph in Bridge Distribution"
    target: https://petsymposium.org/2023/files/papers/issue1/popets-2023-0029.pdf
  MCP-AUTH:
    title: Model Context Protocol Authorization
    target: https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization
  MCP-DISCOVERY:
    title: Model Context Protocol Registry
    target: https://modelcontextprotocol.io/development/roadmap#registry
  OAUTH2-RFC: RFC6749
  OAUTH-BEARER-RFC: RFC6750
  OPENAPI3-AUTH:
    title: OpenAPI 3.0 Authentication
    target: https://swagger.io/docs/specification/v3_0/authentication/
  PRIVACYPASS-HTTP-AUTH-RFC: RFC9577
  PRIVACYPASS-IN-TLS: I-D.draft-pauly-privacypass-for-tls
  PRIVACY-PASS-KAGI:
    title: Introducing Privacy Pass authentication for Kagi Search
    target: https://blog.kagi.com/kagi-privacy-pass
  PRIVATE-ACCESS-TOKEN:
    title: "Challenge: Private Access Tokens"
    target: https://developer.apple.com/news/?id=huqjyh7k
  PRIVATE-PROOF-API:
    title: Explainer by Googlers Private Proof API
    target: https://explainers-by-googlers.github.io/private-proof/
  PRIVATE-STATE-TOKEN:
    title: W3C Private State Token API
    target: https://wicg.github.io/trust-token-api/
  REQ-MTLS: I-D.draft-jhoyla-req-mtls-flag
  VERIFIABLE-CREDENTIALS:
    title: "Verifiable Credentials Data Model v1.1"
    target: https://www.w3.org/TR/2022/REC-vc-data-model-20220303/


--- abstract

Automated traffic authentication presents unique security challenges,
constraints, and opportunities that impact all Internet users. This document
seeks to collect terminology and examples within the space, with a specific
focus on AI related technologies.


--- middle

# Introduction

Agents are increasingly used in business and user workflows, including AI
assistants, search indexing, content aggregation, and automated testing. These
agents need to reliably identify themselves to origins for several reasons:

1. Regulatory compliance requiring transparency of automated systems
2. Origin resource management and access control
3. Protection against impersonation and reputation management
4. Service level differentiation between human and automated traffic

Current identification methods such as IP allowlisting, User-Agent strings, or
shared API keys have significant limitations in security, scalability,
manageability, and fairness. This document presents these examples, as well as
possible paths to address them.

# Motivation {#motivation}

There is an increase in agent traffic on the Internet. Many agents
choose to identify their traffic today via IP Address lists and/or unique
User-Agents. This is often done to demonstrate trust and safety claims, support
allowlisting/denylisting the traffic in a granular manor, and enable sites to
monitor and rate limit per agent operator. However, these mechanisms have drawbacks:

 1. User-Agent, when used alone, can be spoofed meaning anyone may attempt to
    act as that agent. It is also overloaded - an agent may be using Chromium and
    wish to present itself as such to ensure rendering works, yet it still wants to
    differentiate its traffic to the site.
 2. IP blocks alone can present a confusing story. IPs on cloud plaforms have
    layers of ownership - the platform owns the IP and registers it in their
    published IP blocks, only to be re-published by the agent with little to bind
    the publication to the actual service provider that may be renting infra. Purchasing
    dedicated IP blocks is expensive, time consuming, and requires significant
    specialist knowledge to set up. These IP blocks may have prior reputation
    history that needs to be carefully inspected and managed before purchase and
    use.
 3. An agent may go to every website on the Internet and share a secret with
    them like a Bearer from {{OAUTH-BEARER-RFC}}. This is impractical to scale for any
    agent beyond select partnerships, and insecure, as key rotation is challenging
    and becomes less secure as the consumers scale.

Using well-established cryptography, we can instead define a simple and secure
mechanism that empowers small and large agents to share their identity.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

**Agent**
: autonomous entities that perceive the environment and can take actions on behalf of users.

**Bot**
: A type of agent that operates automatically, often performing repetitive tasks. Bots may identify themselves or attempt to mimic human behavior.

**Origin (Server)**
: The primary server hosting the web content or service that an agent intends to access.

**Application Firewall**
: control incoming traffic to an origin based on a set of rules. This may include but is not limited to IP filtering, User-Agent match, or cryptographic signature verification.

**Reverse proxy**
: An intermediary server that forwards client requests to the origin server, often performing functions like load balancing, authentication, or caching.

**Browser**
: A client application used to access web content. Browsers may also be orchestrated.

**Human**
: A physical person, like you and me.

**Rate limit**
: A control mechanism that restricts access of an Agent to a resource provided
  by an Origin Server. An Origin can decide to rate limit all connections from
  an individual Client, from a specific Provider, or to a specific resource.
  This may be a fixed number of request, a budget, a time, location, legal.

**Unlinkability**
: A property ensuring that multiple interactions or credentials from the same
  agent cannot be correlated by the verifier.

**Account**
: Persistent identifier of an agent to an origin. This requires a registration.

**Registration**
: The creation of an identity. It can involve one time payment, a subscription,
  an account with user name/password, an age, a legal jurisdiction, others.

# should we quote privacy pass arch here, or use VC model with a Holder?
**Issuer**:
  An entity that generates and provides credentials to agents after the Attester
  has verified certain attributes.

**Attester**:
  An entity that evaluates an agent's characteristics or behavior and provides
  evidence to an Issuer to support credential issuance.

**Verifier**:
  An entity that validates the authenticity and integrity of a credential
  presented by an agent.

# Web bot authentication categories

We divide web bot authentication in three categories.

## Identifying providers {#use-case-company}

Organizations operating bots may need to authenticate their agents to access
certain web resources. Authentication mechanisms can help distinguish legitimate
bots from malicious ones.

Examples:

- Web crawlers wanting to authenticate against origins such as search engines,
- Security companies that want to perform scans to identify malicious URLs,
- AI augmented queries that are looking to identify against a set of newspapers.

## User Account Identification {#use-case-known-account}

Bots acting on behalf of registered users may require authentication to access
user-specific data or services.

Examples:

- Authenticating and authorizing a known user against a particular resources,
  such as a newspaper they have a subscription for,
- Most authorisation use cases for {{MCP-AUTH}} and {{A2A-AUTH}}.


## Attribute-Based Access {#use-case-selective-disclosure}

In scenarios where full identification is unnecessary or undesirable, agents may
present credentials that attest to specific attributes without revealing their
identity.

Examples:

- Add a signal to limit visual CAPTCHA challenge such as {{PRIVATE-ACCESS-TOKEN}},
- Gating access to a resource for longstanding users such as {{LOX}},
- Search engine with a fixed number of request such as {{PRIVACY-PASS-KAGI}},
- Selective disclosure of a credential attribute (location, age) such as
  {{PRIVATE-PROOF-API}}.


# Architecture

The architecture involves multiple actors: a credential issuer that requires an
certain criteria to be passed via an attester, the client which an be a bot or
human-mediated agent which IP is not known, and the web origin placed behind a
reverse proxy that may be fronting its infrastructure. The issuer provides
cryptographic credentials to the client, which are then attached to requests and
optionally verified by proxies before reaching the origin. This chain allows for
authentication without necessarily revealing identifying details to each
intermediate.

## Overview

~~~aasvg
                     +---------------+             +----------+
                .--->| Reverse Proxy |------------>|  Origin  |
               /     +---------------+             +-----+----+
              /                                          ^
             /                                           ║
+----------+/                                            ║
|  Client  +                                           Trust
+----------+\                                            ║
             \                                           ║
              \                                          v
               \     +----------+                  +-----+----+
                '--->| Attester |----------------->|  Issuer  |
                     +----------+                  +----------+
~~~

## AI agent use example

Humans and bots often interact with origins indirectly via clients like
browsers, agents, or CLI tools. These clients handle requests, potentially
traversing reverse proxies that manage TLS termination, DDoS protection, and
caching.

The rise of advanced browser orchestration blurs lines between human-driven and
automated requests, making reliable bot identification increasingly complex.

~~~aasvg
                      .-------------------------------------.
                      |    Company           +---------+    |
                      |                      | Bot     +--------------.
                      |                      +---------+    |         |
                      |  +---------+                        |         |
                      |  |         +----------------------------------+
               .-------->| Agent   |         +---------+    |         |   +---------------+        +--------+
+-------+      |      |  |         +-------->| Browser +--------------+-->| Reverse Proxy +------->| Origin |
| Human +------+      |  +---------+         +---------+    |         |   +---------------+        +--------+
+-------+      |      |                                     |         |
               |      '-------------------------------------'         |
               |                                                      |
               |                             +---------+              |
               '---------------------------->| Browser |--------------|
                                             +---------+              |
                                                                      |
                                             +---------+              |
                                             | Bot     +--------------'
                                             +---------+
~~~

The attester/issuer roles could be filled by the AI company, reverse proxy,
origin, or a third party. Origins need mechanisms to identify organizations,
rate-limit individuals, and authenticate users without relying solely on client
IP or heuristics presented in {{motivation}}.


# Security goals and threat model

The security model includes several actors: credential issuers, attesters,
clients (bots or agents), reverse proxies, and origin servers. The primary goals
are to prevent impersonation, allow for credential revocation, support
delegation and rotation, and maintain trust boundaries.


## Public vs private presentation

If the Issuer is also the Origin or its reverse proxy, it is possible to use
shared secrets for verification. In cases where the issuer and verifier are
different entities, asymmetric cryptography becomes necessary, allowing the bot
to prove authenticity using a public key infrastructure.

## Single vs multi show {#single-vs-multi-show}

Some credentials may be designed for one-time use only (for anti replay or
privacy reasons), while others can support multiple presentations through the
use of cryptographic derivation techniques.
This distinction affects privacy, scalability, and implementation complexity.

## Transport

Authentication tokens may be exchanged at different protocol layers and through
different transports.
Each may have different deployment, performance, and security guarantees.

For TLS, we have seen {{REQ-MTLS}} and {{PRIVACYPASS-IN-TLS}} respectively
addressing {{use-case-company}} and {{use-case-selective-disclosure}}.

For HTTP, we see {{HTTP-MESSAGE-SIGNATURE-FOR-BOTS}} or {{DPOP-AUTH-RFC}}, and
{{PRIVACYPASS-HTTP-AUTH-RFC}} respectively addressing {{use-case-company}} and
{{use-case-selective-disclosure}}. {{OAUTH-BEARER-RFC}} fits as well for
{{use-case-known-account}}.

Other methods have been seen such as leveraging a dedicated format on top of a
JavaScript API. This is the case for W3C {{PRIVATE-STATE-TOKEN}} or the more
recent {{PRIVATE-PROOF-API}}.

Focusing on AI specifically, it's worth mentioning two proponent protocol
definition efforts:

- {{A2A-AUTH}} which follows {{OPENAPI3-AUTH}}. This means it allows for Basic,
  Bearer, API Keys, and {{OAUTH2-RFC}}. OpenAPI mentions using {{HTTP-AUTHSCHEME}}
  registry, but there does not seem to be a definition for recent schemes such as
  {{PRIVACYPASS-HTTP-AUTH-RFC}}, {{CONCEALED-AUTH-RFC}}, or {{DPOP-AUTH-RFC}}.
- {{MCP-AUTH}} uses {{OAUTH2-RFC}} as a resource server.


## Round trip

Protocols should thrive to minimise the number of round trips between a client
and the issuer, and between clients and the origin.

# Key management and discovery {#key-management}

## Catalog

Like there are registries to resolve IP address metadata, there are going to be
registries to identify the owner of public key material.
These are mentioned by {{A2A-DISCOVERY}} and {{MCP-DISCOVERY}}.

The primary goal of these catalog is to associate metadata to public key, and to
discover them. They SHOULD have some sort of tamper resistent, to prevent the
provider of a catalog to provide from information.

As an analogy, one can think of {{CERTIFICATE-TRANSPARENCY-RFC}}, or the more
recent effort in {{KEY-TRANSPARENCY-ARCHITECTURE}}.

## Submission / out-of-band

Sumbission is also going to happen out of band. This is both for a practical
reason -- it is simpler than setting up a catalog --, and for privacy reason
given you don't have to expose information through a catalog.

## In-flight

Discovery may happen in-flight, that is when a request arrives from a client to
an origin.
This could be considered a trust on first use. While the level of trust is low,
it could be viable for certain use cases.

Such discovery could be via an HTTP header containing a domain name with a
well-known, a URL, a certificate, others.

## Format

There is a multitude of Key and directory format. This include and is not
limited to JWKS, CWKS, Privacy Pass, Agent Card, or HTTP Message Signatures.


# Security Considerations

This glossary provides terminology for web bot authentication. While this
document does not define or recommend specific protocols, terminology choices
have direct security implications:

**Impersonation Resistance**
: Clearly defined roles are essential for preventing entities from falsely
  claiming identities.

**Credential Replay and Theft**
: Definitions such as {{single-vs-multi-show}} help describe key mechanisms that
  mitigate the misuse of credentials if stolen.

**Key Management**
: {{key-management}} is key to protocol security, and has to be considered.

In addition, protocols should consider decentralization {{RFC9518}} and end-user
impact {{RFC8890}}.


# Privacy Considerations

Authentication mechanisms should minimize the collection and exposure of
personal data. Techniques like selective disclosure and unlinkability help
protect user privacy. Protocols should refer to {{RFC6973}}.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
