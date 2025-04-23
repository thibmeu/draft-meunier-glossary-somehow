---
title: "Web bot auth Glossary"
abbrev: "Agent Glossary"
category: info

docname: draft-meunier-web-bot-auth-glossary-somehow-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - not-yet
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: thibmeu/draft-meunier-glossary-somehow
  latest: "https://thibmeu.github.io/draft-meunier-glossary-somehow/draft-meunier-web-bot-auth-glossary-somehow.html"

author:
 -
    fullname: Thibault Meunier
    organization: Cloudflare
    email: ot-ietf@thibault.uk

normative:
  RFC6973:

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
  PRIVATE-PROOF-API:
    title: Explainer by Googlers Private Proof API
    target: https://explainers-by-googlers.github.io/private-proof/
  PRIVATE-STATE-TOKEN:
    title: W3C Private State Token API
    target: https://wicg.github.io/trust-token-api/
  REQ-MTLS: I-D.draft-jhoyla-req-mtls-flag


--- abstract

Automated traffic authentication provides a unique set of security challenges, constraints, and possibilities
that affect every user of the Internet. This document seeks to collect use cases within the space,
with a specific focus on AI related technologies.


--- middle

# Introduction

> Thibault: this is taken from http message signature draft

Agents are increasingly used in business and user workflows, including AI assistants,
search indexing, content aggregation, and automated testing. These agents need to reliably identify
themselves to origins for several reasons:

1. Regulatory compliance requiring transparency of automated systems
2. Origin resource management and access control
3. Protection against impersonation and reputation management
4. Service level differentiation between human and automated traffic

Current identification methods such as IP allowlisting, User-Agent strings, or shared API keys have
significant limitations in security, scalability, manageability, and fairness. This document
presents these use cases, as well as possible paths to address them.

# Motivation

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
    them like a Bearer from  {{OAUTH-BEARER-RFC}}. This is impractical to scale for any
    agent beyond select partnerships, and insecure, as key rotation is challenging
    and becomes less secure as the consumers scale.

Using well-established cryptography, we can instead define a simple and secure
mechanism that empowers small and large agents to share their identity.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Agent
: autonomous entities that perceive the environment and can take actions on behalf of users.

Origin
: Host resources, which are of interest to agents.

Bot
: same as agent, but not hiding their identity (web crawling, ddos) and usually larger scale. Think Google web crawler, Facebook link shortner, Microsoft email scanner.

Application Firewall
: control incoming traffic to an origin based on a set of rules. This may include but is not limited to IP filtering, User-Agent match, or cryptographic signature verification.

Forward proxy
: Presents request on behalf of users. I include the act of understanding a users request from a prompt and making all necessary requests in a different format. Agents are forward proxy for eyeballs, Cloudflare MASQUE relay is a forward proxy for agents. Think Private Relay, Workers, OHTTP.

Eyeball
: ultimate beneficiary of the content.

Browser
: Software acting as a gateway to the web. These are agents with a lot more human gating for actions. Examples are Chrome desktop, Firefox mobile, Safari webview.

Human
: a physical person, like you and me.

Rate limit
: limiting access to a resource. An origin can decide to rate limit all connections from an individual agent, from a specific operator, or to a specific resource. This may be a fixed number of request, a budget, a time, location, legal.

Unlinkability
: Cryptographic guarantee making two requests undistinguishable from one another.

Account
: Persistent identifier of a client to an origin. This requires a registration phase.

Registration
: creation of an identity. That identity may be private, with anonymous credentials for instance. It can involve one time payment, subscription, an account with user name/password, an age, a legal jurisdiction, others

# Architecture

The architecture involves multiple actors: a credential issuer that requires an c ertain criteria to be passed via an attester, the client which an be a bot or human-mediated agent which IP is not known, and the web origin placed behind a reverse proxy that may be fronting its infrastructure. The issuer provides cryptographic credentials to the client, which are then attached to requests and optionally verified by proxies before reaching the origin. This chain allows for authentication without necessarily revealing identifying details to each intermediate.

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

## Use cases

We divide the use cases in three categories.

### Identifying a company {#use-case-company}

This is required by origins to know where the traffic is coming from
This is also a demand of honest companies that want to identify themselves but have a hard time doing it at the moment.

You can think of web crawlers wanting to authenticate against origins, security companies that want to perform scans, or AI crawlers that are looking to identify against a set of partners.

### Identification of a known account {#use-case-known-account}

An individual should be able to identify against a provider it has an existing relationship with in the form of an account.
This can be because the individual has a subscription, or others.

