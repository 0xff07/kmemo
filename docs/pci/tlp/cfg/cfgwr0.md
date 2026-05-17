---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Configuration Write Type 0 (CfgWr0)

A Configuration Write Type 0, usually abbreviated as CfgWr0, is one of the standard Transaction Layer Packets defined by the PCI Express Base Specification. It is used whenever an agent needs to write one DWord into the Configuration Space of a Function that sits directly on the secondary bus of the bridge issuing the TLP. Like its read sibling, CfgWr0 is the form that actually reaches the target Function; any Configuration Write that originates further upstream travels through the fabric as a CfgWr1 and is rewritten into a CfgWr0 only at the final hop, where the bridge whose Secondary Bus matches the target Bus performs the conversion.

The TLP is always 3DW followed by exactly one payload DWord, always carries `Length = 1`, and always sets `TC` and `Attr` to zero. The header re-purposes DW 2 to carry the target's Bus, Device, Function, Extended Register, and Register fields (see the Address overload section below), and the FBE nibble selects which of the four bytes inside the payload DWord are meaningful for the write. Because the transaction is non-posted, the Completer is required to acknowledge the write with a single Completion-without-Data (Cpl) whose Completion Status (`CS`) field encodes either `SC` (Successful Completion) on the normal path or one of `UR`, `CA`, or `CRS` on the failure paths. The acknowledgement is what makes a CfgWr0 synchronous from the kernel's point of view, and it is also what makes the helper safe to use as a barrier when programming a Function's registers in sequence.

```
PCIe Configuration Write Type 0 (CfgWr0) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  00100  │R│ 000 │R│R│0│0│D│E│00 │00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │ 0000  │  FBE   │
           ├───────────────┬─────────┬─────┼───────┬───────┼───────┴───┬────┤
DW 2       │    Bus Num    │   Dev   │ Fn  │   R   │Ext Reg│ Register  │ R  │
           ├───────────────┴─────────┴─────┴───────┴───────┴───────────┴────┤
DW 3       │                          Data (1 DW)                           │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

A Configuration Write Type 0 commits a software-supplied DWord into the Configuration Space registers of a Function that sits directly on the local bus of the issuing bridge. Because the transaction is non-posted, the Requester is supposed to wait for the matching Completion to come back before considering the write complete; in this respect CfgWr0 is quite different from the posted Memory Write (MWr) and behaves more like an Memory Read (MRd) in terms of synchronisation. The Completer's acknowledgement turns CfgWr0 into a natural barrier, which is why the kernel can write a BAR register, immediately read it back through a CfgRd0, and observe the new value without any explicit flush.

The address fields in DW 2 carry the target Bus, Device, and Function together with the 10-bit register offset assembled from the Extended Register and Register subfields, and the payload DWord in DW 3 carries the value that the Completer is supposed to store at that offset (with the FBE nibble selecting which of the four bytes are meaningful). On success the response comes back as a Cpl whose `CS` field is `SC` and whose Length is zero. Failures produce a Cpl with `CS = UR` when no Function decodes the address, `CS = CA` when the Function rejects the write, or `CS = CRS` when the Function is still initialising. CfgWr0 is the form that ultimately reaches the target; the longer-range CfgWr1 is the corresponding routed form that bridges convert into a CfgWr0 at the final hop.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.4: Configuration Request Type Encodings
- PCI Express Base Specification, section 2.2.8.6: Configuration Request Header Format
- PCI Express Base Specification, section 2.3.2: Configuration Request Routing
- PCI Express Base Specification, section 6.2.2.3: Configuration Routing
- PCI Express Base Specification, section 7.2: PCI Express Configuration Mechanisms

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) |
| Type[4:0] | `00100` |
| Length[9:0] | `1` (always single DW) |
| TC[2:0] | `0` |
| Attr[1:0] | `00` |
| LBE[3:0] | `0000` |
| FBE[3:0] | selects valid bytes within DW 3 (the payload) |
| Bus Num[7:0] | target bus |
| Device[4:0] | target device |
| Function[2:0] | target function |
| Extended Register[3:0] | upper 4 bits of the register offset |
| Register[5:0] | lower 6 bits of the register offset |
| Data DW (DW 3) | the value to write |

## DETAILS

