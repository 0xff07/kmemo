---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Vendor_Defined Type 0 Message

A Vendor_Defined Type 0 Message is a 4DW Msg or MsgD (`Fmt[2:0] = 001` or `011`) routed By ID. It carries vendor-specific signalling between two functions on the same PCIe fabric. The Message Code is `0x7E` and the Type encodes `10010` (By ID). Type 0 distinguishes itself from Type 1 (Message Code `0x7F`) by the receiver's required behaviour. A receiver that does not recognise the vendor's sub-type must report Unsupported Request.

```
Vendor_Defined Type 0  (Routing: By ID)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ Fmt │  10010  │R│ TC  │R│R│0│0│D│E│Att│00 │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    01111110    │
           ├───────────────────────────────┼───────────────┴────────────────┤
DW 2       │         Dest ID (BDF)         │           Vendor ID            │
           ├───────────────────────────────┴────────────────────────────────┤
DW 3       │                  Vendor-defined header bytes                   │
           ├────────────────────────────────────────────────────────────────┤
DW 4       │              Vendor-defined payload (0..N DWords)              │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

Vendor_Defined Type 0 Messages carry vendor protocols over the PCIe fabric without consuming any of the function's own BAR space. The Vendor ID in DW 2 low 16 bits identifies which vendor's protocol is in play; the Dest ID in DW 2 high 16 bits names the target Function. DW 3 carries vendor-defined header bytes; DW 4..N carry zero or more DWords of vendor payload, as indicated by Length. A function receiving a Type 0 message whose Vendor ID it does not recognise (or whose sub-protocol within the recognised Vendor ID is unknown) must emit a Cpl with `CS = UR`; this gives the sender a definite failure signal.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.13: Vendor-Defined Messages
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCI Express Base Specification, section 2.3.1.2: Failed Completion Status Codes (UR)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (Msg) or `011` (MsgD) |
| Type[4:0] | `10010` (By ID) |
| Length[9:0] | `0` or DW count of payload |
| Message Code | `0x7E` |
| Requester ID[15:0] | the function emitting the Message |
| Tag[7:0] | vendor-defined (often part of the protocol's sequence number) |
| Dest ID (DW 2 high 16 bits) | target Endpoint's BDF |
| Vendor ID (DW 2 low 16 bits) | PCI-SIG-assigned vendor identifier |
| DW 3 | vendor-defined header bytes |
| DW 4..N | vendor-defined payload |

## DETAILS

