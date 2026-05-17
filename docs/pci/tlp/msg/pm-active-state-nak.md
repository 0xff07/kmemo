---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# PM_Active_State_Nak

PM_Active_State_Nak is a 4DW Msg (no data) routed Local (terminates at the receiver). It is emitted by an upstream port to refuse a downstream port's request to enter the ASPM L0s or L1 link-power state. The Message Code is `0x14` and the Type encodes `10100` (Local routing).

```
PM_Active_State_Nak  (Routing: Local)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10100  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00010100    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

ASPM (Active State Power Management) lets a PCIe link drop to a lower-power state (L0s or L1) when both endpoints agree there is no traffic in flight. The downstream port initiates the request by emitting protocol DLLPs (not TLPs). The upstream port may refuse the transition by emitting PM_Active_State_Nak back to the downstream port. The Nak is Local-routed because the Message is meaningful only on the immediate link; it never propagates further upstream. After receiving PM_Active_State_Nak, the downstream port stays in L0 and may retry later.

## SPECIFICATIONS

- PCI Express Base Specification, section 5.4: Active State Power Management
- PCI Express Base Specification, section 5.4.1: L0s
- PCI Express Base Specification, section 5.4.2: L1
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [LWN, "PCIe Active State Power Management"](https://lwn.net/Articles/449369/)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10100` (Local) |
| Length[9:0] | `0` |
| Message Code | `0x14` |
| Requester ID[15:0] | the upstream port refusing the request |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

