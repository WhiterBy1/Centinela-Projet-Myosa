# CENTINELA — Sensores: qué se mide y qué se deriva

Catálogo de referencia: qué dato crudo entrega cada dispositivo del esquemático y qué valores se calculan a partir de ellos. Base para diseñar las estructuras de datos del firmware antes de escribir código.

---

## 1. Qué sensa cada dispositivo (valores crudos)

| Dispositivo | Ref. | Variable medida | Unidad / rango nativo | Resolución | Frecuencia | Notas |
|---|---|---|---|---|---|---|
| MPU6050 | U1 | Aceleración X, Y, Z | LSB → g (rango ±4g) | 16-bit, 8192 LSB/g | 50 Hz (FIFO) | Sensor primario del clasificador |
| MPU6050 | U1 | Velocidad angular X, Y, Z (gyro) | LSB → °/s (rango ±500°/s) | 16-bit, 65.5 LSB/(°/s) | 50 Hz (FIFO) | Idem |
| MPU6050 | U1 | Temperatura del die (interna al chip) | °C | 16-bit | bajo demanda | No es temperatura ambiente real, no usar como contexto — para eso está el BMP280 |
| BMP280 | U2 | Temperatura ambiente | °C | ~0.01°C típico | 1 muestra / 60 s (forced mode) | Variable de contexto, no de decisión |
| BMP280 | U2 | Presión atmosférica | hPa | 20-bit | 1 / 60 s | Disponible por el chip pero no usada en el MVP |
| APDS-9960 | U3 | Proximidad | 0–255 (unidad relativa) | 8-bit | event-driven (INT) | Dispara la ISR de gesto/wake |
| APDS-9960 | U3 | Gesto | evento discreto: UP / DOWN / LEFT / RIGHT | — | event-driven (INT) | Triple uso: wake, zona, cancelación (ver tabla derivados) |
| APDS-9960 | U3 | Luz ambiental (Clear/R/G/B) | cuentas ADC | 16-bit | bajo demanda | Disponible pero no usada en el MVP |
| Adafruit Feather M0 RFM9x | A1 | RSSI del último paquete recibido | dBm | — | por paquete (downlink/ACK) | Indicador de calidad del enlace, no del entorno |
| Adafruit Feather M0 RFM9x | A1 | SNR del último paquete recibido | dB | — | por paquete | Idem |
| ADC batería (ESP32) | — | Voltaje de la celda 18650 | mV | 12-bit | periódico (ej. 1/min) | **Pendiente:** confirmar si el esquemático ya tiene el divisor resistivo hacia el ADC |

---

## 2. Qué se deriva / calcula a partir de esos datos

| Valor derivado | Se calcula a partir de | Cómo | Para qué se usa |
|---|---|---|---|
| Magnitud de aceleración `\|a\|` | ax, ay, az | `sqrt(ax²+ay²+az²)` | Feature base de movimiento |
| Magnitud de giro `\|ω\|` | gx, gy, gz | `sqrt(gx²+gy²+gz²)` | Feature de actividad rotacional |
| Media y desv. estándar de `\|a\|` por ventana | `\|a\|` de 64 muestras | estadística sobre la ventana | Entrada al clasificador TinyML |
| Energía de la señal por eje | ax, ay, az de la ventana | `Σ x²` por eje | Entrada al clasificador TinyML |
| Cruces por cero | derivada de `\|a\|` | conteo de cambios de signo | Proxy de temblor/actividad, entrada al clasificador |
| `immobility_score` (prob. de caída/inmovilidad) | features anteriores | salida del modelo TinyML (0.0–1.0) | Dispara sospecha si supera umbral |
| Estado de movimiento (NORMAL / SOSPECHA / CONFIRMANDO / ALERTA) | `immobility_score` + duración sostenida (nº de ventanas consecutivas) | máquina de estados | Controla todo el flujo de decisión |
| `zone_id` (1–4) | gesto detectado (APDS-9960) | mapeo gesto → zona declarada | Sustituto de GPS, viaja en cada paquete LoRa |
| Evento de cancelación de alarma | gesto detectado durante la ventana de confirmación | — | Corta la transmisión de alerta |
| `severity_bonus` | temperatura ambiente (BMP280) | `temp_c > 35°C ? 1 : 0` | Modula la severidad de la alerta |
| `alert_severity` final (0–3) | `immobility_score` + `severity_bonus` | `base_severity(score) + severity_bonus` | Campo del paquete LoRa, define urgencia |
| `battery_pct` (%) | voltaje de batería (mV) | curva de descarga típica Li-ion 18650 (tabla mV→%) | Mostrado en OLED, incluido en heartbeat |
| Calidad de enlace / necesidad de reintento | RSSI, SNR, si llegó ACK o no | umbral de RSSI/SNR o timeout de ACK | Decide reintentos de transmisión (sección 6 del documento de arquitectura) |
| Registro final en la torre de control | todos los campos del paquete + `received_at` (timestamp del servidor) | persistencia en servidor | Lo que ve el supervisor en el panel web |

---

*Complementa a `readme.md` (arquitectura de firmware) — ahí está el detalle de código de cómo se adquiere cada dato y cómo se arma el paquete final.*
