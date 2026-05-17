---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Message with Data (MsgD)

A Message with Data (MsgD) is a 4DW TLP identical in shape to Msg except for `Fmt[1] = 1` which announces a payload following the header. The payload size is given by `Length[9:0]`, the same field used by Memory transactions. The payload semantics are Message-specific: Set_Slot_Power_Limit puts the power-limit encoding in a single DW; PTM ResponseD carries the propagation delay; Vendor_Defined MsgD carries vendor-specific data of arbitrary length.

```
PCIe Message with Data (MsgD) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  10rrr  │R│ TC  │R│R│0│0│D│E│Att│00 │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │  Message Code  │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                  Message-specific (see notes)                  │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                  Message-specific (see notes)                  │
           ├────────────────────────────────────────────────────────────────┤
DW 4..N    │                     Payload Data (DWords)                      │
           │                              ...                               │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

MsgD carries Length DWords of payload after the 4DW header. Routing follows the same `rrr` sub-field as Msg. The Tag field, DW 2, and DW 3 carry Message-specific routing or data words; the actual data of the Message lives in DW 4..N. Most Messages are Msg (no data); the spec-defined MsgD instances are Set_Slot_Power_Limit (Length = 1), PTM ResponseD (Length = 3, master-time low + master-time high + propagation delay), and Vendor_Defined Type 0 / Type 1 (variable Length per vendor protocol).

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.6: Message Request Type Encodings
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCI Express Base Specification, section 2.2.9: Routing Sub-field Encoding
- PCI Express Base Specification, section 7.5.3.13: Slot Capabilities (Set_Slot_Power_Limit)
- PCI Express Base Specification, section 6.21: Precision Time Measurement
- PCI Express Base Specification, section 6.13: Vendor-Defined Messages

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `011` (4DW with data) |
| Type[4:0] | `10rrr` (rrr = routing sub-field) |
| Length[9:0] | DW count of payload |
| TC[2:0] | typically `0` |
| Attr[1:0] | typically `00` |
| Tag[7:0] | Message-specific |
| Message Code[7:0] | identifies the Message family |
| DW 2 | Message-specific routing or data word |
| DW 3 | Message-specific routing or data word |
| Payload (DW 4..N) | `Length` DWords of Message-specific data |

## DETAILS

