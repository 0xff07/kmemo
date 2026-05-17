---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# INTx Deassert Messages (Deassert_INT{A,B,C,D})

The four Deassert_INT Messages emulate the de-assertion edges of PCI's INTA#, INTB#, INTC#, and INTD# interrupt wires. Each is a 4DW Msg (no data) routed To Root Complex, with Message Code `0x24` (INTA), `0x25` (INTB), `0x26` (INTC), or `0x27` (INTD). A Function emits Deassert exactly when its previously-asserted INTx line transitions back to "not asserted", after the driver's ISR clears the function's internal interrupt status register.

```
INTx Deassert_INTA  (Routing: To Root Complex)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00100100    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

```
INTx Deassert_INTB  (Routing: To Root Complex)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00100101    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

```
INTx Deassert_INTC  (Routing: To Root Complex)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00100110    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

```
INTx Deassert_INTD  (Routing: To Root Complex)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00100111    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

Deassert messages close the level-triggered interrupt protocol Assert started. After the ISR has read the function's status register and cleared the interrupt source, the function's hardware emits the matching Deassert. Switches decrement their per-lane assert counter; when the counter reaches zero on the upstream side, the Switch emits a Deassert upstream so the next Switch (or the Root Complex) can drop the virtual line. The Root Complex translates the line drop into deassertion of the corresponding IOAPIC pin (or equivalent). This faithfully reproduces PCI's level-triggered behaviour over the message-passing PCIe fabric.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCI Express Base Specification, section 6.1.1: Legacy INTx Support
- PCI Local Bus Specification, Revision 3.0, section 2.2.6 (INTx# definitions inherited by PCIe)

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10000` (Routing: To Root Complex) |
| Length[9:0] | `0` |
| Message Code | `0x24` for INTA, `0x25` for INTB, `0x26` for INTC, `0x27` for INTD |
| Requester ID[15:0] | the function deasserting INTx |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved (all zero) |

## DETAILS

