---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Generic 3DW TLP Header

The 3DW (three-DWord) TLP header is the 12-byte common header used by Transaction Layer Packets whose target can be addressed in 32 bits or whose target is identified by Bus/Device/Function/Register rather than by an MMIO address. It is selected by `Fmt[2:0] = 000` (no data) or `010` (with data) and applies to Memory Requests with 32-bit MMIO addresses, all I/O Requests, all Configuration Requests, and all Completions. The header is followed by a payload only when `Fmt[2] = 1`.

```
Generic 3DW TLP Header
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ Fmt │  Type   │R│ TC  │R│R│N│H│D│E│Att│AT │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           └───────────────────────────────────────────────────────────┴─┴──┘
```

## SUMMARY

Every TLP starts with a one-DWord `DW 0` whose layout is identical across all TLP types and which encodes the packet class (`Fmt` selects header length and whether a data payload is attached, `Type` selects the subtype), the traffic class, the error/digest/attribute bits, and the payload length in DWords. The 3DW header adds two more DWords. `DW 1` carries the Requester ID, the Tag value that identifies an outstanding non-posted request, and First/Last DW Byte Enables that select valid bytes inside the first and last payload DWord. `DW 2` carries 30 bits of DW-aligned address; the low two bits are Reserved because PCIe addresses payload at DWord granularity and the per-byte selection is delegated to FBE/LBE.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2: Packet Format
- PCI Express Base Specification, section 2.2.1: Common Packet Header Fields
- PCI Express Base Specification, section 2.2.2: TLPs with Data Payloads
- PCI Express Base Specification, section 2.2.4: TLP Digest Rules
- PCI Express Base Specification, section 2.2.5: TLP Prefix Rules
- PCI Express Base Specification, section 2.2.6.1: Length Field Encoding
- PCI Express Base Specification, section 2.2.6.4: Traffic Class (TC) Field
- PCI Express Base Specification, section 2.2.7: First/Last DW Byte Enables Rules
- PCI Express Base Specification, section 6.2.7: AER TLP Header Log
- PCI Express Base Specification, section 7.8.4: Advanced Error Reporting Extended Capability

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [LWN, "An AER and DPC update for 5.18"](https://lwn.net/Articles/881687/)

## FIELDS

The 3DW header occupies 12 bytes on the wire. Each DWord is transmitted MSB-first per byte and byte 0 first per DWord.

### DW 0 fields

```
       byte 0                          byte 1                          byte 2                          byte 3
       bit:  7 6 5  4 3 2 1 0           7  6 5 4  3 2 1 0           7 6 5 4 3 2 1 0           7 6 5 4 3 2 1 0
            ┌─────┬─────────┐          ┌─┬─────┬───────┐            ┌─┬─┬───┬─┬─┐             ┌───────────────┐
            │ Fmt │  Type   │          │R│ TC  │   R   │            │T│E│Att│R│R│             │   Length      │
            └─────┴─────────┘          └─┴─────┴───────┘            └─┴─┴───┴─┴─┘             └───────────────┘

       Fmt[2:0]    = 000  3DW, no payload
                     010  3DW, with payload
                     (001/011 = 4DW header variants)
                     100  = TLP Prefix
       Type[4:0]   = 00000 MRd / MWr  (memory request)
                     00010 IORd / IOWr
                     00100 CfgRd0 / CfgWr0
                     00101 CfgRd1 / CfgWr1
                     01010 Cpl / CplD
                     01100 FetchAdd
                     01101 Swap
                     01110 CAS
       TC[2:0]     traffic class (0..7); selects the Virtual Channel for ordering / QoS
       TD          TLP Digest present (DW of ECRC appended after payload)
       EP          Poisoned data
       Attr[1:0]   {Relaxed Ordering, No Snoop}
       Length[9:0] payload size in DWords, 0 == 1024 DW (Memory Read only)
```

### DW 1 fields

```
       byte 4                          byte 5                          byte 6                       byte 7
       ┌───────────────────────────────────────────────┐                ┌───────────────┐  ┌───────┬────────┐
       │              Requester ID                     │                │      Tag      │  │  LBE  │  FBE   │
       └───────────────────────────────────────────────┘                └───────────────┘  └───────┴────────┘

       Requester ID = {Bus[7:0], Device[4:0], Function[2:0]}
       Tag          = Requester-assigned identifier for this non-posted request.
                      Completer copies it into every Cpl/CplD so the Requester can
                      reassemble responses and match them to outstanding requests.
       FBE[3:0]     = First DW Byte Enables. Bit n=1 means byte n of the first
                      payload DW is valid. All four bits 1 means a full DW.
       LBE[3:0]     = Last DW Byte Enables. Must be 0000b when Length=1 (single DW).
```

### DW 2 fields

```
       byte 8                          byte 9                          byte 10                         byte 11
       ┌───────────────────────────────────────────────────────────────────────────────────────────┬───────┐
       │                                    Address [31:2]                                         │   R   │
       └───────────────────────────────────────────────────────────────────────────────────────────┴───────┘

       Address[31:2] = DW-aligned 32-bit address (for memory / I/O requests).
                       Configuration Requests reinterpret this DWord as
                       {Bus, Device, Function, R, Extended Register, Register, R}.
                       Completions reinterpret it as
                       {Requester ID, Tag, R, Lower Address[6:0]}.
       Address[1:0]  = 0b00 (Reserved). Byte-level selection is via FBE/LBE.
```

## DETAILS

