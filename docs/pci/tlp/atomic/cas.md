---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# AtomicOp CAS (Compare and Swap)

CAS is a non-posted PCIe AtomicOp that conditionally swaps a DWord (or QWord) in MMIO space based on a comparison. The Completer reads the current value at the target DW-aligned MMIO address, compares it to the CompareValue from the TLP payload, and (only if equal) writes the SwapValue from the payload. The original value is always returned in CplD, and the Requester compares it to its expected CompareValue to determine whether the swap occurred. The 3DW variant uses a 32-bit MMIO address with 32-bit operands and Length = 2 DW (1 DW compare plus 1 DW swap); the 4DW variant uses a 64-bit MMIO address with either 32-bit or 64-bit operands. `Type[4:0] = 01110` identifies CAS.

```
AtomicOp CAS, 3DW Header (32-bit address, 32-bit operands)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  01110  │R│ TC  │R│R│0│0│D│E│Att│AT │     0000000010     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 3       │                     Compare Value (32-bit)                     │
           ├────────────────────────────────────────────────────────────────┤
DW 4       │                      Swap Value (32-bit)                       │
           └────────────────────────────────────────────────────────────────┘
```

```
AtomicOp CAS, 4DW Header (64-bit address, 64-bit operands)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  01110  │R│ TC  │R│R│0│0│D│E│Att│AT │     0000000100     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴────────┤
DW 2       │                        Address [63:32]                         │
           ├───────────────────────────────────────────────────────────┬─┬──┤
DW 3       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 4       │                      Compare Value [31:0]                      │
           ├────────────────────────────────────────────────────────────────┤
DW 5       │                     Compare Value [63:32]                      │
           ├────────────────────────────────────────────────────────────────┤
DW 6       │                       Swap Value [31:0]                        │
           ├────────────────────────────────────────────────────────────────┤
DW 7       │                       Swap Value [63:32]                       │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

CAS is the foundation of lock-free programming on PCIe. The Completer atomically reads the target, compares to CompareValue, and (only if equal) writes SwapValue. The pre-operation value is always returned in CplD; the Requester compares it to CompareValue to detect success (returned == CompareValue ⇒ swap happened) or failure (returned ≠ CompareValue ⇒ swap did not happen, retry with new CompareValue). CAS payload size is the largest of the AtomicOps: 2 DW for 32-bit operands (1 DW compare + 1 DW swap), 4 DW for 64-bit operands (2 DW compare + 2 DW swap).

## SPECIFICATIONS

- PCI Express Base Specification, section 6.15: Atomic Operations
- PCI Express Base Specification, section 6.15.5: Compare and Swap
- PCI Express Base Specification, section 2.2.6.7: AtomicOp Request Type Encodings
- PCI Express Base Specification, section 2.2.8.4: Memory Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) or `011` (4DW with data) |
| Type[4:0] | `01110` |
| Length[9:0] | `2` (32-bit operands) or `4` (64-bit operands) |
| TC[2:0] | typically `0` |
| Attr[1:0] | typically `00` |
| Tag[7:0] | Requester-assigned for matching the CplD |
| FBE / LBE | byte enables for the operands |
| Address | 4-byte or 8-byte aligned |
| Compare Value | 1 DW or 2 DW |
| Swap Value | 1 DW or 2 DW |

## DETAILS

