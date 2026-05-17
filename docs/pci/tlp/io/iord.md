---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# I/O Read Request (IORd)

An I/O Read Request, usually abbreviated as IORd, is a non-posted Transaction Layer Packet (TLP) that reads a single four-byte DWord from a Function's I/O space. The I/O space is the legacy address space (16 bits on conventional PCI and 32 bits on PCIe) that predates Memory-Mapped I/O. PCIe carries an IORd as a 3DW header with no payload at all. The `Length` field is always 1, and both `TC` and `Attr` are required to be 0. The Completer is supposed to respond with a single CplD carrying the four bytes if the read succeeds, or with a Cpl whose `CS` field carries an error code if it does not.

```
PCIe I/O Read Request (IORd) TLP, Header Format
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 000 │  00010  │R│ 000 │R│R│0│0│D│E│00 │00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │ 0000  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           └───────────────────────────────────────────────────────────┴─┴──┘
```

## SUMMARY

PCIe I/O Read Requests exist primarily to support the legacy x86 I/O port instructions (`in`, `inl`, `outb`, `outl` and friends) when they happen to target PCIe Endpoints whose ancestors expose an I/O BAR. During enumeration the OS allocates an I/O resource window for each bridge that has children with I/O BARs, and every Switch along the path programs an I/O Base / I/O Limit window so that the bridge knows which range to forward downstream. When the CPU executes a port-input instruction afterwards, the Root Complex translates it into an IORd of `Length = 1` and routes it downstream as appropriate. The Completer is supposed to answer with a CplD carrying the four bytes (with FBE selecting which of them are meaningful) or with a Cpl carrying an error status. Modern Endpoints rarely implement an I/O BAR at all, because the PCIe specification actively encourages MMIO instead.

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
| Fmt[2:0] | `000` (3DW, no payload) |
| Type[4:0] | `00010` |
| Length[9:0] | `1` (always single DW) |
| TC[2:0] | `0` (required) |
| Attr[1:0] | `00` (Relaxed Ordering and No Snoop both forbidden) |
| LBE[3:0] | `0000` (Length=1 ⇒ no LBE) |
| FBE[3:0] | selects which of the four bytes are valid |
| Address[31:2] | DW-aligned 32-bit I/O space address |

I/O space is at most 32 bits wide. IORd is always 3DW; there is no 4DW form.

## DETAILS

