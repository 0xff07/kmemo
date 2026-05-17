---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Configuration Read Type 1 (CfgRd1)

A Configuration Read Type 1, usually abbreviated as CfgRd1, is one of the standard Transaction Layer Packets defined by the PCI Express Base Specification. It is used whenever an agent needs to read one DWord from the Configuration Space of a Function that does not sit directly on the local bus of the issuing bridge. On the wire the packet is identical in shape to its Type 0 sibling (see the Type 0 vs Type 1 section below), differing only in the value of `Type[4:0]`, which is `00101` instead of `00100`. Intermediate bridges are supposed to forward a CfgRd1 unchanged through their secondary side until the TLP reaches the one bridge whose own Secondary Bus Number matches the target Bus. At that final hop the bridge rewrites the Type field, converts the CfgRd1 into a CfgRd0, and emits the result onto its secondary link, where the target Function consumes it and replies.

CfgRd1, therefore, is the form that the Root Complex and downstream Switches use to reach any Function that lives more than one bus away, and it is the form that exists on every PCIe wire segment except the final one. Because the address fields and the response shape are the same as for CfgRd0, the only thing CfgRd1 contributes is the routing semantics: bridges along the path consult their Primary, Secondary, and Subordinate Bus Number registers to decide whether to consume the TLP (by converting to CfgRd0), to forward it further (by leaving the Type as CfgRd1), or to reject it as an Unsupported Request when the target Bus does not fall inside the bridge's window at all.

```
PCIe Configuration Read Type 1 (CfgRd1) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 000 │  00101  │R│ 000 │R│R│0│0│D│E│00 │00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │ 0000  │  FBE   │
           ├───────────────┬─────────┬─────┼───────┬───────┼───────┴───┬────┤
DW 2       │    Bus Num    │   Dev   │ Fn  │   R   │Ext Reg│ Register  │ R  │
           └───────────────┴─────────┴─────┴───────┴───────┴───────────┴────┘
```

## SUMMARY

A Configuration Read Type 1 carries the same `{Bus, Device, Function, Extended Register, Register}` payload as its Type 0 sibling, and the distinction between the two forms is purely in how bridges along the path treat the TLP. Each bridge inspects the target Bus number in DW 2 and compares it against its own bus-number registers. When the target Bus equals the bridge's Secondary Bus Number, the bridge is supposed to rewrite the `Type[4:0]` field from `00101` to `00100` and emit the resulting CfgRd0 on its secondary side; when the target Bus falls strictly between the Secondary Bus and the Subordinate Bus, the bridge is expected to forward the CfgRd1 unchanged onto its secondary side so that the next bridge in the chain can take the next decision; and when the target falls outside the bridge's window entirely, the bridge synthesises a Cpl carrying `CS = UR`.

The kernel does not pick directly between CfgRd0 and CfgRd1. The choice is made by the Root Complex hardware (and by every downstream Switch) on the basis of the local bus-number windows that the kernel has already programmed during enumeration. From software's point of view there is a single Configuration Read API; the Type 0 versus Type 1 distinction is visible only when the TLPs are captured on the wire or when AER reports an error.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.4: Configuration Request Type Encodings
- PCI Express Base Specification, section 2.2.8.6: Configuration Request Header Format
- PCI Express Base Specification, section 2.3.2: Configuration Request Routing
- PCI Express Base Specification, section 6.2.2.3: Configuration Routing
- PCI Express Base Specification, section 7.2: PCI Express Configuration Mechanisms
- PCI-to-PCI Bridge Architecture Specification, Revision 1.2 (rules for Type 1 forwarding and conversion)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `000` (3DW, no payload) |
| Type[4:0] | `00101` |
| Length[9:0] | `1` |
| TC[2:0] | `0` |
| Attr[1:0] | `00` |
| LBE[3:0] | `0000` |
| FBE[3:0] | selects valid bytes of the response DW |
| Bus Num[7:0] | target bus number |
| Device[4:0] | target device |
| Function[2:0] | target function |
| Extended Register[3:0] | upper 4 bits of register offset |
| Register[5:0] | lower 6 bits of register offset |

The wire-format differs from CfgRd0 only in `Type[4:0]` (last bit set to `1`).

## DETAILS

