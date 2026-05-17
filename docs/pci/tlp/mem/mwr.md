---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Memory Write Request (MWr)

A Memory Write Request, usually abbreviated as MWr, is one of the standard Transaction Layer Packets (TLPs) defined by the PCI Express Base Specification. It is used whenever an agent on the PCIe fabric needs to write a contiguous block of DWord-aligned data into a location in the PCIe Memory Address Space. As with the Memory Read Request, the kernel and most driver writers usually refer to that address space as MMIO space, because from the CPU's point of view a store instruction issued against a virtual address backed by a Function's Base Address Register (BAR) is what causes the host bridge to synthesise the MWr and place it on the wire. An MWr issued by an Endpoint, on the other hand, normally targets system memory advertised behind the Root Complex and is what driver writers usually call a DMA write.

The Memory Write Request is a posted transaction, which means that the Requester is supposed to treat the transaction as complete the moment its Transaction Layer has accepted the TLP for transmission and is not supposed to wait for any kind of acknowledgement from the Completer. No Completion (Cpl or CplD) TLP is ever returned for a successful MWr; in fact, the MWr is the only Memory Request type that does not generate a Completion at all. If something does go wrong at the Completer (for example, the address turns out to be unclaimed, or the payload is poisoned), the Completer is supposed to report the failure asynchronously by emitting an ERR_COR, ERR_NONFATAL, or ERR_FATAL Message TLP back to the Root Complex; the originating Requester only learns about the failure indirectly, by way of the Advanced Error Reporting (AER) capability.

```
PCIe Memory Write Request (MWr) TLP, 3DW Header
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  00000  │R│ TC  │R│R│N│H│D│E│Att│AT │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 3..N    │                     Payload Data (DWords)                      │
           │                              ...                               │
           └────────────────────────────────────────────────────────────────┘
```

```
PCIe Memory Write Request (MWr) TLP, 4DW Header
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  00000  │R│ TC  │R│R│N│H│D│E│Att│AT │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴────────┤
DW 2       │                        Address [63:32]                         │
           ├───────────────────────────────────────────────────────────┬─┬──┤
DW 3       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 4..N    │                     Payload Data (DWords)                      │
           │                              ...                               │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

A Memory Write Request travels from the Requester to the Completer that owns the target address. The Requester is usually a CPU executing an MMIO store against a BAR mapping (in which case the host bridge synthesises the MWr) or an Endpoint performing a DMA write into system memory (in which case the Endpoint's own controller emits the TLP upstream toward the Root Complex). On its way through the fabric, the TLP is routed strictly by address using the same rules as a Memory Read Request: every Switch downstream port compares the address against its Memory Base / Limit window and forwards the TLP toward the matching child bus.

The payload follows the header as a sequence of one or more DWords. The `Length` field counts the number of payload DWords, and the First and Last DW Byte Enables (`FBE` and `LBE`) select which bytes inside the first and last payload DWord are actually meaningful. Because the MWr is posted, the Requester is allowed to consider its transaction complete the moment its Transaction Layer has accepted the TLP for transmission. The Completer commits the write asynchronously, on its own schedule, and there is no Completion-class response of any kind to confirm that the write actually landed.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.3: Memory Request Type Encodings
- PCI Express Base Specification, section 2.2.8.4: Memory Request Header Format
- PCI Express Base Specification, section 2.2.9: Request and Completion Rules
- PCI Express Base Specification, section 2.4: Transaction Ordering
- PCI Express Base Specification, section 2.4.1: Transaction Ordering Rules (Producer-Consumer)
- PCI Express Base Specification, section 2.5: Virtual Channel (VC) Mechanism
- PCI Express Base Specification, section 7.5.3.4: Device Control Register (Max_Payload_Size)
- PCI Express Base Specification, section 6.2.3: Posted Request Error Handling

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW) or `011` (4DW); payload follows |
| Type[4:0] | `00000` |
| Length[9:0] | DW count of payload (bounded by MPS) |
| TC[2:0] | Traffic Class |
| Attr[1:0] | {Relaxed Ordering, No Snoop} |
| Tag[7:0] | Requester-assigned (Tag is logged but unused for matching since no Cpl returns) |
| FBE[3:0] | First DW Byte Enables |
| LBE[3:0] | Last DW Byte Enables (zero when Length=1) |
| Address[31:2] (3DW) | 32-bit DW-aligned target |
| Address[63:2] (4DW) | 64-bit DW-aligned target |
| Payload | `Length` DWords following the header |

## DETAILS

