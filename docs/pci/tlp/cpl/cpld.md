---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Completion with Data (CplD)

A Completion with Data, usually abbreviated as CplD, is a 3DW Transaction Layer Packet (TLP) that the Completer uses to deliver the bytes asked for by a successful non-posted read or AtomicOp. The payload follows the header as a sequence of one or more DWords; the header itself carries the original Requester ID, the Tag value the Requester chose for the original request, a Lower Address field that locates the bytes inside the requested range, and a Byte Count field that tells the Requester how many bytes are still outstanding (including the ones in the current CplD). Together these four fields let the Requester demultiplex the response back to the originating request without scanning a list of outstanding requests, and let it reassemble the data when the Completer chooses to split a single read response across multiple CplD TLPs.

```
PCIe Completion with Data (CplD) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  01010  │R│ TC  │R│R│0│0│D│E│Att│00 │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴─┬─┼───┴────────────────────┤
DW 1       │         Completer ID          │ CS  │B│       Byte Count       │
           ├───────────────────────────────┼─────┴─┴───────┬─┬──────────────┤
DW 2       │         Requester ID          │      Tag      │R│Lower Address │
           ├───────────────────────────────┴───────────────┴─┴──────────────┤
DW 3..N    │                     Payload Data (DWords)                      │
           │                              ...                               │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

A CplD carries between 1 and 1024 DWords of payload, and on its way back through the fabric it is routed by Requester ID. The Completer is allowed to break a large read response into several CplD TLPs aligned to its own Read Completion Boundary (RCB). Each CplD in the chain carries the Lower Address of the first byte it contains together with the Byte Count of bytes still outstanding including those in the current CplD. The Requester walks the chain by matching Tag and watching `Byte Count` drop to zero.

The same TLP shape is also used to respond to a successful AtomicOp. In that case the payload carries the pre-operation value of the target word so that the Requester can either use it directly (for FetchAdd or Swap) or compare it against its expected value (for CAS). If the request fails for any reason, the response is a Cpl without any data payload and a `CS` field that encodes the failure code.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.9: Request and Completion Rules
- PCI Express Base Specification, section 2.2.9.2: Completion Header Format
- PCI Express Base Specification, section 2.3.1: Completion Routing
- PCI Express Base Specification, section 2.3.1.1: Data Return for Read Requests
- PCI Express Base Specification, section 2.4.5: Read Request / Completion Ordering
- PCI Express Base Specification, section 7.5.3.4: Device Control Register (RCB)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) |
| Type[4:0] | `01010` |
| Length[9:0] | DW count of payload (`1..1024`); for AtomicOp success, equals the operand size in DW |
| Completer ID[15:0] | Function that produced the CplD |
| CS[2:0] | `000` (SC); other values turn it into Cpl |
| B | BCM (PCI-X compat); `0` for PCIe |
| Byte Count[11:0] | bytes still outstanding including this CplD; `0` means final CplD |
| Requester ID[15:0] | original Requester BDF |
| Tag[7:0] | copied from the original request |
| Lower Address[6:0] | low 7 bits of the first byte address this CplD covers |
| Payload (DW 3..N) | `Length` DWords of data |

## DETAILS

