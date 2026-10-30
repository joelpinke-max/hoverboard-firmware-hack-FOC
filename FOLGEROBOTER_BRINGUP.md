# Folgeroboter: Ersatzplatine (GD32F103RCT6) – Safe-Start

**STATUS: ONLY A RESTRICTED POWER-LATCH TEST. NOT A DRIVING FIRMWARE.**

This branch is based on stock BOARD_VARIANT 0 because that variant previously
appeared to maintain the power LED after button release. That observation
does not prove which GPIO actually controls the latch.

## Verified and unverified findings

- The ST-Link successfully programmed and verified an ELF image.
- The vector table at 0x08000000 contains plausible SP/PC start values.
- When debug-halted, GPIOx_CRH registers read 0x44444444 and APB2ENR was zero;
  these are consistent with reset, not evidence for a particular board pinout.
- The PCB's motor gate-drive, current measurement, Hall, charger, buzzer,
  button and latch wiring are still **NOT VERIFIED**.
- Neither BOARD_VARIANT 0 nor BOARD_VARIANT 1 is confirmed as a safe
  driver for this visually different replacement PCB.

## Changes on this branch

- Uses BOARD_VARIANT 0 as an *unverified starting point*.
- Asserts the candidate PA5 power-latch GPIO immediately after HAL_Init,
  before any clock/peripheral/motor setup.
- Adds independently editable pin definitions for latch, button,
  buzzer, charger and battery ADC in Inc/config.h.
- `FOLGEROBOTER_SAFE_START=1` activates only the candidate latch pin,
  holds it for 8 seconds and then deasserts it; loops afterwards.
- Does **not** configure or run PWM, MOSFET drive timers, UART, FOC,
  Hall feedback, or ADC in the safe-start code path.
- Full motor startup is blocked at compile time while
  `FOLGEROBOTER_BOARD_WIRING_VERIFIED=0`.

## Safety warning

The candidate PA5 pin is not traced on the new PCB. Do not treat
this experimental firmware as compatible with the 36 V power hardware.
After two board failures including fire, testing requires a qualified
electronics person to check the supply, wiring, shorts, battery safety,
and actual power-stage mapping, preferably using safe current-limited
supply methods. No motor or Raspberry Pi should be attached for bring-up.
The original battery/charger connection is not approved by this build.

## Artifact

GitHub > Actions > branch `folgeroboter-v2-latch-safe-20261030`
> latest successful Build > Artifacts > open
`VARIANT_USART/firmware.elf`.

A successful compiler build does not establish electrical compatibility.
