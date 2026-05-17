---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Generic 4DW TLP Header

The 4DW (four-DWord) TLP header is the 16-byte common header used by Transaction Layer Packets whose target needs a 64-bit MMIO address, by all Message TLPs, and by 64-bit AtomicOps. It is selected by `Fmt[2:0] = 001` (no data) or `011` (with data). The extra DWord versus the 3DW form holds the high 32 bits of the target address (Address[63:32]).

```
Generic 4DW TLP Header
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ Fmt │  Type   │R│ TC  │R│R│N│H│D│E│Att│AT │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴────────┤
DW 2       │                        Address [63:32]                         │
           ├───────────────────────────────────────────────────────────┬─┬──┤
DW 3       │                      Address [31:2]                       │R│R │
           └───────────────────────────────────────────────────────────┴─┴──┘
```

## SUMMARY

The 4DW header shares `DW 0` and `DW 1` with the 3DW form. It diverges at `DW 2`, where a 3DW header places the 32-bit address, by instead splitting the 64-bit address across `DW 2` (high bits 63:32) and `DW 3` (low bits 31:2 plus two Reserved bits). The format selection lives in `Fmt[2:0]`. Memory operations target a 64-bit address when the destination range crosses 4 GiB; Messages always use this format regardless of payload, with `DW 2` and `DW 3` repurposed as Message-specific routing or data words; AtomicOps use the 4DW form for 64-bit address or 64-bit operand variants.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.1: Common Packet Header Fields
- PCI Express Base Specification, section 2.2.4: TLP Digest Rules
- PCI Express Base Specification, section 2.2.6.1: Length Field Encoding
- PCI Express Base Specification, section 2.2.6.3: Memory Request Type Encodings
- PCI Express Base Specification, section 2.2.8: Transaction Descriptor
- PCI Express Base Specification, section 2.2.8.4: Memory Request Header Format (64-bit)
- PCI Express Base Specification, section 2.2.9: Message Request Rules
- PCI Express Base Specification, section 6.2.7: AER TLP Header Log
- PCI Express Base Specification, section 7.8.4: Advanced Error Reporting Extended Capability

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)
- [LWN, "An AER and DPC update for 5.18"](https://lwn.net/Articles/881687/)

## FIELDS

The 4DW header occupies 16 bytes on the wire. Each DWord is transmitted MSB-first per byte and byte 0 first per DWord.

### DW 0 fields

```
       byte 0                          byte 1                          byte 2                          byte 3
       bit:  7 6 5  4 3 2 1 0           7  6 5 4  3 2 1 0           7 6 5 4 3 2 1 0           7 6 5 4 3 2 1 0
            ┌─────┬─────────┐          ┌─┬─────┬───────┐            ┌─┬─┬───┬─┬─┐             ┌───────────────┐
            │ Fmt │  Type   │          │R│ TC  │   R   │            │T│E│Att│R│R│             │   Length      │
            └─────┴─────────┘          └─┴─────┴───────┘            └─┴─┴───┴─┴─┘             └───────────────┘

       Fmt[2:0]    = 001  4DW, no payload
                     011  4DW, with payload
                     (000/010 = 3DW header variants)
                     100  = TLP Prefix
       Type[4:0]   = 00000 MRd / MWr  (64-bit memory request)
                     01100 FetchAdd   (4DW AtomicOp variant)
                     01101 Swap       (4DW AtomicOp variant)
                     01110 CAS        (4DW AtomicOp variant)
                     10rrr Msg / MsgD (Messages are always 4DW; rrr = routing)
       TC[2:0]     traffic class (0..7); selects the Virtual Channel for ordering / QoS
       TD          TLP Digest present (DW of ECRC appended after payload)
       EP          Poisoned data
       Attr[1:0]   {Relaxed Ordering, No Snoop}
       Length[9:0] payload size in DWords; 0 == 1024 DW for Memory Read; 0 for
                   no-data TLPs (Msg, MRd)
```

### DW 1 fields

Same layout as the 3DW form.

```
       ┌───────────────────────────────────────────────┐ ┌───────────────┐ ┌───────┬────────┐
       │              Requester ID                     │ │      Tag      │ │  LBE  │  FBE   │
       └───────────────────────────────────────────────┘ └───────────────┘ └───────┴────────┘

       Requester ID = {Bus[7:0], Device[4:0], Function[2:0]}
       Tag          = Requester-assigned identifier for outstanding non-posted
                      requests; Messages reuse this byte as the upper half of the
                      Tag/Code split (the Message Code occupies the LSBs).
       FBE / LBE    = First / Last DW Byte Enables for memory and AtomicOps.
                      For Messages these nibbles are repurposed:
                      DW 1 byte 7 (the FBE position) holds the Message Code.
```

For Messages the rightmost byte of DW 1 (where FBE/LBE live for memory requests) holds the 8-bit Message Code that identifies the specific Message (INTx assert/deassert, PM_PME, ERR_COR, LTR, FRS, etc.).

### DW 2 fields

```
       byte 8                          byte 9                          byte 10                         byte 11
       ┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
       │                                       Address [63:32]                                               │
       └─────────────────────────────────────────────────────────────────────────────────────────────────────┘

       For Memory Requests / AtomicOps:
         Address[63:32] = high half of the 64-bit DW-aligned target address.

       For Messages (Routing-dependent):
         "Routed by Address" : DW 2 carries the high 32 bits of the target address.
         "Routed by ID"      : DW 2 high 16 bits hold the Destination ID
                                (Bus/Device/Function); low 16 bits are
                                Message-Code-specific.
         "To Root Complex"   : DW 2 is Reserved or Message-specific (e.g. LTR latency).
         "Broadcast from RC" : DW 2 is Reserved or Message-specific.
         "Local"             : DW 2 is Reserved or Message-specific.
```

### DW 3 fields

```
       byte 12                         byte 13                         byte 14                         byte 15
       ┌─────────────────────────────────────────────────────────────────────────────────────────┬───────┐
       │                                  Address [31:2]                                         │   R   │
       └─────────────────────────────────────────────────────────────────────────────────────────┴───────┘

       For Memory Requests / AtomicOps:
         Address[31:2] = low DW-aligned half of the target address.
         Address[1:0]  = 0b00 (Reserved).

       For Messages routed by Address:
         Same Address[31:2] semantics.
       For Messages with other routing:
         DW 3 is Reserved or carries Message-specific data (Set_Slot_Power_Limit
         power-limit DWord, PTM master time low, Vendor-Defined header bytes).
```

## DETAILS

