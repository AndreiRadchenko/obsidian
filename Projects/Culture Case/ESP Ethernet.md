---
date_created: 2026-01-20
date_modified: 2026-01-20
document_type: note
project: Culture Case
tags: culture-case note
---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
# ESP Ethernet
**Overview**
Description:: 

## Note

Це плата розробника **WT32-ETH01** від Wireless-Tag на базі модуля ESP32 WT32‑S1 з вбудованим Ethernet‑портом (LAN8720A PHY).[1][2]

![[image.jpg | center | 600]]
## Основні характеристики

- Мікроконтролер WT32‑S1 (ESP32‑клас): 2 ядра Xtensa 32‑bit, до 240 МГц, 520 КБ SRAM, 4 МБ флеш.[2][3]
- Інтерфейси: Ethernet 10/100 Мбіт/с через роз’єм RJ‑45, Wi‑Fi 2.4 ГГц 802.11b/g/n, Bluetooth 4.2 BR/EDR + BLE.[4][2]
- Підтримка стеку TCP/IP з режимами TCP Server/Client, UDP Server/Client, можливе прозоре “serial‑to‑Ethernet/Wi‑Fi” передавання.[5][4]

## Призначення плати

- Шлюз RS‑485/Modbus або UART‑пристроїв у Ethernet та/або Wi‑Fi мережу.[6][4]
- IoT‑шлюзи, промислова автоматика, домашня автоматизація, де потрібен одночасно дротовий LAN і бездротовий доступ.[1][2]

## Виводи та живлення

- Живлення: 5 V або 3.3 V (є окремі пін‑ряди 5V, 3V3, GND; на борту присутній стабілізатор до 3.3 V).[7][1]
- Системні піни: EN (скидання/enable), TX0/RX0 для програмування по UART, IO0 використовується як BOOT при прошивці.[8][7]
- Додаткові GPIO: виведені IO32–IO39, а також лінії з підписами 485_EN, RXD, TXD, які відповідають GPIO33, GPIO5, GPIO17 і можуть використовуватись як звичайні GPIO.[6][1]

## Програмування та прошивки

- Підтримується Arduino Core for ESP32, ESP-IDF, ESPHome та інші фреймворки для ESP32.[2][8]
- Для прошивки зазвичай використовують послідовний порт: підключення до TX0/RX0, за потреби притягування IO0 до GND для входу в режим завантажувача.[7][8]
 > [!при прошивці треба посадити на нуль IO0 до кінця прошивки модуля по послідовному інтерфейсу]
> 

For the WT32-ETH01 board, finding 12 available GPIO inputs is challenging because the board's integrated Ethernet PHY consumes many of the standard ESP32 pins. While the board has 12–13 exposed GPIO pins, several have significant hardware restrictions or "strapping" requirements during boot.

### Available GPIO Pins for Input
To reach 12 inputs, you will need to use almost every exposed pin, including serial and strapping pins. The pins below are listed by their suitability for input.

| GPIO | Label | Note | Internal Pull-up? |
| :--- | :--- | :--- | :--- |
| **36** | **36** | **Input Only**. Used as Sensor_VP. | No |
| **39** | **39** | **Input Only**. Used as Sensor_VN. | No |
| **35** | **35** | **Input Only**. Often swapped with IO5 on some boards. | No |
| **32** | **CFG** | General purpose; safe to use. | Yes |
| **33** | **485_EN**| General purpose; safe to use. | Yes |
| **4** | **4** | General purpose; safe to use. | Yes |
| **14** | **14** | General purpose; safe to use. | Yes |
| **5** | **RXD** | Safe to use, but note it is used for strapping (high) at boot. | Yes |
| **17** | **TXD** | Safe to use. | Yes |
| **15** | **15** | Strapping pin. Must be floating or LOW for some functions. | Yes |
| **12** | **12** | Strapping pin. **Must be LOW** at boot for standard 3.3V flash. | Yes |
| **2** | **2** | Strapping pin. Must be LOW to enter flashing mode. | Yes |

