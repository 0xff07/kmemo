---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# PM_PME

PM_PME is a 4DW Msg (no data) routed To Root Complex. It is emitted by a Function whose internal logic detects a wake-up event and signals the Root Complex to wake the system or to bring the function back to D0. The Message Code is `0x18` and the Type encodes `10000` (To Root Complex).

```
PM_PME  (Routing: To RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00011000    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

A PCIe Function in a low-power state (D1, D2, D3hot) that wants to signal a wake event sends PM_PME upstream toward the Root Complex. The Function's Requester ID identifies which device initiated the wake. The Root Complex's PME service raises a PME interrupt to the OS; the kernel walks the hierarchy to find the function (matching against Requester ID), wakes its driver, and clears the function's PME status bit so the function can rearm. PM_PME is the PCIe-native equivalent of the PME# wire on legacy PCI.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.1.4: Power Management Event Mechanism
- PCI Express Base Specification, section 5.3.2: Function Power Management Events
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCI Power Management Specification, Revision 1.2

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [LWN, "Runtime PM for PCI devices"](https://lwn.net/Articles/444065/)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10000` (To RC) |
| Length[9:0] | `0` |
| Message Code | `0x18` |
| Requester ID[15:0] | function initiating the wake event |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

