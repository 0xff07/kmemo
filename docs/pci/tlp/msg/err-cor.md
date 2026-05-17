---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# ERR_COR

ERR_COR is a 4DW Msg (no data) routed To Root Complex. It reports a correctable error from a Function or Switch, meaning an error that PCIe hardware was able to recover from automatically (e.g. a Bad TLP that the Data Link Layer's retry buffer replayed successfully, a Bad DLLP, or a Replay Timer Timeout that did not result in lost data). The Message Code is `0x30` and the Type encodes `10000` (To Root Complex).

```
ERR_COR  (Routing: To RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00110000    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

ERR_COR reports an error that did not corrupt data delivery. The reporting Function's Requester ID identifies who detected the error. The Root Complex receives the Message, logs it in AER, and may signal an interrupt if AER masks permit. ERR_COR is the lowest-severity error class; OS-level handling increments a counter and may surface the event to userspace through `dmesg` or `/sys/bus/pci/devices/*/aer_dev_correctable`.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.2: PCI Express Advanced Error Reporting
- PCI Express Base Specification, section 6.2.3: Error Signaling Mechanisms
- PCI Express Base Specification, section 6.2.6: Error Logging
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
| Message Code | `0x30` |
| Requester ID[15:0] | the function reporting the error |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

