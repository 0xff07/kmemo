---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Set_Slot_Power_Limit

Set_Slot_Power_Limit is a 4DW MsgD (with data) routed By ID. The downstream port of a Switch or Root Complex emits this Message to inform the downstream Endpoint how much power the slot is allowed to draw. The Endpoint records the value in its Slot Capabilities register; the device's firmware may then throttle features that depend on the available power budget. The Message Code is `0x50`, Type encodes `10010` (By ID), and a single DWord of payload carries the power-limit encoding.

```
Set_Slot_Power_Limit  (Routing: By ID, with data)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  10010  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    01010000    │
           ├───────────────────────────────┼───────────────┴────────────────┤
DW 2       │         Dest ID (BDF)         │            Reserved            │
           ├───────────────────────────────┴────────────────────────────────┤
DW 3       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 4       │       Power Limit [7:0] | Scale [9:8] | Reserved [31:10]       │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

PCIe sockets and add-in card slots have a defined maximum power budget that depends on the form factor, the slot's voltage rails, and the platform's power-supply design. The downstream port whose slot contains the device sends Set_Slot_Power_Limit to advertise the budget. The Endpoint's firmware reads the new limit from its Slot Capabilities register and may disable optional features (high-power link speeds, multi-port operation, optional integrated GPUs) to fit. The encoded value is `Power Limit Value[7:0]` (8 bits) plus `Power Limit Scale[1:0]` (2 bits) that selects `1.0 W` / `0.1 W` / `0.01 W` / `0.001 W` per unit. The Routing-by-ID form targets a specific Endpoint via the Dest ID in DW 2.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.7.3: Slot Power Limit Support
- PCI Express Base Specification, section 7.5.3.13: Slot Capabilities Register
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCIe Card Electromechanical (CEM) Specification

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [PCISIG, PCIe Card Electromechanical Specification](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `011` (4DW with data) |
| Type[4:0] | `10010` (By ID) |
| Length[9:0] | `1` |
| Message Code | `0x50` |
| Requester ID[15:0] | the downstream port emitting the Message |
| Tag[7:0] | Reserved |
| Dest ID (DW 2 high 16 bits) | target Endpoint's BDF |
| Payload DW 4[7:0] | Power Limit Value (raw) |
| Payload DW 4[9:8] | Scale: `00` = 1.0 W, `01` = 0.1 W, `10` = 0.01 W, `11` = 0.001 W |
| Payload DW 4[31:10] | Reserved |

The effective power budget is `Power Limit Value * scale[Scale]`.

## DETAILS

