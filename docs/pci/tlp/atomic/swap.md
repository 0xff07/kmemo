---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# AtomicOp Swap

Swap is a non-posted PCIe AtomicOp that atomically performs `tmp = *addr; *addr = SwapValue; return tmp` against a DW-aligned target in MMIO space. The Completer reads the current value at the target MMIO address, replaces it with the SwapValue from the TLP payload, and returns the original value in a CplD. The 3DW variant uses a 32-bit MMIO address with a 32-bit operand (Length = 1); the 4DW variant uses a 64-bit MMIO address with either a 32-bit or 64-bit operand. `Type[4:0] = 01101` identifies Swap.

```
AtomicOp Swap, 3DW Header (32-bit address, 32-bit operand)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  01101  │R│ TC  │R│R│0│0│D│E│Att│AT │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 3       │                  Swap Value (32-bit operand)                   │
           └────────────────────────────────────────────────────────────────┘
```

```
AtomicOp Swap, 4DW Header (64-bit address, 64-bit operand)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  01101  │R│ TC  │R│R│0│0│D│E│Att│AT │     0000000010     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴────────┤
DW 2       │                        Address [63:32]                         │
           ├───────────────────────────────────────────────────────────┬─┬──┤
DW 3       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 4       │                       Swap Value [31:0]                        │
           ├────────────────────────────────────────────────────────────────┤
DW 5       │                       Swap Value [63:32]                       │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

Swap is the unconditional atomic exchange. It always replaces the value at the target MMIO address with the SwapValue and returns the original value. Compare with CAS, which performs a conditional swap based on a CompareValue. Swap's `Type[4:0] = 01101`; the operand encoding follows the same 1-DW / 2-DW rule as FetchAdd. Alignment is 4 bytes for 32-bit operands, 8 bytes for 64-bit operands; misaligned Swaps generate Malformed TLP.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.15: Atomic Operations
- PCI Express Base Specification, section 6.15.4: Unconditional Swap
- PCI Express Base Specification, section 2.2.6.7: AtomicOp Request Type Encodings
- PCI Express Base Specification, section 2.2.8.4: Memory Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) or `011` (4DW with data) |
| Type[4:0] | `01101` |
| Length[9:0] | `1` (32-bit operand) or `2` (64-bit operand) |
| TC[2:0] | typically `0` |
| Attr[1:0] | typically `00` |
| Tag[7:0] | Requester-assigned for matching the CplD |
| FBE / LBE | byte enables for the operand |
| Address | 4-byte or 8-byte aligned |
| Swap Value | 1 DW or 2 DW |

## DETAILS

