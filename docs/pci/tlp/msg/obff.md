---
topics: pcie
tags:
    - "pcie"
    - "verification-needed"
---

# OBFF (Optimized Buffer Flush/Fill)

OBFF is a 4DW Msg (no data) Broadcast from Root Complex. It tells every downstream Endpoint what the current platform state is, so the Endpoint can align bursty traffic with periods when the CPU and memory subsystem are already awake. The Message Code is `0x12` and the Type encodes `10011` (Broadcast from RC). The 4-bit OBFF Code in DW 2 byte 3 conveys the current state.

```
OBFF (Optimized Buffer Flush/Fill)  (Routing: Broadcast from RC)
═════════════════════════════════════════════════════════════════════════════════

Byte Offset:     +0              +1              +2              +3
Bit:         7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0 7 6 5 4 3 2 1 0
           ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
DW 0       │ 001 │  10011  │R│ TC  │R│R│0│0│D│E│Att│00 │     0000000000     │
           ├─────┴─────────┴─┴─────┴─┴─┴─┴─┼─┴─┴───┴───┴───┬────────────────┤
DW 1       │         Requester ID          │      Tag      │    00010010    │
           ├───────────────────────────────┴───────────────┴───────┬────────┤
DW 2       │                       Reserved                        │OBFF Cod│
           ├───────────────────────────────────────────────────────┴────────┤
DW 3       │                            Reserved                            │
           └────────────────────────────────────────────────────────────────┘
```

## SUMMARY

OBFF is the platform's broadcast hint to all downstream Endpoints: "right now, the CPU is active" or "right now, only memory and I/O are active" or "right now, everything is idle, please defer non-urgent traffic". Endpoints that participate in OBFF batch their bursty traffic into the active windows to maximise idle time elsewhere. The OBFF Code in DW 2 byte 3 encodes the state: `0000` = CPU Active, `0001` = OBFF (Memory and I/O active but CPU idle), `0011` = Idle, other values Reserved. OBFF can also be signalled out-of-band through the WAKE# pin or via PCIe Physical Layer beaconing; the Message form is the in-band path.

## SPECIFICATIONS

- PCI Express Base Specification, section 6.19: Optimized Buffer Flush/Fill
- PCI Express Base Specification, section 7.8.3: OBFF Extended Capability
- PCI Express Base Specification, section 2.2.8.7: Message Request Header Format

## OTHER SOURCES

- [PCISIG, PCI Express Base Specification (member access)](https://pcisig.com/specifications)

## FIELDS

| Field | Value |
|---|---|
| Fmt[2:0] | `001` (4DW, no data) |
| Type[4:0] | `10011` (Broadcast from RC) |
| Length[9:0] | `0` |
| Message Code | `0x12` |
| Requester ID[15:0] | the Root Complex |
| Tag[7:0] | Reserved |
| DW 2 high 28 bits | Reserved |
| DW 2 low 4 bits | OBFF Code |
| DW 3 | Reserved |

### OBFF Code values

```
       OBFF Code   Meaning
       ────────────────────────────────────────
       0000        CPU Active (full performance)
       0001        OBFF (Memory + I/O active, CPU idle)
       0011        Idle (everything quiescent)
       others      Reserved
```

## DETAILS

