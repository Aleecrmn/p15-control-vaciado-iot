# Control IoT Tolerante a Fallos para Gestión de Estanques (Proyecto P15)

Sistema embebido basado en **ESP32** para el monitoreo de nivel, caudal y control automatizado de vaciado de estanques mediante una **Máquina de Estados Finitos (FSM)**, telemetría en tiempo real vía **MQTT** y visualización/control remoto en **Node-RED**.

> **Feria de Proyectos IoT 2026** — Equipo E18

---

## Arquitectura y Características Principales

* **Control Determinístico (FSM):** Tres estados operativos (`REPOSO`, `VACIANDO`, `ERROR_SEGURO`) con filtros anti-olas por histéresis temporal (contadores de confirmación antes de conmutar el relé).
* **Protección Activa (`ERROR_SEGURO`):**
  * Detección de fallo en sensor ultrasónico tras 5 lecturas consecutivas fuera de rango.
  * Protección de bomba/válvula en seco: corte automático tras 10 ciclos (5 s) sin detección de caudal (`< 0.5 L/min`).
* **Telemetría y Control Bidireccional (MQTT):** Publicación periódica cada 10 segundos en formato JSON y recepción de comandos manuales (`ON` / `OFF`) desde el dashboard con reconexión automática no bloqueante a WiFi y broker MQTT.
* **Interfaz Local:** Pantalla OLED SSD1306 (128x64) con distancia calibrada, porcentaje de llenado, caudal instantáneo y estado de la FSM, acompañado de indicadores LED y alarma sonora intermitente (buzzer pasivo a 2 kHz).

---

## Lógica de la Máquina de Estados (FSM)

![Diagrama FSM](docs/diagrama_fsm.png)

| Transición | Condición | Acción |
| :--- | :--- | :--- |
| `REPOSO` $\rightarrow$ `VACIANDO` | `nivel >= 70%` (3 muestras) o `cmd: ON` | Activa relé de vaciado, LED rojo y alerta sonora intermitente |
| `VACIANDO` $\rightarrow$ `REPOSO` | `nivel <= 30%` (4 muestras) o `cmd: OFF` | Apaga relé, enciende LED verde, silencia buzzer |
| `Cualquier estado` $\rightarrow$ `ERROR_SEGURO` | 5 errores de lectura ultrasónica | Apaga relé por seguridad, enciende LED rojo fijo |
| `VACIANDO` $\rightarrow$ `ERROR_SEGURO` | 10 ciclos (5 s) con caudal `< 0.5 L/min` | Corte de emergencia por falta de flujo |

---

## Pinout del Hardware (ESP32)

| Componente | Pin ESP32 | Tipo | Notas |
| :--- | :---: | :--- | :--- |
| Sensor Ultrasónico (TRIG) | `GPIO 26` | Salida | Calibración lineal aplicada ($m=0.99, b=0.95$) |
| Sensor Ultrasónico (ECHO) | `GPIO 25` | Entrada | Timeout de 30 ms |
| Sensor de Caudal (YF-S201) | `GPIO 23` | Entrada (IRQ) | `INPUT_PULLUP`, interrupción por flanco de subida |
| Módulo Relé (Vaciado) | `GPIO 13` | Salida | Activo en alto (`HIGH`) |
| LED Rojo (Alerta / Vaciado) | `GPIO 19` | Salida | Encendido en `VACIANDO` o `ERROR_SEGURO` |
| LED Verde (Sistema OK) | `GPIO 4` | Salida | Encendido en `REPOSO` normal |
| Buzzer Pasivo | `GPIO 27` | Salida PWM | Tono intermitente de 2000 Hz en vaciado |
| Pantalla OLED SSD1306 | `GPIO 21 / 22` | I2C | Dirección `0x3C` (SDA=21, SCL=22) |

---

## Formato de Telemetría MQTT

* **Tópico de publicación:** `curso/<usuario>/P15/estanque`
* **Tópico de estado (LWT):** `curso/<usuario>/P15/estanque/estado` (`online` / `offline`)
* **Tópico de comandos:** `curso/<usuario>/P15/estanque/cmd` (`{"cmd": "ON"}` / `{"cmd": "OFF"}`)

**Ejemplo de payload JSON publicado:**
```json
{
  "distancia_cm": 12.0,
  "nivel_pct": 35.1,
  "caudal_l_m": 0.0,
  "estado_fsm": 0,
  "sensor_ok": 1,
  "rssi_dbm": -62
}
