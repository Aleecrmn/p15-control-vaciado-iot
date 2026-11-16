# Esquema de Conexiones y Montaje — Proyecto P15

## Alimentación y Consideraciones Eléctricas
* **Microcontrolador (ESP32):** Alimentado a 5V vía USB o regulador externo estable.
* **Módulo Relé (1 canal):** Alimentado a 5V; señal de control conectada a `GPIO 13`. Se recomienda aislamiento optoacoplado y diodo flyback si se conmuta una electroválvula o bomba DC inductiva.
* **Sensor de Caudal (YF-S201):** Alimentado a 5V; cable de señal (amarillo) conectado a `GPIO 23` con resistencia `INPUT_PULLUP` interna activada.
* **Sensor Ultrasónico (HC-SR04):** Alimentado a 5V; pin `ECHO` (`GPIO 25`) adaptado mediante divisor resistivo ($1\text{ k}\Omega$ / $2\text{ k}\Omega$) para proteger la entrada lógica de $3.3\text{V}$ del ESP32.
* **Pantalla OLED SSD1306 (0.96"):** Alimentada a $3.3\text{V}$ en bus I2C estándar (`SDA` en `GPIO 21`, `SCL` en `GPIO 22`, dirección `0x3C`).

## Tabla de Cableado Rápido

| Pin ESP32 | Destino | Color / Notas |
| :--- | :--- | :--- |
| `3V3` | VCC OLED SSD1306 | Bus lógico $3.3\text{V}$ |
| `VIN / 5V` | VCC HC-SR04, YF-S201, Relé | Bus periféricos $5\text{V}$ |
| `GND` | Tierra común (GND) | Compartida entre fuente, sensores y ESP32 |
| `GPIO 26` | HC-SR04 `TRIG` | Pulso de disparo $10\ \mu\text{s}$ |
| `GPIO 25` | HC-SR04 `ECHO` | Con divisor de voltaje $5\text{V} \rightarrow 3.3\text{V}$ |
| `GPIO 23` | YF-S201 `SIGNAL` | Interrupción externa (`RISING`) |
| `GPIO 13` | Relé `IN` | Activo en alto (`HIGH` = vaciando) |
| `GPIO 19` | Ánodo LED Rojo | Con resistencia limitadora de $220\ \Omega$ |
| `GPIO 4` | Ánodo LED Verde | Con resistencia limitadora de $220\ \Omega$ |
| `GPIO 27` | Positivo Buzzer Pasivo | Señal PWM $2000\text{ Hz}$ |
| `GPIO 21` | OLED `SDA` | Datos I2C |
| `GPIO 22` | OLED `SCL` | Reloj I2C |