---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Configuration Read Type 0 (CfgRd0)

A Configuration Read Type 0, usually abbreviated as CfgRd0, is one of the standard Transaction Layer Packets defined by the PCI Express Base Specification. It is used whenever an agent needs to read one DWord from the Configuration Space of a Function that sits directly on the secondary bus of the bridge issuing the TLP. Configuration Space is a per-Function 4 KiB region that hardware exposes through a separate address space (distinct from MMIO and from I/O space), and the kernel uses it during enumeration and at runtime to discover Functions, size BARs, program PCIe Capabilities, and manage error reporting. Because the Type 0 form is consumed by a Function sitting on the local bus, it is the form that every Switch downstream port and the Root Complex's host bridge eventually emits once a Configuration Request has been forwarded down through the hierarchy and is ready for delivery to the target.

The TLP is always 3DW, always carries `Length = 1`, and always sets `TC` and `Attr` to zero. It has no data payload. The address fields in DW 2 are reinterpreted as `{Bus, Device, Function, Extended Register, Register}` so the TLP can name any DWord inside the 4 KiB of Configuration Space of any Function on the local bus. The Completer is supposed to respond with a single Completion-with-Data (CplD) carrying the four bytes, with First DW Byte Enables (FBE) in the header selecting which of those bytes the Requester actually cares about. If the read fails, the response comes back as a Completion-without-Data (Cpl) whose Completion Status (`CS`) field encodes one of `UR` (no Function present at that Bus/Device/Function triple), `CA` (the Function aborted the access), or `CRS` (the Function is still initialising and is not yet ready to answer), the last of which is unique to Configuration accesses.

```
PCIe Configuration Read Type 0 (CfgRd0) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 000 │  00100  │R│ 000 │R│R│0│0│D│E│00 │00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │ 0000  │  FBE   │
           ├───────────────┬─────────┬─────┼───────┬───────┼───────┴───┬────┤
DW 2       │    Bus Num    │   Dev   │ Fn  │   R   │Ext Reg│ Register  │ R  │
           └───────────────┴─────────┴─────┴───────┴───────┴───────────┴────┘
```

## SUMMARY

A Configuration Read Type 0 travels from a Switch downstream port (or from the Root Complex's host bridge) to a Function that sits directly on the bridge's secondary bus. Configuration accesses are the foundation of how the kernel enumerates and configures every PCIe Function in the hierarchy, and Type 0 is the variant that actually lands at the target. Bus number, Device number, and Function number together identify which Function on the local bus should claim the TLP, and Extended Register (4 bits) concatenated with Register (6 bits) together select one of 1024 DWords (4 KiB) inside that Function's Configuration Space. On success the Completer answers with a single CplD carrying the requested DWord, with FBE selecting which of the four bytes the Requester actually cares about.

Failure paths produce a Cpl whose Completion Status encodes the failure code (`UR` when no Function decodes the address, `CA` when the Function aborts the access, or `CRS` when the Function is still initialising). The `CRS` code is unique to Configuration accesses and is normally not an error at all; rather, it is the way an Endpoint that has just come out of reset tells the Requester to try again later. The Root Complex handles `CRS` either by retrying transparently or, when CRS Software Visibility has been enabled, by synthesising a `Vendor ID` of `0x0001` so that the kernel can decide for itself how long to keep polling.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.4: Configuration Request Type Encodings
- PCI Express Base Specification, section 2.2.8.6: Configuration Request Header Format
- PCI Express Base Specification, section 2.3.2: Configuration Request Routing
- PCI Express Base Specification, section 2.3.2.1: Configuration Request Retry Status (CRS)
- PCI Express Base Specification, section 6.2.2.3: Configuration Routing
- PCI Express Base Specification, section 7.2: PCI Express Configuration Mechanisms

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `000` (3DW, no payload) |
| Type[4:0] | `00100` |
| Length[9:0] | `1` (always single DW) |
| TC[2:0] | `0` |
| Attr[1:0] | `00` |
| LBE[3:0] | `0000` |
| FBE[3:0] | selects which of the four bytes of the response are valid |
| Bus Num[7:0] | target bus |
| Device[4:0] | target device on that bus |
| Function[2:0] | target function within that device |
| Extended Register[3:0] | upper 4 bits of the register offset |
| Register[5:0] | lower 6 bits of the register offset (DW-aligned) |

`Extended Register[3:0] || Register[5:0]` form a 10-bit DWord offset, addressing 1024 DWords = 4 KiB of Configuration Space.

## DETAILS

