---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Completion without Data (Cpl)

A Completion without Data, usually abbreviated as Cpl, is a 3DW Transaction Layer Packet (TLP) that the Completer uses to close out a non-posted request when there is no payload to return. There are two situations in which the Completer is supposed to emit a Cpl rather than a CplD. The first is when the request was a non-posted write (an IOWr or a CfgWr) and the write succeeded; in that case the Cpl serves as an explicit acknowledgement so that the Requester knows the write has been accepted. The second is when any non-posted request, read or write, has failed; in that case the Cpl carries an error code in its Completion Status (CS) field and tells the Requester what went wrong. The header itself carries the Completer ID, the CS field, the Byte Count, the original Requester ID, the Tag value copied verbatim from the request, and a Lower Address field; there is no payload at all.

```
PCIe Completion without Data (Cpl) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 000 │  01010  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴─┬─┼───┴────────────────────┤
DW 1       │         Completer ID          │ CS  │B│       Byte Count       │
           ├───────────────────────────────┼─────┴─┴───────┬─┬──────────────┤
DW 2       │         Requester ID          │      Tag      │R│Lower Address │
           └───────────────────────────────┴───────────────┴─┴──────────────┘
```

## SUMMARY

On its way back through the fabric a Cpl is routed by ID. The Requester ID in `DW 2` is the destination address that bridges use to forward the TLP back to the originator, and the Completer ID identifies the Function that produced the response. The 3-bit Completion Status (`CS`) field encodes the outcome of the original request: `000` for Successful Completion (SC), `001` for Unsupported Request (UR), `010` for Configuration Request Retry Status (CRS), and `100` for Completer Abort (CA). The B bit, also known as BCM (Byte Count Modified), is a vestige from PCI-X and is always zero on PCIe-native links. The `Byte Count` field carries the number of bytes still outstanding in the original transaction, which is zero for the final Cpl of a write completion. The `Lower Address[6:0]` field carries the low 7 bits of the first byte address the Cpl represents and is meaningful only when a read completion has been split into multiple CplDs.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.9: Request and Completion Rules
- PCI Express Base Specification, section 2.2.9.2: Completion Header Format
- PCI Express Base Specification, section 2.3.1: Completion Routing
- PCI Express Base Specification, section 2.3.1.1: Successful Completion
- PCI Express Base Specification, section 2.3.1.2: Failed Completion Status Codes
- PCI Express Base Specification, section 2.3.2.1: CRS Software Visibility

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `000` (3DW, no payload) |
| Type[4:0] | `01010` |
| Length[9:0] | `0` (no data) |
| Completer ID[15:0] | Function that produced the Cpl |
| CS[2:0] | Completion Status: `000`=SC, `001`=UR, `010`=CRS, `100`=CA |
| B | BCM (Byte Count Modified, PCI-X compatibility); `0` for PCIe-only |
| Byte Count[11:0] | bytes remaining including this Cpl (`0` for write completions) |
| Requester ID[15:0] | original Requester BDF, used to route the Cpl back |
| Tag[7:0] | copied from the original request, matches outstanding request |
| Lower Address[6:0] | low 7 bits of the byte address (used for split read completions; for write completions reads `0x00`) |

## DETAILS

