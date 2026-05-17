---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Unlock Message

Unlock is a 4DW Msg (no data) Broadcast from Root Complex. It terminates a Locked Transaction sequence on every link below the Root Complex. The Message Code is `0x00` and the Type encodes `10011` (Broadcast from RC). Locked Transactions are a legacy PCI feature carried forward into PCIe for compatibility; their use is restricted to Root Complex CPUs and only against legacy bridges. The Unlock Message tells every downstream device that the in-flight locked sequence is over and they may resume normal transaction processing.

```
Unlock  (Routing: Broadcast from RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10011  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00000000    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

Locked Transactions originated in PCI as a way for a CPU to perform an atomic read-modify-write against an MMIO target. The CPU issues a Memory Read Lock (MRdLk), then a Memory Write Lock (MWrLk) targeting the same MMIO address, with the link "held" between the two so no other agent can intervene. PCIe inherits this for backward compatibility, but PCIe-native devices and Switches treat Locked Transactions as a special case that must explicitly support a "lock holdoff" state. The Unlock Message is the broadcast that releases every downstream device from the locked state.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.5: Locked Transactions
- PCI Express Base Specification, section 2.2.6.6: Message Request Type Encodings
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10011` (Broadcast from RC) |
| Length[9:0] | `0` |
| Message Code | `0x00` |
| Requester ID[15:0] | the Root Complex emitting the broadcast |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

