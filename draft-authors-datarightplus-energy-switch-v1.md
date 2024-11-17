%%%
title = "DataRight+: Energy Account Switch V1"
area = "Internet"
workgroup = "datarightplus"
submissionType = "independent"

[seriesInfo]
name = "Internet-Draft"
value = "draft-authors-datarightplus-energy-switch-v1-latest"
stream = "independent"
status = "experimental"

date = 2024-05-02T00:00:00Z

[[author]]
initials="S."
surname="Low"
fullname="Stuart Low"
organization="Biza.io"
    [author.address]
    email = "stuart@biza.io"

[[author]]
initials="W."
surname="Cai"
fullname="Wei Cai"
organization="Biza.io"
    [author.address]
    email = "wcai@biza.io"

%%%

.# Abstract

This specification outlines the technical requirements related to functionality to enable an Energy Account switching to be initiated under the DataRight+ ecosystem framework.

Energy Account Switch V1 is intended to be an account opening specification for Energy sector participants. It incorporates initiation of the switch with the target Provider, tracking of the status of the switch by an Initiator and the optional sharing of energy data from the new Provider to the Initiator.

This specification is considered entirely new functionality currently outside the scope of the Australian Consumer Data Right.

.# Notational Conventions

The keywords  "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

The scope of this specification is focused on describing the authorisation related components of initiating an energy account switch to a new Provider, tracking the status of this switch and onward sharing to the Initiator of energy account data.

# Terms & Definitions

This specification uses the terms "Agreement Identifier", Consumer", "Provider", "Initiator", "Personally Identifiable Information (PII)",
"Pairwise Pseudonymous Identifier (PPID)", "Initiator", "CDR Sharing Arrangement" and "CDR Sharing Agreement Identifier"  as defined by [@!DATARIGHTPLUS-ROSETTA].

# High Level Process

This document, as extended further in OpenAPI format within [@!DATARIGHTPLUS-REDOCLY], describes the endpoints to deliver the Sharing Arrangement V2 capability. The current approach to data sharing within the Consumer Data Right is brittle and does not provide sufficient feedback as to the status of sharing requests as they are handed over from Initiator to the Provider. The existing sharing establishment process also assumes a live establishment rather than a potentially asynchronous, back-channel or machine to machine (IoT) authorisation approach.

At a high level the process expected to be followed through the implementation of this specification is:

1. Initiator utilises a client credentials grant to obtain a token from the Provider;
2. Initiator calls the Request Sharing Agreement endpoint and obtains an `actionId` in the initial status of `PENDING`;
3. Initiator submits a pushed authorisation request with a Request Object containing the `urn:dio:action_id` parameter with a value equal to the `actionId` returned in (2)
4. Initiator redirects the Consumer user-agent to the Provider authorisation server
5. The Provider authorisation server completes the Consumer onboarding process, using any associated pre-fill supplied to the Initiate Energy Switch endpoint
6. On completion of the authorisation process the Initiator performs `authorization_code` token exchange with the Provider authorisation server.

_Note 1:_ At any time before, during or after the authorisation process the Initiator can use the Get Sharing Agreement endpoint provided by the Resource Server utilising the obtained `actionId`.

_Note 2:_ It is at the discretion of the Provider what resources, if any, are available utilising the token obtained at the end of the authorisation process. In future versions fo this specification the token may represent an established sharing arrangement as specified in [@!DATARIGHTPLUS-SHARING-ARRANGEMENT-V2].

# Provider

The following provisions apply to services delivered by Providers.

## Authorisation Server

In addition to the provisions outlined in [@!DATARIGHTPLUS-INFOSEC-BASELINE] the authorisation server:

1. **SHALL** support the `dio:action` authorisation scope;
2. **SHALL** include the `dio:action` authorisation scope within Dynamic Client Registration responses;
3. **SHALL** support request objects containing an essential ID Token claim named `urn:dio:action_id` referencing a valid _Action Identifier_;
2. **SHALL** reject request objects containing a `urn:dio:action_id` claim that is unknown, expired or not associated with the requesting Initiator;

### Example

The following is a non-normative example of a decoded request object requesting authorisation for a previously lodged `actionId`

