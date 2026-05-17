---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# PME_Turn_Off

PME_Turn_Off is a 4DW Msg (no data) Broadcast from Root Complex. It announces to every downstream Function that the platform is about to enter a low-power state in which the PCIe link will lose power; each Function must acknowledge with PME_TO_Ack so the Root Complex knows the hierarchy is ready to shut down. The Message Code is `0x19` and the Type encodes `10011` (Broadcast from RC).

```
PME_Turn_Off  (Routing: Broadcast from RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10011  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00011001    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

When the system is about to suspend (S3 / S4) or otherwise enter a state where PCIe links go down, the Root Complex emits PME_Turn_Off as a broadcast. Every Switch downstream port replicates it to every active link; every Endpoint receives a copy. Each Function must respond with PME_TO_Ack within a specific time window. Once every Function has acknowledged (or the timeout expires), the platform proceeds with the power-down. This handshake gives Functions a chance to finish in-flight transactions, flush queues, and quiesce before they lose power.

## SPECIFICATIONS

- PCI Express Base Specification, section 5.3.3: PME_Turn_Off / PME_TO_Ack Handshake
- PCI Express Base Specification, section 6.1.4: Power Management Event Mechanism
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCI Power Management Specification, Revision 1.2

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10011` (Broadcast from RC) |
| Length[9:0] | `0` |
| Message Code | `0x19` |
| Requester ID[15:0] | the Root Complex's own ID |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