### Critical Hardware Warnings
1.  **Missing Pull-ups**: GPIO 35, 36, and 39 **do not have internal pull-up resistors**. If you are using these for switches or sensors, you must add an external 10k$$\Omega$$ resistor to 3.3V.[1]
2.  **Strapping Pins**: GPIOs 2, 5, 12, and 15 are used by the ESP32 to determine boot modes. If you pull these HIGH or LOW externally (e.g., with a sensor) during power-on, the board may fail to start or fail to enter the correct flash voltage.[2]
3.  **Serial Pins**: You can technically use the **RX0** (GPIO 3) and **TX0** (GPIO 1) pins for a total of 14 inputs, but this will disable your ability to debug via USB-to-TTL serial.[3][4]

### Recommendation for 12 Inputs
If your project requires high reliability, using an **I2C Port Expander** like the **MCP23017** (16 extra pins) or **MCP23008** (8 extra pins) is the professional standard for this board. You can connect the expander to GPIO 32 and 33 (SDA/SCL) and gain all 12+ inputs with a single I2C bus, avoiding all boot-loop risks associated with strapping pins.[1]

Reducing the number of sensors to 6 makes the project significantly more stable and allows you to power them directly from the WT32-ETH01's onboard regulator with much lower risk of overheating.

### Power Impact for 6 Sensors
With 6 TCRT5000 modules, your current consumption drops to a manageable level for the internal 3.3V system.

*   **Sensor Draw**: $$6 \times 15\text{ mA} = 90\text{ mA}$$.
*   **Total System Load**: ~590mA (including the ESP32 and Ethernet PHY).[1][2]
*   **Safety**: This is well within the limits of the AMS1117-3.3 regulator, which can safely handle this load without excessive heat buildup when fed by a stable 5V source.[3][4]

### Optimized Pin Selection
Since you only need 6 pins now, you can avoid the most "dangerous" strapping pins (like GPIO 12) that often cause boot loops if sensors are active during power-on.

| GPIO | Label | Sensor Type | Why this pin? |
| :--- | :--- | :--- | :--- |
| **32** | **CFG** | **With Pull-up** | High reliability, no boot restrictions. |
| **33** | **485** | **With Pull-up** | High reliability, no boot restrictions. |
| **4** | **IO4** | **With Pull-up** | standard GPIO, safe at boot. |
| **36** | **I36** | **No Pull-up** | Input-only; your sensor provides the high signal [5]. |
| **39** | **I39** | **No Pull-up** | Input-only; safe and isolated from other logic. |
| **35** | **I35** | **No Pull-up** | Input-only; excellent for active-low sensors. |

### Configuration Advantages
*   **No Voltage Shifting**: By using the **3V3** pin of the ESP32 to power these 6 sensors, their output signal (DO) will naturally be 3.3V, which is perfectly safe for the ESP32's 3.3V logic.[6]
*   **Internal Pull-ups**: For GPIO 32, 33, and 4, you can still enable `INPUT_PULLUP` in your code for extra noise immunity, even though your sensors provide a high signal when inactive.
*   **Stability**: You no longer need to worry about the "Strapping Pins" (GPIO 2, 5, 12, 15), meaning the board will boot reliably every time regardless of whether something is in front of the sensors.

### Final Wiring Tip
Even with 6 sensors, ensure your 5V power supply is rated for at least **2A**. The WT32-ETH01 can have brief current spikes when the Ethernet controller is active or when the ESP32's radio is transmitting, and a weak power supply is the most common cause of "random" reboots.[7]


## References


---
[[Projects/Culture Case/Home|Home]] | [[Projects/Culture Case/Meetings/All Meetings|Meetings]] | [[Projects/Culture Case/Notes/All Notes|Notes]] | [[Projects/Culture Case/References|References]]
