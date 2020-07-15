# PsNee v6 ModChip

## Diagram

Uses the same diagrams as MM3, as this is pin-compatible. Only connect Pins 1,4,5,6,7,8. Pins 2,3 are unused.

```diagram
                      ________  ________
                      |       \/       |
           VDD (5V) --+ 1 >>      >> 8 +-- VSS (Ground)
                      |                |
           /        --+ 2         << 7 +-- signal from door (GPIO0)
                      |                |
           /        --+ 3         >> 6 +-- data stream (GPIO1)
                      |                |
         MCLR Reset --+ 4 >>      >> 5 +-- gate output (GPIO2)
                      |                |
                      +----------------+
```
