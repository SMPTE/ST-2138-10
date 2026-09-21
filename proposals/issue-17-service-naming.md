# Proposal for issue #17: two-label service names with RFC 6763 subtypes

Addresses [SMPTE/ST-2138-10#17](https://github.com/SMPTE/ST-2138-10/issues/17),
per maintainer request to prepare text against clauses 7.4 and 7.5.
This file is proposal text for committee consideration. It changes no PDF;
if accepted, the editors apply it to the draft source.

Conventions: `shall`/`shall not` normative, matching the draft. Every
replacement below quotes the current draft text it supersedes.

---

## 1. Clause 7.4.1 — replace the naming form and value tables

CURRENT (non-conformant, four labels):

> Each service instance shall have an instance name which shall conform to
> the following:
> `<instance-name>._st2138.<api>.<transport>.<network>.local`

PROPOSED:

> Each service instance shall have an instance name which shall conform to
> the following:
> `<instance-name>._st2138.<network>.local`
>
> Where:
> - `<instance-name>` is a human-readable identifier for the service instance
> - `_st2138` identifies the control protocol as SMPTE ST 2138
> - `<network>` specifies the transport protocol per RFC 6763 section 7,
>   which shall be one of these values:
>   - `_tcp`
>   - `_udp`, for instances using `quic` (see Table 2, `transport`)
>
> The two-label service type keeps every ST 2138 instance discoverable with
> stock DNS-SD tooling: a browse for `_st2138._tcp` enumerates all TCP-based
> instances and a browse for `_st2138._udp` all others, per RFC 6763
> section 7.
>
> API and transport narrowing, previously encoded as extra name labels, shall
> use RFC 6763 section 7.1 subtypes. An instance offering an API over TCP
> shall advertise the corresponding subtype:
>
> | API offered | subtype advertised |
> |---|---|
> | gRPC | `_grpc._sub._st2138._tcp` |
> | REST | `_rest._sub._st2138._tcp` |
> | WebSocket | `_ws._sub._st2138._tcp` |
>
> QUIC instances live under `_st2138._udp` and advertise no subtype; their
> API and transport are carried by the Table 2 keys alone. (If the
> committee wants filtered QUIC discovery, `_grpc._sub._st2138._udp` rows
> can be added here — left out deliberately, not overlooked.)
>
> A client discovering only a subset (for example, gRPC endpoints) browses
> the subtype; a client discovering everything browses `_st2138._tcp`. The
> advertised Service Instance Name is unchanged by the use of subtypes. TLS
> posture (`http2s` vs `http2`, `https` vs `http`) is expressed with the
> Table 2 `transport` key, not with name labels — which means a subtype
> browse alone cannot distinguish TLS variants; see the Table 2 pairing
> rule below.
>
> Naming Requirements (unchanged):
> - The instance name shall be unique on the local network.
> - Implementations may include the hostname or another unique identifier.

NOTE (non-normative, for the editors): the Service Name `st2138` is not
present in the IANA registry that RFC 6763 section 4.1.2 reaches via
RFC 6335. Registering it is a process action accompanying this change, not
a protocol behavior — stated here so it is not lost.

## 2. Clause 7.4.2 — replace the record examples

CURRENT PTR/SRV examples use the four-label form. PROPOSED (same router,
same port semantics, conformant names):

> - PTR Record
>
>   The PTR record allows clients to discover all available
>   `_st2138._tcp.local.` services.
>
>   For example, this record:
>
>   `_st2138._tcp.local. PTR router._st2138._tcp.local.`
>
>   Announces an instance called `router` that provides the ST 2138 service.
>   A client seeking only gRPC instances browses
>   `_grpc._sub._st2138._tcp.local.` instead.
>
> - SRV Record
>
>   The SRV record provides connection details for a specific instance:
>
>   For example:
>
>   `router._st2138._tcp.local. SRV 0 0 <port> <hostname>.local.`
>
>   Where `<port>` is the TCP or UDP port number on which the service is
>   available.
>
> - Subtype PTR records
>
>   For each API subtype the instance advertises under section 7.4.1, it
>   shall also publish a PTR record from the subtype name to the same
>   Service Instance Name, so filtered browsing resolves. For example:
>
>   `_grpc._sub._st2138._tcp.local. PTR router._st2138._tcp.local.`
>
>   Without these records the section 7.4.1 advertisement requirement has
>   no record backing it.
>
> - A/AAAA records address `<hostname>.local.`, not `<instance-name>.local.`:
>   the SRV target is the hostname, and the address records shall follow it.
>   (The draft's A/AAAA owner names use the instance name; corrected here
>   because the SRV record above points at the hostname.)

TXT record paragraph: replace
`<instance-name>._st2138.<api>.<transport>.<network>.local. TXT <key>=<value>`
with
`<instance-name>._st2138.<network>.local. TXT <key>=<value>`.

## 3. Table 2 — add `api` and `transport` keys

The subtype labels of section 7.4.1 need a single defined vocabulary, shared
with Table 3, so the name and the record cannot drift apart:

| Key | Required? | Description | Example |
|---|---|---|---|
| api | Yes | The APIs the instance offers, one or more of `grpc`, `rest`, `ws`, comma-separated when several. The set shall equal the set of subtypes advertised under section 7.4.1; QUIC instances, which advertise no subtypes, list the APIs offered. | `api=grpc,rest` |
| transport | Yes | The transports beneath the offered APIs, one or more of `http2s`, `http2`, `https`, `http`, `quic`, comma-separated and positionally aligned with `api` (first transport belongs to the first API, and so on). The permitted pairings are: gRPC with `http2s`, `http2` or `quic`; REST and WebSocket with `https` or `http`. | `transport=http2s,http` |

The positional rule is the whole point: without it `api=grpc,rest` with
`transport=http` cannot say which API the transport belongs to, and the
draft's own pairing (gRPC never on plain `http`, REST never on `http2s`)
would become expressible-but-wrong. No exceptions: a single transport
value with several APIs is not permitted — repeat it per API.

## 4. Clause 7.5 — replace the Avahi example

The example advertises one instance over two APIs; with subtypes it is one
instance, one type, two subtype attestations:

```xml
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
    <name replace-wildcards="yes">router</name>

    <!-- ST 2138, secure gRPC plus clear-text REST on one instance.
         Browsers see _st2138._tcp; filtered browsers see _grpc._sub or
         _rest._sub. Transports align positionally with api (7.4.1). -->
    <service>
        <type>_st2138._tcp</type>
        <subtype>_grpc._sub._st2138._tcp</subtype>
        <subtype>_rest._sub._st2138._tcp</subtype>
        <port>6254</port>
        <txt-record>authz=off</txt-record>
        <txt-record>interface=https://smpte.org/registry/st2138/service</txt-record>
        <txt-record>if_version=2025.1</txt-record>
        <txt-record>api=grpc,rest</txt-record>
        <txt-record>transport=http2s,http</txt-record>
        <txt-record>sdk=https://github.com/rossvideo/Catena</txt-record>
        <txt-record>sdk_version=cpp-v0.0.7-1</txt-record>
        <txt-record>content-type=application/json</txt-record>
    </service>

</service-group>
```

(Note: the current draft serves gRPC on 6254 and REST on 8080 under one
instance name, which DNS-SD cannot express — one SRV port per instance. The
replacement keeps a single service entry; if dual-port operation is
intended, that needs its own instance naming rule, which is out of scope
for this issue. `content-type=application/json` is restored: it was dropped
without a note and belongs to the REST leg.)

## 5. Table 3 — correct the value sets

- `api`: one or more of `grpc`, `rest`, `ws`, comma-separated, matching
  Table 2 case and vocabulary exactly (the draft's `gRPC, REST` display case
  would reintroduce the drift Table 2 exists to prevent). Update the 7.9
  example from `"api": "gRPC"` to `"api": "grpc"`.
- `transport`: correct `qui` to `quic`.

No other Table 3 change. `service-name`, `hostname`, `port`, address,
authorization, and metadata rows are untouched. 7.9 changes only in the
`api` value quoted above.

---

## Cross-checks performed (WO acceptance)

- RFC 6763 §4.1.2: proposed names are label-pairs (`_st2138` + `_tcp`/`_udp`).
- RFC 6763 §7.1: subtype form `_x._sub._st2138._tcp` matches the printer
  worked example; instance names unchanged by subtypes.
- RFC 6335: IANA registration noted as editor action, not mandated.
- Table 3 `api`/`transport` values are now a subset of the Table 2 key
  vocabularies, case included — no drift between name, TXT, registration
  payload, and the 7.9 example.
- Instance-name uniqueness rule and hostname guidance carried over verbatim.
- Touched: 7.4.1, 7.4.2 (including A/AAAA owner names), Table 2, 7.5
  (including its intro, which now describes one instance instead of two
  ports), Table 3, and the 7.9 `api` value. Nothing else.

## Verification appendix (primary sources, 2026-09-20)

- RFC 6763 §4.1.2 pair rule + §7.1 subtype mechanics verified verbatim
  against rfc-editor.org rfc6763.txt (not from memory).
- Avahi `<subtype>` (repeatable child of `<service>`) verified against the
  authoritative DTD (`avahi-daemon/avahi-service.dtd`,
  `<!ELEMENT service (type,subtype*,...)>`), so the 7.5 example's two
  subtype elements are schema-valid.
- Draft text extracted from the repo PDF (34 pp); every `_st2138._x._y`
  occurrence located — all live in 7.4.2/7.5, both covered. Table 3 `qui`
  typo and missing `ws` confirmed against extracted text.
- One correction made during audit: an early draft claimed resolvers
  consult the IANA registry; reworded to the process action it is.
