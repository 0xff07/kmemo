---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# I/O Write Request (IOWr)

An I/O Write Request, usually abbreviated as IOWr, is a non-posted Transaction Layer Packet (TLP) that writes a single four-byte DWord into a Function's I/O space. Unlike a Memory Write Request, the IOWr is non-posted, which means that the Requester is supposed to wait for the Completer's Cpl before considering the write complete. The TLP itself is always 3DW, always has `Length = 1`, and always has `TC = 0` and `Attr = 0`; immediately after the header it carries one DWord of payload. The FBE nibble in the header selects which of the four payload bytes the Completer is supposed to commit to its register file.

```
PCIe I/O Write Request (IOWr) TLP, Header Format
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  00010  │R│ 000 │R│R│0│0│D│E│00 │00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │ 0000  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 3       │                          Data (1 DW)                           │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

The IOWr exists primarily for symmetry with the IORd, so that the host bridge has somewhere to send a legacy `out` instruction when its target port has been assigned to a PCIe-backed Function. Because the TLP is non-posted, the Requester is supposed to wait for a Cpl with `CS = SC` before retiring the originating CPU instruction. The Completer is expected to commit the write to its register file and then to emit the Cpl. From the originator's perspective the write is therefore synchronous, in the sense that by the time the `outb` or `outl` instruction retires the write has been positively acknowledged. If something fails (for example, the address is unclaimed or the Function explicitly aborts the write), the Cpl carries a non-SC `CS` value and the kernel records the failure through AER.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.5: I/O Request Type Encodings
- PCI Express Base Specification, section 2.2.8.5: I/O Request Header Format
- PCI Express Base Specification, section 2.4.1: I/O Ordering Rules
- PCI Express Base Specification, section 6.2.2.2: I/O Routing
- PCI Express Base Specification, section 7.5.1.3: Base Address Registers (I/O BAR)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) |
| Type[4:0] | `00010` |
| Length[9:0] | `1` (always single DW) |
| TC[2:0] | `0` (required) |
| Attr[1:0] | `00` (Relaxed Ordering and No Snoop both forbidden) |
| LBE[3:0] | `0000` (Length=1) |
| FBE[3:0] | selects valid bytes of the payload DW |
| Address[31:2] | DW-aligned 32-bit I/O space address |
| Data DW (DW 3) | the payload |

## DETAILS

