---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# PTM Response / ResponseD

PTM Response and PTM ResponseD are 4DW MsgD (with data) routed Local. They are emitted by a PTM Responder in reply to a PTM Request. The Message Code is `0x53`. The Type encodes `10100` (Local). Response carries 2 DW of payload (64-bit Master Time only); ResponseD carries 3 DW (adds 32-bit Propagation Delay). Both are MsgD (`Fmt[2:0] = 011`).

```
PTM Response / ResponseD  (Routing: Local, with data)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  10100  │R│ TC  │R│R│0│0│D│E│Att│00 │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    01010011    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                    Master Time bits [31:0]                     │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                    Master Time bits [63:32]                    │
           ├────────────────────────────────────────────────────────────────┤
DW 4       │          Propagation Delay [31:0] (ResponseD only)             │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

The PTM Responder receives a PTM Request and replies with the 64-bit master time observed at its end of the link. If the Responder has measured the link's propagation delay through a calibration cycle, it emits ResponseD with a third payload DWord carrying the delay; otherwise it emits Response without the delay. The Requestor uses the master time plus its own RX timestamp (and the optional propagation delay) to compute the local clock offset and update its time-base register.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.21: Precision Time Measurement
- PCI Express Base Specification, section 7.8.1: Precision Time Measurement Extended Capability
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `011` (4DW with data) |
| Type[4:0] | `10100` (Local) |
| Length[9:0] | `2` for Response, `3` for ResponseD |
| Message Code | `0x53` |
| Requester ID[15:0] | the Responder (upstream port) |
| Tag[7:0] | Reserved |
| Master Time low (DW 2) | 32-bit |
| Master Time high (DW 3) | 32-bit |
| Propagation Delay (DW 4) | ResponseD only; 32-bit |

## DETAILS

