---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Memory Read Request (MRd)

A Memory Read Request, usually abbreviated as MRd, is one of the standard Transaction Layer Packets (TLPs) defined by the PCI Express Base Specification. It is used whenever an agent on the PCIe fabric needs to read one or more DWords of data from a location in the PCIe Memory Address Space. The kernel and most driver writers usually refer to this address space as MMIO space, because from the CPU's point of view a load instruction issued against a virtual address backed by a Function's Base Address Register (BAR) is precisely what causes the host bridge to synthesise the MRd and place it on the wire. A Memory Read Request issued by an Endpoint, on the other hand, normally targets system memory advertised behind the Root Complex and is what driver developers usually call a DMA read.

The Memory Read Request is a non-posted transaction, which means that the Requester is supposed to wait for the Completer to send back a response carrying the requested bytes before considering the read complete. To make this matching possible, the Requester chooses a Tag value when it issues the request and writes that Tag together with its own Requester ID into the header. The Completer copies these two fields verbatim into every Completion it sends in response, so the Requester can demultiplex incoming responses without having to scan a list of outstanding requests. The number of DWords the Completer is supposed to return is given in the `Length` field, and two byte-enable nibbles called First DW Byte Enables (FBE) and Last DW Byte Enables (LBE) tell the Completer which bytes inside the first and last DWord of the response are meaningful. If the request succeeds, the response comes back as one or more Completion-with-Data (CplD) TLPs; if the address is unclaimed, the device aborts, or a timeout fires, the response comes back as a Completion-without-Data (Cpl) whose Completion Status (`CS`) field encodes the failure.

```
PCIe Memory Read Request (MRd) TLP, 3DW Header
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 000 │  00000  │R│ TC  │R│R│N│H│D│E│Att│AT │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           └───────────────────────────────────────────────────────────┴─┴──┘
```

```
PCIe Memory Read Request (MRd) TLP, 4DW Header
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  00000  │R│ TC  │R│R│N│H│D│E│Att│AT │       Length       │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴────────┤
DW 2       │                        Address [63:32]                         │
           ├───────────────────────────────────────────────────────────┬─┬──┤
DW 3       │                      Address [31:2]                       │R│R │
           └───────────────────────────────────────────────────────────┴─┴──┘
```

## SUMMARY

A Memory Read Request travels from the Requester to the Completer that owns the target address. The Requester is usually a CPU executing an MMIO load against a BAR mapping (in which case the host bridge synthesises the MRd) or an Endpoint performing a DMA read against system memory (in which case the Endpoint's own controller emits the TLP upstream toward the Root Complex). On its way through the fabric the TLP is routed strictly by address. Every Switch downstream port compares the address against its Memory Base / Limit window and forwards the TLP toward the matching child bus, and the Root Complex performs the same check before letting the request leave the host bridge.

The Memory Read Request itself carries no payload of its own. The data that the Requester is asking for comes back later as one or more Completion-with-Data (CplD) TLPs, each carrying a fragment of the bytes requested. The Completer is permitted to break the response into multiple CplD TLPs aligned to its Read Completion Boundary (RCB), which is typically either 64 or 128 bytes on modern systems. The Requester is supposed to reassemble the data implicitly by matching the Tag in every CplD back to the originating MRd and by using the Lower Address field in each CplD to compute the destination offset.

If the target address is unclaimed, the device aborts, or a Completion Timeout fires, the response comes back as a Completion-without-Data (Cpl) whose Completion Status (`CS`) field encodes the failure code (Unsupported Request, Completer Abort, Configuration Request Retry Status, or Completion Timeout). In that case the Requester is expected to record the event through the Advanced Error Reporting (AER) capability so that the operating system can decide whether to recover or to report the failure to userspace.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.3: Memory Request Type Encodings
- PCI Express Base Specification, section 2.2.8.4: Memory Request Header Format
- PCI Express Base Specification, section 2.2.9: Request and Completion Rules
- PCI Express Base Specification, section 2.3: Handling of Received TLPs
- PCI Express Base Specification, section 2.3.1.1: Data Return for Read Requests
- PCI Express Base Specification, section 2.4.5: Read Request / Completion Ordering
- PCI Express Base Specification, section 7.5.3.4: Device Control Register (Max_Read_Request_Size)
- PCI Express Base Specification, section 7.5.3.12: Device Capabilities Register (Read Completion Boundary)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `000` (3DW) or `001` (4DW); no payload |
| Type[4:0] | `00000` |
| Length[9:0] | DW count of the read (0 means 1024 DW = 4 KiB) |
| TC[2:0] | Traffic Class (selects Virtual Channel) |
| Attr[1:0] | {Relaxed Ordering, No Snoop} |
| Tag[7:0] | Requester-assigned ID for matching the CplD |
| FBE[3:0] | First DW Byte Enables |
| LBE[3:0] | Last DW Byte Enables (zero if Length=1) |
| Address[31:2] (3DW) | 32-bit DW-aligned target |
| Address[63:2] (4DW) | 64-bit DW-aligned target across DW 2/3 |

The 3DW variant is used when `Address[63:32] == 0`. The 4DW variant carries the high half in DW 2 and the low half in DW 3.

## DETAILS

