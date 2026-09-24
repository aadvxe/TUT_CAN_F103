# TUT_CAN_F103 (STM32F103 "Blue Pill" CAN Bus Tutorial)

A starter and tutorial project for the popular **STM32F103C8T6 ("Blue Pill")** development board demonstrating CAN bus communication, interrupt-driven message reception, and hardware LED response.

---

## 🎯 Project Overview

This project serves as an introductory reference for working with the **bxCAN** controller on the STM32F1 series using the STM32 HAL library. It listens for incoming CAN frames, parses command parameters, blinks the onboard LED according to the received payload, and transmits a reply frame back to the bus.

- **MCU:** STM32F103C8T6 (ARM Cortex-M3 @ 72 MHz, 64 KB Flash, 20 KB RAM)
- **Board:** STM32F103 "Blue Pill"
- **Toolchain:** STM32CubeIDE / GCC ARM Embedded
- **CAN Pins:** `PA11` (CAN_RX), `PA12` (CAN_TX)
- **Status LED:** Onboard LED on `PC13` (active LOW)

---

## ⚙️ Communication Protocol & Operation

1. **CAN Initialization & Notification:**
   - Starts CAN controller: `HAL_CAN_Start(&hcan)`
   - Activates RX interrupt: `HAL_CAN_ActivateNotification(&hcan, CAN_IT_RX_FIFO1_MSG_PENDING)`

2. **Reception & LED Blink Sequence:**
   When a CAN message is received into `RxData`:
   - `RxData[0]`: Blink delay in milliseconds.
   - `RxData[1]`: Number of blink cycles.
   - The onboard LED (`PC13`) is toggled for `RxData[1]` repetitions with `RxData[0]` ms interval.

3. **Transmission Acknowledgment:**
   After completing the blink routine, the board transmits a CAN message back to the bus:
   - **Standard ID:** `0x103`
   - **DLC:** `2` bytes
   - **Payload:** `[200, 20]` (`TxData[0] = 200`, `TxData[1] = 20`)

---

## 🔌 Hardware Wiring

Because STM32 microcontrollers only provide TTL-level CAN TX and RX signals, an external **3.3V CAN Transceiver** (e.g. SN65HVD230 or VP230) is required:

```
STM32F103 Blue Pill               CAN Transceiver (SN65HVD230)          CAN Bus
  [ PA11 ] -----------------------> [ RX / CRX ]
  [ PA12 ] -----------------------> [ TX / CTX ]
  [ 3.3V ] -----------------------> [ 3V3 / VCC ]
  [ GND  ] -----------------------> [ GND ]
                                    [ CANH ] ------------------------> CAN_H
                                    [ CANL ] ------------------------> CAN_L
                                    (120 Ω bus termination resistor across CAN_H & CAN_L)
```

---

## 📁 Folder Structure

```
TUT_CAN_F103/
├── Core/
│   ├── Inc/
│   │   ├── main.h              # Pin & peripheral declarations
│   │   └── stm32f1xx_hal_conf.h
│   └── Src/
│       ├── main.c              # CAN initialization, RX callback, blink loop & TX
│       ├── stm32f1xx_hal_msp.c # Clock & GPIO setup for CAN1 & PC13
│       ├── stm32f1xx_it.c      # CAN1 RX1 interrupt handler
│       └── system_stm32f1xx.c  # Clock configuration
├── Drivers/                    # STM32F1xx HAL and CMSIS
├── TUT_CAN_F103.ioc            # STM32CubeMX project definition
└── STM32F103C8TX_FLASH.ld      # Linker script
```

---

## 🛠️ Building & Flashing

1. Open **STM32CubeIDE**.
2. Go to **File > Open Projects from File System...** and open `TUT_CAN_F103`.
3. Connect your **ST-LINK V2** programmer to the SWD pins of the Blue Pill board (`SWDIO`, `SWCLK`, `GND`, `3.3V`).
4. Build the project (`Ctrl+B`).
5. Flash the board: `Run > Run` (`Ctrl+F11`).
6. Send a CAN frame to the board with payload bytes (e.g. `[100, 5]` for 5 fast blinks) to observe the response.
