---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# ERR_FATAL

ERR_FATAL is a 4DW Msg (no data) routed To Root Complex. It reports an uncorrectable error that compromises link integrity (the link can no longer reliably carry TLPs), such as Data Link Protocol Error, Surprise Down, or Flow Control Protocol Error. The Message Code is `0x33` and the Type encodes `10000` (To Root Complex).

```
ERR_FATAL  (Routing: To RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10000  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00110011    │
           ├───────────────────────────────┴───────────────┴────────────────┤
DW 2       │                            Reserved                            │
           ├────────────────────────────────────────────────────────────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

ERR_FATAL signals that the link is no longer trustworthy. Beyond simply logging the offending TLP, the kernel forces a recovery flow that isolates the affected subtree, performs a slot reset, and asks each driver below the failing point to re-initialise. The platform may also trigger DPC (Downstream Port Containment) to quarantine the link automatically. Common triggers are Surprise Down (cable yanked), Data Link Protocol Error (corrupt DLLP state), Flow Control Protocol Error (credit accounting violation), and link training failures.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.2: PCI Express Advanced Error Reporting
- PCI Express Base Specification, section 6.2.3.2: Uncorrectable Error Severity
- PCI Express Base Specification, section 6.2.10: Downstream Port Containment
- PCI Express Base Specification, section 7.8.4: Advanced Error Reporting Extended Capability
- PCI Express Base Specification, section 7.8.5: Downstream Port Containment Extended Capability

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10000` (To RC) |
| Length[9:0] | `0` |
| Message Code | `0x33` |
| Requester ID[15:0] | the function reporting the error |
| Tag[7:0] | Reserved |
| DW 2 / DW 3 | Reserved |

## DETAILS

