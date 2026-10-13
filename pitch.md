# CENTINELA — Pitch final IEEE MYOSA (borrador v0.1)

> Supuestos (ajustar cuando se confirme el formato de la final): pitch en **inglés**, ~**5 min** + Q&A, con **demo en vivo**. Si el tiempo es distinto, ver la sección "Versiones" al final.
> Notas en *cursiva* = instrucciones para quien presenta, no se dicen.

---

## Mensaje central (una sola frase que deben recordar los jueces)

**"A worker who falls alone in a field has no one to notice. CENTINELA notices for them — on the wrist, with no phone signal, no GPS and no cloud."**

---

## Guion (≈ 5:00)

### 1. Gancho — 0:00–0:30
*(Sin diapositiva de título todavía. Pausa, mirar a los jueces.)*

"Imagine a farmer working alone in a large field. Midday heat, 38 degrees. Nobody within a kilometre. He collapses. How long until someone finds him? Minutes? Hours? In agriculture, **that delay is what turns an accident into a fatality.**"

### 2. Problema — 0:30–1:00
"Agricultural workers face heat stress, dehydration and falls — often **without direct supervision** and **without mobile coverage**. Existing solutions assume a smartphone, GPS and cellular data. In the places where this matters most, none of those exist."

### 3. Solución — 1:00–1:45
"CENTINELA is a wrist-worn safety node built on the **MYOSA platform**. An ESP32 runs a **TinyML model on the device**: it analyses motion from an IMU and detects a fall followed by abnormal immobility. If it suspects an emergency, it does **not** alert immediately — it gives the worker **60 seconds to cancel with a hand gesture**. If there is no cancellation, it sends an alert over **LoRa** to a base station, which shows up on a **web control tower**."

*Diapositiva: el diagrama de arquitectura de `Idea proyecto.pdf` (pulsera → LoRa → estación base → torre de control).*

### 4. Por qué es diferente — 1:45–2:30
Tres ideas, una frase cada una:

1. **Edge AI, not cloud AI.** The decision is made on the wrist. Radio is only used when it matters → days of battery, no infrastructure.
2. **Human-in-the-loop.** The 60 s gesture window (APDS-9960) kills false alarms *before* they cost anyone's trust. Cancelled alarms are logged locally and can retrain the model.
3. **Context-aware severity, no GPS.** The temperature sensor raises the alert severity when it's >35 °C (heat stroke risk while waiting for rescue). The worker declares their field zone with a gesture, so every packet carries a `zone_id` — location without GPS.

### 5. DEMO EN VIVO — 2:30–4:00  ⭐ lo más importante
Guion exacto de la demo (ensayar hasta que sea automático):

| Seg. | Acción | Qué se ve |
|---|---|---|
| 0:00 | Pulsera en la muñeca, caminar/mover el brazo | OLED: `NORMAL`, zona activa |
| 0:10 | Gesto → declarar zona (ej. swipe derecha) | OLED muestra zona; aparece en la torre de control |
| 0:25 | Simular caída + quedarse inmóvil | OLED: `SOSPECHA` → `CONFIRMANDO (60 s)`, buzzer |
| 0:40 | **Mostrar la cancelación**: gesto izquierda | Vuelve a `NORMAL` (prueba del anti-falsos-positivos) |
| 0:55 | Repetir caída, **esta vez no cancelar** | Cuenta atrás → `ALERTA ENVIADA` |
| 1:15 | Mirar la torre de control | Alerta con device, zona, severidad, temperatura, batería |
| 1:25 | ACK de vuelta | Buzzer 2 beeps, OLED "Ayuda en camino" |

*Recomendación fuerte:* para la demo **acorten la ventana de 60 s a ~10 s con un parámetro/flag de demo** y díganlo explícitamente ("in the field this is 60 s; for the demo it's 10 s"). Esperar 60 s en vivo mata el ritmo.
*Plan B obligatorio:* video grabado de la demo completa (30–45 s) por si falla el radio, el WiFi del lugar o el sensor.

### 6. Validación y honestidad — 4:00–4:30
"This is a proof of concept and we are clear about its limits: it infers status from **wrist motion only**, uses **area-based zone localisation instead of GPS**, and the classifier was trained on public wrist fall datasets plus our own recordings with the real hardware. *[completar con cifras reales — ver pendientes]*"

Decir los límites **uno mismo** convierte la mayor debilidad en credibilidad y se adelanta a la pregunta del jurado.

### 7. Impacto y siguiente paso — 4:30–5:00
"Next steps: field trials with real agricultural workers, a model trained on field-labour motion, GPS/LoRaWAN gateways for large farms, and a solar-assisted battery. CENTINELA turns a wearable that *measures* into one that *protects*. Thank you."

*Cerrar con la frase central, mirando a los jueces, sin diapositiva de "Questions?".*

---

## Diapositivas (máx. 6, casi sin texto)

1. Imagen potente de un campo + "Who notices when no one is there?"
2. El problema (3 datos, cada uno con fuente — **buscar estadísticas reales** de muertes/accidentes laborales agrícolas y golpe de calor; no inventarlas)
3. Arquitectura (diagrama del PDF, redibujado limpio)
4. Flujo de decisión (el flowchart de `readme.md` §1.1, simplificado: IMU → TinyML → 60 s → LoRa)
5. Resultados del modelo (matriz de confusión, falsos negativos, latencia en ESP32, consumo)
6. Roadmap + equipo + mentor

---

## Preguntas probables del jurado (preparar respuesta de 20 s cada una)

