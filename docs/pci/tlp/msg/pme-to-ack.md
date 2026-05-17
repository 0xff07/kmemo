---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# PME_TO_Ack

PME_TO_Ack is a 4DW Msg (no data) routed Gathered to Root Complex. It is the acknowledgement that each Function emits in response to PME_Turn_Off. Switches aggregate per-port acknowledgements, so a Switch upstream port emits one PME_TO_Ack upstream after collecting the corresponding Acks from every active downstream port. The Message Code is `0x1B` and the Type encodes `10101` (Gathered & routed to RC).

```
PME_TO_Ack  (Routing: Gathered to RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10101  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00011011    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

PME_TO_Ack closes the PME_Turn_Off handshake. After a Function receives the PME_Turn_Off broadcast, it finishes any in-flight transactions and emits PME_TO_Ack upstream. Switches gather Acks per-port, with each Switch upstream port waiting until every active downstream port has reported a PME_TO_Ack, then emitting a single aggregated PME_TO_Ack upstream. The Root Complex sees one PME_TO_Ack per immediate child port; once all immediate children have acknowledged, the platform is ready to power down. The "Gathered" routing collapses an N-way fan-in into a single upstream Message at each Switch, keeping the upstream link from being flooded with per-Function Acks.

## SPECIFICATIONS

- PCI Express Base Specification, section 5.3.3: PME_Turn_Off / PME_TO_Ack Handshake
- PCI Express Base Specification, section 6.1.4: Power Management Event Mechanism
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10101` (Gathered to RC) |
| Length[9:0] | `0` |
| Message Code | `0x1B` |
| Requester ID[15:0] | the function (Endpoint) or Switch upstream port emitting the Ack |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

