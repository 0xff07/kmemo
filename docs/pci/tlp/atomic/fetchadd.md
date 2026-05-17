---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# AtomicOp FetchAdd

FetchAdd is a non-posted PCIe AtomicOp that atomically performs `tmp = *addr; *addr = tmp + AddValue; return tmp` against a DW-aligned target in MMIO space. The pre-operation value is returned to the Requester in a CplD, and the post-operation value is committed by the Completer to the same MMIO address. The 3DW variant uses a 32-bit MMIO address with a 32-bit operand (Length = 1 DW); the 4DW variant uses a 64-bit MMIO address with either a 32-bit or 64-bit operand (Length = 2 DW for 64-bit). `Type[4:0] = 01100` identifies FetchAdd.

```
AtomicOp FetchAdd, 3DW Header (32-bit address, 32-bit operand)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 010 │  01100  │R│ TC  │R│R│0│0│D│E│Att│AT │     0000000001     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴───┬─┬──┤
DW 2       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 3       │                   Add Value (32-bit operand)                   │
           └────────────────────────────────────────────────────────────────┘
```

```
AtomicOp FetchAdd, 4DW Header (64-bit address, 64-bit operand)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 011 │  01100  │R│ TC  │R│R│0│0│D│E│Att│AT │     0000000010     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬───────┬────────┤
DW 1       │         Requester ID          │      Tag      │  LBE  │  FBE   │
           ├───────────────────────────────┴───────────────┴───────┴────────┤
DW 2       │                        Address [63:32]                         │
           ├───────────────────────────────────────────────────────────┬─┬──┤
DW 3       │                      Address [31:2]                       │R│R │
           ├───────────────────────────────────────────────────────────┴─┴──┤
DW 4       │                        Add Value [31:0]                        │
           ├────────────────────────────────────────────────────────────────┤
DW 5       │                       Add Value [63:32]                        │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

PCIe AtomicOps let a Requester perform atomic read-modify-write against memory in a single non-posted TLP. The Completer atomically reads the current value, applies the operation, writes the new value, and returns the original value in CplD. FetchAdd specifically computes `original + AddValue` and stores it, returning `original`. The 32-bit operand variant is 3DW header with 1 DW payload (the Add Value); the 64-bit operand variant is 4DW header with 2 DW payload (Add Value low + high). Alignment is 4 bytes for 32-bit operands and 8 bytes for 64-bit operands; misaligned AtomicOps generate Malformed TLP.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.15: Atomic Operations
- PCI Express Base Specification, section 6.15.3: FetchAdd
- PCI Express Base Specification, section 2.2.6.7: AtomicOp Request Type Encodings
- PCI Express Base Specification, section 2.2.8.4: Memory Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `010` (3DW with data) or `011` (4DW with data) |
| Type[4:0] | `01100` |
| Length[9:0] | `1` (32-bit operand) or `2` (64-bit operand) |
| TC[2:0] | typically `0` |
| Attr[1:0] | typically `00` |
| Tag[7:0] | Requester-assigned for matching the CplD |
| FBE / LBE | byte enables for the operand |
| Address | 4-byte (32-bit operand) or 8-byte (64-bit operand) aligned |
| Add Value | 1 DW or 2 DW operand |

## DETAILS