### Ability to rate limit/gate access based on selective disclosure {#use-case-selective-disclosure}

Not all use case require an account, or can be solely done by a company.
Origins may want to rate limit individuals, without identifying them.
Service providers would like to ensure that their users are not abusive without necessarily coordinating with each and every origins or insting client traffics.
In addition, origins may want to know a limited set of informations about a user such as thenir location, which they used to infer by deriving it from others, like IP, ASN, timing, browser fingerprint.

## Security goals and threat model

The security model includes several actors: credential issuers, attesters, clients (bots or agents), reverse proxies, and origin servers. The primary goals are to prevent impersonation, allow for credential revocation, support delegation and rotation, and maintain trust boundaries. Mechanisms must be resilient to replay attacks and Sybil attacks, and should ensure that credential issuance is auditable and accountable.

> TODO: consider adding deployment models


## Public vs private presentation

If the Issuer is also the Origin or its reverse proxy, it is possible to use shared secrets for verification. In cases where the issuer and verifier are different entities, asymmetric cryptography becomes necessary, allowing the bot to prove authenticity using a public key infrastructure.

## Single vs multi show

Some credentials may be designed for one-time use only (for anti replay or privacy reasons), while others can support multiple presentations through the use of cryptographic derivation techniques.
This distinction affects privacy, scalability, and implementation complexity.

## Transport

Authentication tokens may be exchanged at different protocol layers and through different transports.
Each may have different deployment, performance, and security guarantees.

For TLS, we have seen {{REQ-MTLS}} and {{PRIVACYPASS-IN-TLS}} respectively addressing {{use-case-company}} and {{use-case-selective-disclosure}}.

For HTTP, we see {{HTTP-MESSAGE-SIGNATURE-FOR-BOTS}} and PRIVACYPASS-HTTP-AUTH-RFC respectively addressing {{use-case-company}} and {{use-case-selective-disclosure}}. {{OAUTH-BEARER-RFC}} fits as well for {{use-case-known-account}}.

Other methods have been seen such as leveraging a dedicated format on top of a JavaScript API. This is the case for W3C {{PRIVATE-STATE-TOKEN}} or the more recent {{PRIVATE-PROOF-API}}.

Focusing on AI specifically, it's worth mentioning two proponent protocol definition efforts:

- {{A2A-AUTH}} which follows {{OPENAPI3-AUTH}}. This means it allows for Basic, Bearer, API Keys, and {{OAUTH2-RFC}}. OpenAPI mentions using {{HTTP-AUTHSCHEME}} registry, but there does not seem to be a definition for recent schemes such as {{PRIVACYPASS-HTTP-AUTH-RFC}}, CONCEALED-AUTH-RFC, or {{DPOP-AUTH-RFC}}.
- {{MCP-AUTH}} uses {{OAUTH2-RFC}} as a resource server, and does not support WWW-Authenticate headers. This is currently being debated on GitHub.


## Round trip

We thrive to minimise the number of round trips. Even though a nonce may be provided, it add delays and compute

# Key management and discovery

## Catalog

Like there are registries to resolve IP address metadata, there are going to be registries to
identify the owner of public key material.
These are mentioned by {{A2A-DISCOVERY}} and {{MCP-DISCOVERY}}.

The primary goal of these catalog is to associate metadata to public key, and to discover them. They SHOULD have some sort of tamper resistent, to prevent the provider of a catalog to provide from information.

As an analogy, one can think of {{CERTIFICATE-TRANSPARENCY-RFC}}, or the more recent effort in {{KEY-TRANSPARENCY-ARCHITECTURE}}.

## Submission / out-of-band

Sumbission is also going to happen out of band. This is both for a practical reason that it's simpler than setting up a catalog, or for privacy reason.

## In-flight

Discovery may happen in-flight, that is when a request arrives from a client to an origin.
This could be considered a trust on first use. While the level of trust is low, it could be viable for certain use cases.
Such discovery could be via an HTTP header containing a domain name with a well-known, a URL, a certificate, others.

## Format

There is a multitude of Key and directory format. This include and is not limited to JWKS, CWKS, Privacy Pass, Agent Card, or HTTP Message Signatures.


# Security Considerations

TODO Security

Quote {{!RFC9518}} and {{!RFC8890}}


# Privacy Considerations

See {{RFC6973}}.

> TODO: anonymity set, correlation, it depends on the issuer and origin. origins SHOULD thrive to ask the minimum that's required


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
