Omega

Background.
This project started around December 2017, when I was doing some bare metal coding on the RaspberryPi. 
After getting a basic kernel booting I realised I had no software to run, the killer of all hobbyOS projects. So my mind wandered what would it be like if the RaspberryPI had a 68000 CPU instead of an ARM? 
There are plenty of popular "old school" 68k operating systems out there (TOS, Classic Mac, AmigaOS) with lots of great software, why can't I run these on the raspberryPi wihout a host OS?

After the birth of my daughter, I found I had some free time so I set about trying to make a RaspberryPi kernel which was nothing but a 68000 emulator running on the bare metal, with full access to the RaspberryPi address space. 
I decided to use the Musashi 68k emulator as the basis as i had used it in another project so had the source code working in "raw C" (no support libraries or host OS functions) and knew the emulator well. As the project developed, I realised that since I needed a set of functions to do the byte swapping to and from the memory there was no reason why I couldn't trap calls to certan addresses an emulate extra hardware functionality required to get an old 68k OS booting.

This is how the Omega project was born, although I initially called it Zorro (after one of the Commodore Amiga prototypes). Since the only real hardware coding I have done on a 68k machine was with the Amiga, that it the machine I knew best to emulate.
In retrospect choosing a simpler machine (like the Atari ST or classic Mac) would probably have been a better idea, as Amiga emulators are difficult to get working and nothing will ever come close to the functionlity of (Win)UAE. It seems unlikely there will be any more Amiga emulators, this could well be the last.. So I decided that a half pun, half tongue in cheek name of Omega was probably better than Zorro (which has other meanings in the Amiga world).

The current code base is a mixture of the original "Zorro" project with one coding style which was hacked together to "just work"(tm) and the project rewrite with a different coding style, making the code very messy. I hope to clean this up over time.

Architecture.
The emulator is actually very simple. It is built around a core set of functions which manage memory access to the lower 16 megabytes of the system address space (For the time being the upper address space isn't accesible, since that adds a whole level of complexity). Within this 16meg space, there are four descrete areas, the 'Chipram", the "CIA" chips, the custom "chipset" registers, and the ROM. Almost all memory access to this space is big endian, where my plan is that the upper address space will be native endian. Right now the chipset registers are stored as a seprate structure (but in future I will probably allocate the memory for them in the actual 16meg space). 

The next part is the "chipset" register trap, a set of functions which trap read/writes to any address between 0xDFF000 and 0xDFF1FF and implements any special hardware function located at that address. For example a write to address 0xDFF09C needs to write a bit pattern to address DFF01E and potentially riase an IRQ on the 68K. This is imlmented as a set of function tables with specific functions for different sized acccesses (8bit, 16bit, 32bit). Since the data written to these registers is used by the extensively by the emulator internally, the are kept in native endian byte ordering for efficency. 
The "CIA" chips are similar to the chipset part, but also require a periodic function since they implement timers and are able to raise interrupts.

The next part is the DMA sequencer. This should have been the core of the project, but it wasn't originally part of my design. This is fundamental to how the Amiga works, again implemeneted as a single function table containg 227 funcitions, a counter steps through each function advance every system cycle wrapping back to 0 when it reaches the end. There are "ood" and "even" cycles. "Odd" cycles hvae specific hardware functions mostly related to display generation. 
