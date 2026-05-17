---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Vendor_Defined Type 1 Message

A Vendor_Defined Type 1 Message is a 4DW Msg or MsgD (`Fmt[2:0] = 001` or `011`) routed By ID. The Message Code is `0x7F`. Type 1 differs from Type 0 (Message Code `0x7E`) in the receiver's required behaviour. A receiver that does not recognise the vendor's sub-type silently discards the Message rather than emitting Unsupported Request. This makes Type 1 safe for probing and broadcast-style protocols where the sender cannot enumerate which peers support the protocol.

```
Vendor_Defined Type 1  (Routing: By ID)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ Fmt │  10010  │R│ TC  │R│R│0│0│D│E│Att│00 │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    01111111    │
           ├───────────────────────────────┼───────────────┴────────────────┤
DW 2       │         Dest ID (BDF)         │           Vendor ID            │
           ├───────────────────────────────┴────────────────────────────────┤
DW 3       │                  Vendor-defined header bytes                   │
           ├────────────────────────────────────────────────────────────────┤
DW 4       │              Vendor-defined payload (0..N DWords)              │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

Vendor_Defined Type 1 has the same shape as Type 0. The Vendor ID in DW 2 low 16 bits identifies which vendor's protocol applies; the Dest ID in DW 2 high 16 bits names the target Function. DW 3 carries vendor-defined header bytes; DW 4..N carry zero or more DWords of vendor payload. The receiver that does not recognise the Vendor ID or sub-type drops the Message silently and emits no response of any kind. The sender has no way to tell whether the Message reached a supporting peer; the protocol must layer its own acknowledgement on top.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.13: Vendor-Defined Messages
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (Msg) or `011` (MsgD) |
| Type[4:0] | `10010` (By ID) |
| Length[9:0] | `0` or DW count of payload |
| Message Code | `0x7F` |
| Requester ID[15:0] | the function emitting the Message |
| Tag[7:0] | vendor-defined |
| Dest ID (DW 2 high 16 bits) | target Endpoint's BDF |
| Vendor ID (DW 2 low 16 bits) | PCI-SIG-assigned vendor identifier |
| DW 3 | vendor-defined header bytes |
| DW 4..N | vendor-defined payload |

## DETAILS

