---
title: "Using SWV Trace (with printf) on STM32F103 Blue Pill"
description: "Getting SWV Trace debugging (including redirecting printf) to work with my STM32F103 Blue Pill and ST-Link v2 chinese clone, using STM32CubeIDE as my development environment."
---

**Summary:** I had to solder a piece of magnet wire to one of the pins on the chip of the ST-Link so that I could connect it to the `TRACESWO` pin on my STM32F103 Blue Pill. Here's the tutorial I (more or less) followed: <https://lujji.github.io/blog/stlink-clone-trace/> I also had to make sure STM32CubeIDE was configured correctly, which I go over below, and used a short snippet of code (also below) to redirect `printf` to the SWV Data Trace console.

After a lot of flailing (and thinking that maybe my version of STM32CubeIDE was just broken), I finally got SWV Trace working. It turns out that it requires having a pin on the ST-Link connected to the SWO pin on the STM32. For some reason, though, this pin isn't one of the pins that's normally accessible on the chinese ST-Link clones, so I had to solder on a piece of magnet wire. It works now!

1. Take apart the ST-Link. I found that if I held the metal case and pushed on the USB connector, the PCB (which is attached to the USB connector) popped out pretty easily.

2. Solder a length of magnet wire (just thin copper wire with enamel insulation) onto pin 31 (AKA `PA10`) of the chip inside the ST-Link. See tutorial for more info: https://lujji.github.io/blog/stlink-clone-trace/ The chip in my ST-Link is a CKS32F103C8T6, which appears to be a clone of the original STM32F103C8T6. It worked for me, though, so having a cloned chip wasn't a problem.

3. Solder a piece of 22 gauge solid copper wire onto the end of the magnet wire for breadboarding, then put the metal case back onto the ST-Link. I also superglued the magnet wire onto the PCB to (hopefully) prevent it from getting pulled off the trace and cut a little groove into the plastic pin holder thing (?) for the magnet wire to pass through when the case is back on.

4. Enable SWV in STM32CubeIDE. Open the `Debug Configuations...` under the Debug icon (the weird little insect/tick looking thing). In the `Debugger` tab, enable `Serial Wire Viewer (SWV)` and make sure the `Core Clock` is set to your system clock (which you can find in STM32CubeIDE under `SYSCLK`). 

5. In the Debugger perspective, click on `Window -> Show View -> SWV -> SWV Data Trace` (and `SWV Data Trace Timeline Graph`) to be able to view the SWV output.

6. Enable the `SYS_JTDO-TRACESWO` pin on your microcontroller (on my STM32F103C8 this was on pin `PB3`) and set the `Debug` method under `SYS` (which is itself under `System Core` in STM32CubeMX) to `Trace Asynchronous Sw`.

7. Run the generated code (in debug mode, by pressing the little bug icon) on your microcontroller. Make sure the `TRACESWO` on your microntroller (`PB3` in my case) is connected to the new pin you soldered onto your ST-Link. While the debugger is paused on the first breakpoint (I wasn't able to get this working otherwise) bring up the SWV settings by clicking on the wrench icon under the `SWV Data Trace` tab. Under `ITM Stimulus Ports`, enable pin 0. Start tracing by clicking the red circle.

Some of these steps might not be necessary, but it's what I ended up doing to get SWV working. I also redirected `printf` output to the `SWV Data Trace` console by adding the following code to my `main.c`:

```c
// Just make sure stdio.h is included
#include "stdio.h"

// ... some other code in between

int _write(int32_t file, uint8_t *ptr, int32_t len) {
	/* Implement your write code here, this is used by puts and printf for example */
	int i = 0;
	for(i=0; i < len; i++) {
		ITM_SendChar((*ptr++));
	}
	return len;
}
```
