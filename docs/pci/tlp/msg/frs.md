---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# FRS (Function Readiness Status)

FRS is a 4DW Msg (no data) routed To Root Complex. It is emitted by a Function after it has finished initialising following a reset (Fundamental Reset, Conventional Reset, FLR, or Vendor-Specific Reset, as encoded in the FRS Reason field) to tell the OS it is ready, letting the OS skip the spec-mandated worst-case 1-second wait before probing. The Message Code is `0x4A` and the Type encodes `10000` (To RC). DW 2 high bits are Reserved; the low 8 bits carry the FRS Reason.

```
FRS (Function Readiness Status)  (Routing: To RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    01001010    │
           ├───────────────────────────────┴───────────────┼────────────────┤
DW 2       │                   Reserved                    │   FRS Reason   │
           ├───────────────────────────────────────────────┴────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

The PCIe spec mandates that an OS wait up to 1 second after a reset before deciding the function has failed to initialise. This worst-case wait is necessary because some functions take that long to load firmware (e.g. RDMA NICs with large microcode, GPUs with multi-stage BIOS). FRS lets a function announce "I'm ready" early. The OS sees the FRS Message at the Root Complex, identifies the function by its Requester ID, and proceeds to enumerate / probe immediately instead of polling Vendor ID or waiting out the 1 s window. The FRS Reason in DW 2 byte 3 indicates which reset class the function is exiting from.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.6.1: Function-Level Reset (FLR)
- PCI Express Base Specification, section 6.20: Function Readiness Status
- PCI Express Base Specification, section 7.8.x: FRS Queueing Extended Capability
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10000` (To RC) |
| Length[9:0] | `0` |
| Message Code | `0x4A` |
| Requester ID[15:0] | the function reporting readiness |
| Tag[7:0] | Reserved |
| FRS Reason (DW 2 byte 3) | indicates the reset class the function is recovering from |
| DW 2 high bits | Reserved |
| DW 3 | Reserved |

## DETAILS

