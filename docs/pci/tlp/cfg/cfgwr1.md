---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Configuration Write Type 1 (CfgWr1)

A Configuration Write Type 1, usually abbreviated as CfgWr1, is one of the standard Transaction Layer Packets defined by the PCI Express Base Specification. It is used whenever an agent needs to write one DWord into the Configuration Space of a Function that does not sit directly on the local bus of the issuing bridge. On the wire the packet has exactly the same shape as its Type 0 sibling (a 3DW header followed by one payload DWord, with `{Bus, Device, Function, Extended Register, Register}` in DW 2 and the value to be written in DW 3), and differs only in the value of `Type[4:0]`, which is `00101` instead of `00100`. Intermediate bridges are supposed to forward a CfgWr1 unchanged until the TLP reaches the one bridge whose own Secondary Bus Number matches the target Bus. At that final hop the bridge rewrites the Type field, converts the CfgWr1 into a CfgWr0, and emits the result onto its secondary link, where the target Function commits the write and responds with a Cpl.

CfgWr1 is therefore the form that the Root Complex uses (in concert with downstream Switches) to reach any Function that lives more than one bus away, and it is the form that exists on every wire segment except the final one. Because the address fields and the response shape are identical to CfgWr0, the only thing CfgWr1 contributes is the routing semantics: each bridge along the path consults its Primary, Secondary, and Subordinate Bus Number registers to decide whether to consume the TLP (by converting to CfgWr0), to forward it further (by keeping the Type as CfgWr1), or to reject it as Unsupported Request when the target Bus does not fall inside the bridge's window at all.

```
PCIe Configuration Write Type 1 (CfgWr1) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  00101  │R│ 000 │R│R│0│0│D│E│00 │00 │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │ 0000  │  FBE   │
           ├───────────────┬─────────┬─────┼───────┬───────┼───────┴───┬────┤
DW 2       │    Bus Num    │   Dev   │ Fn  │   R   │Ext Reg│ Register  │ R  │
           ├───────────────┴─────────┴─────┴───────┴───────┴───────────┴────┤
DW 3       │                          Data (1 DW)                           │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

A Configuration Write Type 1 carries the same payload and addressing fields as its Type 0 sibling, and the only meaningful difference between the two forms is in how bridges along the path treat the TLP. Each bridge inspects the target Bus number in DW 2 and compares it against its own Primary, Secondary, and Subordinate Bus Number registers. When the target Bus equals the bridge's Secondary Bus, the bridge is supposed to rewrite the `Type[4:0]` field from `00101` to `00100` and emit the resulting CfgWr0 on its secondary side; when the target Bus lies strictly between the Secondary and the Subordinate, the bridge is expected to forward the CfgWr1 unchanged onto its secondary side; and when the target falls outside the bridge's window entirely, the bridge synthesises a Cpl carrying `CS = UR`.

The kernel never picks directly between Type 0 and Type 1. The choice is made by the Root Complex hardware (and by every downstream Switch) on the basis of the local bus-number windows that the kernel has already programmed during enumeration. From software's point of view there is a single Configuration Write API; the Type 0 versus Type 1 distinction is visible only when the TLPs are captured on the wire or when AER reports an error.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.4: Configuration Request Type Encodings
- PCI Express Base Specification, section 2.2.8.6: Configuration Request Header Format
- PCI Express Base Specification, section 2.3.2: Configuration Request Routing
- PCI Express Base Specification, section 6.2.2.3: Configuration Routing
- PCI Express Base Specification, section 7.2: PCI Express Configuration Mechanisms
- PCI-to-PCI Bridge Architecture Specification (Type 1 conversion rules)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) |
| Type[4:0] | `00101` |
| Length[9:0] | `1` |
| TC[2:0] | `0` |
| Attr[1:0] | `00` |
| LBE[3:0] | `0000` |
| FBE[3:0] | selects valid bytes within DW 3 payload |
| Bus Num[7:0] | target Bus number |
| Device[4:0] | target Device |
| Function[2:0] | target Function |
| Extended Register[3:0] | upper 4 bits of register offset |
| Register[5:0] | lower 6 bits of register offset |
| Data DW (DW 3) | payload value |

## DETAILS