| Pregunta | Respuesta base |
|---|---|
| ¿Por qué TinyML y no un simple umbral? | El umbral resuelve la *inmovilidad sostenida* (y lo usamos). La red aporta en detectar el **patrón de caída/impacto**, que un umbral no separa bien de golpes normales del trabajo. Combinamos ambos. |
| ¿Qué pasa si el trabajador queda inconsciente y no puede cancelar? | Justo ese es el caso diseñado: sin gesto, la alerta sale automáticamente a los 60 s. Cancelar es la excepción, no el requisito. |
| ¿Falsos negativos? ¿Cuántas caídas no detecta? | **Tener la cifra.** Es la métrica que más importa; si no está medida, decir cómo se va a medir. |
| ¿Y si no hay cobertura LoRa / la estación está lejos? | Reintentos + **modo baliza persistente**; LoRa llega a km en campo abierto. Admitir que hay que hacer pruebas de alcance reales. |
| ¿Por qué no un smartwatch comercial? | Costo, cobertura celular/Bluetooth inexistente en campo, y batería de días vs. horas. |
| ¿Privacidad? | Solo movimiento y temperatura; nada de audio/vídeo/ubicación continua; procesamiento local. |
| ¿Está cifrada la comunicación? | **Honesto:** en el prototipo el LoRa P2P viaja en texto plano (el payload es un string compacto; el gateway lo entrega por Ethernet/UDP en base64+JSON, que es *codificación*, no cifrado). Los datos son de bajo riesgo (movimiento, temperatura, zona), pero en producción añadiríamos AES-128-GCM extremo a extremo con contador anti-replay (o LoRaWAN, que lo trae de fábrica). **No decir "encrypted" en las diapositivas.** |
| ¿Costo por unidad / escalabilidad? | *Preparar BOM aproximado en USD.* Una estación base cubre muchos trabajadores (topología en estrella). |
| ¿Cómo se alimenta / cuánto dura? | 18650 + deep sleep con wake por interrupción del APDS-9960. *Tener una medición real de consumo.* |
| ¿Qué aporta la plataforma MYOSA? | ESP32 + sensores del kit como núcleo; añadimos LoRa y la lógica de edge AI. |

---

## ⚠️ Riesgos / pendientes a cerrar ANTES del viaje

Revisando el repo, estas son las brechas que un jurado técnico puede notar:

1. **Resultados del modelo**: `readme.md` §9–10 los deja como pendientes (arquitectura, entrenamiento, validación). Para la final necesitan: matriz de confusión, tasa de falsos negativos, tiempo de inferencia y RAM en el ESP32-S2. Sin números, la diapositiva 5 queda vacía.
2. **Datos propios**: §10.6 dice que el modelo solo con datasets públicos probablemente no generaliza. Grabar el dataset con la pulsera real (ustedes dos + voluntarios, caídas sobre colchoneta).
3. **Inconsistencia propuesta vs. hardware**: la propuesta (PDF) dice **BMP180** y ahora usan **BMP280**; menciona "ESP32" y el esquemático usa **ESP32-S2-WROVER**. Unificar nombres de componentes en las diapositivas.
4. **Bloque LoRa**: el esquemático usa un **Adafruit Feather M0 RFM9x como co-procesador** (SPI vs UART sin confirmar, §2/§9). Decidir y probar de punta a punta; es el eslabón más frágil de la demo.
5. **Divisor de batería** sin confirmar (§8) → el campo `battery_mv` del paquete podría no ser real. Si no está, quitarlo de la demo o añadirlo.
6. **Torre de control web**: en el repo no hay código ni mockup del panel. Es la mitad visual de la demo; necesita existir (aunque sea simple: lista de alertas + zona + severidad).
7. **Logística**: radio LoRa y baterías Li-ion (18650) en avión → revisar normas de equipaje de mano y la **banda de frecuencia legal en la UE (868 MHz)**: el Feather RFM9x debe ser la variante de 868/915 MHz correcta para usarla en Países Bajos. Llevar plan B de video y un segundo prototipo.
8. **Datos de impacto con fuente**: estadísticas de accidentes/golpe de calor en trabajadores agrícolas (OIT/ILO, OMS) para la diapositiva 2.

---

## Cadena de comunicación real (según notas del equipo, verificar contra el código)

```
Pulsera ESP32 ──LoRa P2P (texto plano, claves cortas)──► Gateway Heltec HT-M7603
   "seq=3,r=5.9,p=-2.1,t=33.1,pr=1010,net=…"              │ Semtech UDP Packet Forwarder
                                                          ▼  (Ethernet, IP estática, puerto 1700)
                                             PC: tools/data_recorder.py --lora-listen 1700
                                             → responde PUSH_ACK / PULL_ACK, decodifica base64+JSON,
                                               añade RSSI/SNR del gateway → dashboard / grabador
```

Puntos para explicar en el pitch: no requiere módem ni internet (cable directo gateway↔PC); el gateway mide RSSI/SNR real del enlace; el mismo pipeline sirve para USB, WiFi y LoRa.
Nota: `tools/data_recorder.py` y el gateway Heltec **no están en este repo** y el `readme.md` aún describe un Feather M0 RFM9x; actualizar los docs para que no se contradigan.

---

## Versiones según el tiempo asignado

- **3 min:** quitar §4 a una sola idea (edge AI + ventana de cancelación), demo de 60 s, sin diapositiva 6.
- **7–10 min:** añadir una diapositiva técnica sobre el pipeline (ventana 64 muestras @ 50 Hz, 75 % solape, 8 ventanas consecutivas, int8) y mostrar la matriz de confusión.

---

## Próximos pasos sugeridos

1. Confirmar con los organizadores: duración, idioma, formato de demo, equipamiento disponible (corriente, WiFi, proyector).
2. Fijar la lista de pendientes anteriores con responsable y fecha.
3. Ensayar el pitch completo con cronómetro **al menos 5 veces**, incluyendo una con la demo fallando para practicar el plan B.
