---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# ERR_NONFATAL

ERR_NONFATAL is a 4DW Msg (no data) routed To Root Complex. It reports an uncorrectable error that did not compromise link integrity but did corrupt one or more transactions (e.g. a Poisoned TLP received and consumed, a Completer Abort, a Completion Timeout, an Unsupported Request). The Message Code is `0x31` and the Type encodes `10000` (To Root Complex).

```
ERR_NONFATAL  (Routing: To RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00110001    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

ERR_NONFATAL announces that an individual transaction was lost or corrupted but the PCIe link itself remains usable. AER captures the offending TLP header in `PCI_ERR_HEADER_LOG` so the OS can identify the failing request and the affected driver. Common triggers are Unsupported Request (target address is unmapped), Completer Abort, Completion Timeout, Poisoned TLP receipt, and ECRC mismatch. The kernel's AER service may invoke the driver's error-recovery callback (`error_detected`, `mmio_enabled`, `slot_reset`, `resume`) when the device supports `pci_error_handlers`.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.2: PCI Express Advanced Error Reporting
- PCI Express Base Specification, section 6.2.3.2: Uncorrectable Error Severity
- PCI Express Base Specification, section 6.2.7: AER TLP Header Log
- PCI Express Base Specification, section 7.8.4: Advanced Error Reporting Extended Capability
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10000` (To RC) |
| Length[9:0] | `0` |
| Message Code | `0x31` |
| Requester ID[15:0] | the function reporting the error |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