```json
{
  "iss": "33ff17d6-d967-4823-a4d3-1206fd2fff3f",
  "exp": 1725090872,
  "nbf": 1725090872,
  "aud": "https://www.provider.com.au",
  "response_type": "code",
  "response_mode": "jwt",
  "client_id": "33ff17d6-d967-4823-a4d3-1206fd2fff3f",
  "redirect_uri": "https://www.recipient.com.au/coolstuff",
  "scope": "openid",
  "claims": {
    "id_token": {
      "urn:dio:action_id": {
        "essential": true,
        "values": ["496a3ba7-04b4-4362-b775-9e0433e48eea"]
      },
      "acr": {
        "essential": true,
        "values": ["urn:cds.au:cdr:3"]
      }
    },
    "userinfo": {
      "given_name": null,
      "family_name": null
    }
  },
  "code_challenge": "rerbvXfTDYNECzwayM8-SLCWU1FDzBnqMCv1RB5AudU",
  "code_challenge_method": "S256"
}
```

### Claims

The authorisation server:

1. **SHALL** include within the Introspection Endpoint response the `urn:dio:action_id` claim, containing the assigned _Action Identifier_;

## Resource Server

The Provider Resource Server:
1. **SHALL** support the `initiateEnergySwitch`, `getEnergySwitchRequest` and `getEnergySwitchStatus` endpoints as described in [@!DATARIGHTPLUS-REDOCLY];
2. **SHALL** support [@!DATARIGHTPLUS-DISCOVERY-V1] and advertise the `initiateEnergySwitch`, `getEnergySwitchRequest` and `getEnergySwitchStatus` endpoints

# Initiator

The following provisions apply to participants operating Initiators.

## Authorisation Client

In addition to the provisions outlined in Section 4 of [@!DATARIGHTPLUS-INFOSEC-BASELINE] the Initiator authorisation client:

1. **SHALL** support the Initiator provisions of [@!DATARIGHTPLUS-DISCOVERY-V1] to discover the `initiateEnergySwitch`, `getEnergySwitchRequest` and `getEnergySwitchStatus` endpoints;
2. **SHALL** perform a Dynamic Client Registration update, as described in [@!DATARIGHTPLUS-ADMISSION-CONTROL-00], to be granted access to the `dio:action` scope;

# Implementation Considerations

This specification does not explicitly state how long an assigned Action Identifier persists for however it is expected that an Initiator shall be able to reference an Action Identifier while the associated authorisation is still active and for a significant period (more than a year) after it becomes inactive.

# Security Considerations

The Action Identifier **SHALL NOT** be guessable, derivable nor identify the Consumer.

{backmatter}

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ROSETTA" target="https://datarightplus.github.io/datarightplus-rosetta/draft-authors-datarightplus-rosetta.html"> <front><title>DataRight+ Rosetta Stone</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-REDOCLY" target="https://datarightplus.github.io/datarightplus-redocly/"> <front><title>DataRight+: Redocly (Draft)</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author>
<author initials="W." surname="Cai" fullname="Wei Cai"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-INFOSEC-BASELINE" target="https://datarightplus.github.io/datarightplus-infosec-baseline/draft-authors-datarightplus-infosec-baseline.html"> <front><title>DataRight+ Security Profile: Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ADMISSION-CONTROL-00" target="https://datarightplus.github.io/datarightplus-admission-control-baseline/draft-authors-datarightplus-admission-control-00/draft-authors-datarightplus-admission-control.html"> <front><title>DataRight+: Admission Control Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-SHARING-ARRANGEMENT-V2" target="https://datarightplus.github.io/datarightplus-sharing-arrangement-v2/draft-authors-datarightplus-sharing-arrangement-v2.html"> <front><title>DataRight+: Sharing Arrangement V2</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-DISCOVERY-V1" target="https://datarightplus.github.io/datarightplus-discovery/draft-authors-datarightplus-discovery.html"> <front><title>DataRight+: Provider Discovery V1</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="FAPI-GRANT-MANAGEMENT" target="https://openid.net/specs/fapi-grant-management.html"> <front> <title>The OAuth 2.0 Authorization Framework</title><author fullname="D. Hardt"> <organization>Microsoft</organization> </author><date month="Oct" year="2012"/></front> </reference>
