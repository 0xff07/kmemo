---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# Message without Data (Msg)

A Message without Data (Msg) is a 4DW TLP that carries no payload and is used for in-band signalling that replaces PCI sideband signals (INTx assertions, power-management events, error reports, hot-plug, vendor-defined notifications). Every Message uses a 4DW header regardless of whether it actually needs the extra address DWords. The 8-bit Message Code field in DW 1 byte 7 selects the specific Message; the Type field's low three bits encode the routing sub-field.

```
PCIe Message without Data (Msg) TLP
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10rrr  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │  Message Code  │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                  Message-specific (see notes)                  │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                  Message-specific (see notes)                  │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

Messages are PCIe's substitute for PCI sideband wires. Type[4:0] is encoded as `10rrr`, where `rrr` is a 3-bit routing sub-field. The encoding `000` is To Root Complex, `001` is Routed by Address, `010` is Routed by ID, `011` is Broadcast from Root Complex, `100` is Local (terminate at receiver), and `101` is Gathered & routed to Root Complex. The Message Code in DW 1 byte 7 selects the specific Message (INTx, PM_PME, ERR_COR, LTR, FRS, Unlock, etc.). DW 2 / DW 3 carry routing-specific data, which is a 64-bit address for "Routed by Address", a Destination ID for "Routed by ID", or Reserved zeros for the simpler routings.

## SPECIFICATIONS

- PCI Express Base Specification, section 2.2.6.6: Message Request Type Encodings
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format
- PCI Express Base Specification, section 2.2.9: Routing Sub-field Encoding
- PCI Express Base Specification, section 6.1: Interrupt and PME Support
- PCI Express Base Specification, section 6.2: PCI Express Advanced Error Reporting
- PCI Express Base Specification, section 6.7: Hot-Plug Support

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no payload) |
| Type[4:0] | `10rrr` (rrr = routing sub-field) |
| Length[9:0] | `0` (no data) |
| TC[2:0] | typically `0` |
| Attr[1:0] | typically `00` |
| Tag[7:0] | per-Message semantics (often reserved or carrying upper Tag bits for vendor protocols) |
| Message Code[7:0] | identifies the specific Message |
| DW 2, DW 3 | Message-specific (Routing-dependent) |

### Routing sub-field

| `rrr` | Routing | Notes |
|---|---|---|
| 000 | To Root Complex | INTx, errors, PME, LTR, FRS |
| 001 | Routed by Address | DW 2 / DW 3 carry a 64-bit address |
| 010 | Routed by ID | DW 2 high 16 bits = Destination BDF |
| 011 | Broadcast from RC | Hot-plug, PME_Turn_Off, Unlock |
| 100 | Local | ASPM NAK, PTM Request |
| 101 | Gathered & routed to RC | PME_TO_Ack |

## DETAILS

