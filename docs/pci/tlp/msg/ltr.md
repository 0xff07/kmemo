---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# LTR (Latency Tolerance Reporting)

LTR is a 4DW Msg (no data) routed To Root Complex. It is emitted by an Endpoint to inform the platform of its maximum tolerable service latency. The Message Code is `0x10` and the Type encodes `10000` (To RC). Unlike most other Messages, LTR's data lives in `DW 2` rather than as a payload, so the TLP is Msg rather than MsgD.

```
LTR (Latency Tolerance Reporting)  (Routing: To RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00010000    │
           ├───────────────────────────────┼───────────────┴────────────────┤
DW 2       │    No-Snoop Latency [15:0]    │      Snoop Latency [15:0]      │
           ├───────────────────────────────┴────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

LTR lets an Endpoint tell the platform how long it can tolerate before its memory accesses are serviced. The platform aggregates LTR values across all Endpoints and uses the worst-case requirement to decide how deep an idle state the CPU and memory subsystem may enter (deeper idle implies longer wake-up latency). DW 2 carries two 16-bit latency values (one for snoop traffic, one for no-snoop traffic). Each value is encoded as `Requirement[15] | Reserved[14:13] | Scale[12:10] | Value[9:0]`; the effective latency is `Value * scale_to_ns(Scale)`. A `Requirement = 0` value means "no latency requirement". The Endpoint may emit LTR at any time as its workload changes.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.18: Latency Tolerance Reporting
- PCI Express Base Specification, section 7.8.2: Latency Tolerance Reporting Extended Capability
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [Intel, "Optimizing for power and performance with PCI Express LTR"](https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/io-virtualization-power-management-paper.pdf)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10000` (To RC) |
| Length[9:0] | `0` (data is in DW 2, not payload) |
| Message Code | `0x10` |
| Requester ID[15:0] | the function reporting latency |
| Tag[7:0] | Reserved |
| No-Snoop Latency (DW 2 high 16 bits) | `Req | Rsv | Scale | Value` |
| Snoop Latency (DW 2 low 16 bits) | `Req | Rsv | Scale | Value` |
| DW 3 | Reserved |

### Per-16-bit-field encoding

```
       bit:  15         14:13       12:10        9:0
            ┌─────┬───────────┬───────────┬──────────────┐
            │ Req │   Rsvd    │   Scale   │    Value     │
            └─────┴───────────┴───────────┴──────────────┘

       Req = 1 means the value is a real requirement;
             0 means no requirement (use any latency).
       Scale encodes the unit:
         000 = 1 ns
         001 = 32 ns
         010 = 1024 ns
         011 = 32768 ns
         100 = 1048576 ns (= ~1 ms)
         101 = 33554432 ns (= ~34 ms)
         others = Reserved
       Value = raw 10-bit count.
       Effective latency = Value * scale_unit(Scale) ns.
```

## DETAILS

