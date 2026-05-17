---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# PTM Request

PTM Request is a 4DW Msg (no data) routed Local. It is emitted by a PTM Requestor (an Endpoint that wants sub-nanosecond clock alignment with the platform) to ask its immediate upstream port for the current master time. The Message Code is `0x52` and the Type encodes `10100` (Local). DW 2 and DW 3 are Reserved.

```
PTM Request  (Routing: Local)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10100  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    01010010    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

PTM (Precision Time Measurement) lets PCIe components keep their internal clocks aligned to the platform's PTM master with sub-nanosecond accuracy. The Requestor sends PTM Request to its immediate upstream port; the upstream port (which acts as the PTM Responder) replies with PTM Response or PTM ResponseD. PTM Request travels exactly one link hop: `Type[4:0] = 10100` is "Local", so Switches and Root Complex do not forward the Message. The Requestor's hardware records the local time at TLP emission; the Responder's reply carries the master time observed by the Responder; the propagation delay (carried in ResponseD) lets the Requestor compute the round-trip latency.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.21: Precision Time Measurement
- PCI Express Base Specification, section 7.8.1: Precision Time Measurement Extended Capability
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [Intel, "PCIe Precision Time Measurement"](https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/io-virtualization-ptm-white-paper.pdf)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10100` (Local) |
| Length[9:0] | `0` |
| Message Code | `0x52` |
| Requester ID[15:0] | the function emitting the request |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

