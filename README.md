# PsNee v6 ModChip


PsNee, an open source stealth modchip for the Sony Playstation 1 

                        PPPPPPPPPPPPPPPP                  P            P      
                       P              P                  PP           P        
                      P              P                  P P          P        
                     P              P                  P  P         P          
                    P              P                  P   P        P          
                   P              P                  P    P       P            
                  P              P                  P     P      P            
                 PPPPPPPPPPPPPPPP  PPPPPPPPPPP     P      P     P  PPPPPPPPPPP  PPPPPPPPPPP
                P                 P               P       P    P  P            P
               P                 P               P        P   P  P            P  
              P                 P               P         P  P  P            P  
             P                 P               P          P P  P            P    
            P                 PPPPPPPPPPPPPP  P           PP  PPPPPPP      PPPPPPP    
           P                              P  P            P  P            P      
          P                              P  P            P  P            P      
         P                              P  P            P  P            P        
        P                              P  P            P  P            P        
       P                              P  P            P  P            P      
      P                              P  P            P  P            P        
     P                   PPPPPPPPPPPP  P            P  PPPPPPPPPPP  PPPPPPPPPPP

---------------------------------------------------------------
This version is from
http://www.psxdev.net/forum/viewtopic.php?f=47&t=1262&start=40
Is developed by the psxdev team

---------------------------------------------------------------

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

-------------------------------------------------------------------------------------
## General Info

                      Pin assignments
                        
                      PSNee psxdev                                PlayStation
      Arduino   Atinny   ProMicro   Leonardo    name           ps pin         Name

      pin-5v   = VCC    = VCC      = pin-5v    = 3.5v         3.5v           = supply
                 3      =          =           = debugtx
      pin-9    = 4      = pin-15   = icsp-sck  = gate_wfck    IC732.Pin-5    = WFCK           
      pin-8    = 2      = pin-14   = icsp-miso = data         IC732.Pin-42   = CEO
      pin-7    = 1      = pin-3    = pin-3     = subq         IC304.Pin-24   = SUBQ
      pin-6    = 0      = pin-2    = pin-2     = sqck         IC304.Pin-26   = SQCK
      pin 5             = pin-9    = pin-9     = BIOS D2      IC102.Pin-15   = D2
      pin 4             = pin-8    = pin-8     = BIOS A18     Ic102.Pin-31   = A18
      pin-gnd  = GND    = GND      = pin-gnd   = gnd          GND            = gnd


### PLAYSTATION 1 SECURITY - HOW IT DOES ITS THING:

Sony didn't really go through great lenghts to protect its precious Playstation
from running unauthorised software: the main security is based on a simple ASCII
string of text that is read from a part of an original Playstation disc that cannot
be reproduced by an ordinary PC CD burner.
As most of you will know, a CD is basically a very long rolled up (carrier) string in which very
little pits and ehm... little not-pits are embedded that represent the data stored on the disc.
The nifty Sony engineers did not use the pits and stuff to store the security checks for
Playstation discs, but went crazy with the rolled up carrier string. In an ordinary CD, the
string is rolled up so that the spacing between the tracks is as equal as possible. If that
is not the case, the laser itself needs to move a bit to keep track of the track and
reliably read the data off the disc.
If you wonder how the laser knows when it follows the track optimally: four photodiodes, light
intensity measurement, difference measurements, servo. There.
To the point: the Sony engineers decidedly "fumbled up" the track of sector 4 on a Playstation
disc (the track was modulated, in nerd-speak) so that the error correction circuit outputs a
recognisable signal, as the laser needs to be corrected to follow the track optimally.
This output signal actually is a 250bps serial bitstream (with 1 start bit and 2 stop bits) which
in plain ASCII says *SCEA* (Sony Computer Entertainment of America), *SCEE* (Sony Computer Entertainment
of Europe) or *SCEI* (Sony Computer Entertainment of Japan), depending on the region of the disc inserted.
The security thus functions not only as copy protection, but also as region protection.
The text string from the disc is compared with the text string that is embedded in the Playstation
hardware. When these text strings are the same, the disc is interpreted to be authentic and from
the correct region. Bingo!

### The master branch is completely redesigned!

The original code doesn't have a mechanism to turn the injections off. It bases everything on a timer.
After power on, it will start sending injections for some time, then turns off.
It also doesn't know when it's required to turn on again (except for after a reset), so it gets detected by anti-mod games.

### The mechanism to know when to inject and when to turn it off.

This is the 2 wires for SUBQ / SQCK. The PSX transmits the current subchannel Q data on this bus. It tells the console where on the disc the read head is. We know that the protection symbols only exist on the earliest sectors, and that anti-mod games exploit this by looking for the symbols elsewhere on the disk. If they get those symbols, a modchip must be generating them!

So with that information, my code knows when the PSX wants to see the unlock symbols, and when it's "fake" / anti-mod. The chip is continously looking at that subcode bus, so you don't need the reset wire or any other timing hints that other modchips use. That makes it compatible and fully functional with all revisions of the PSX, not just the later ones. Also with this method, the chip knows more about the current CD. This allows it to not send unlock symbols for a music CD, which means the BIOS starts right into the CD player, instead of after a long delay with other modchips.

This has some drawbacks, though:
 * It's more logic / code. More things to go wrong. The testing done so far suggests it's working fine though.
 * It's not a good example anymore to demonstrate PSX security, and how modchips work in general.
