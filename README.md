# dimmer-220v-esp

Dimmer de **6 canales para luces de 220 V CA** con triacs, control por **ángulo de fase**
sincronizado con un **detector de cruce por cero**. Hay versiones para **ESP8266** y para
**Arduino (AVR, UNO/Nano)**. Proyecto de 2024: secuencias de luces (las 6 a la vez, o una por
una a distintas velocidades).

*English: 6-channel phase-angle dimmer for 220 V AC lamps using triacs and a zero-cross
detector, for ESP8266 and Arduino AVR. 2024 hobby project. No network control: the sketches run
fixed lighting sequences. Docs in Spanish.*

---

## ADVERTENCIA: 220 V PUEDE MATAR

- Este proyecto trabaja con la **tensión de red**. Un error de cableado puede electrocutarte,
  quemar el equipo o iniciar un incendio. Si no tenés experiencia con circuitos de red, **no
  lo armes**.
- El microcontrolador **nunca** se conecta directo a la red: el disparo de cada triac va por
  un **optoacoplador para triac** y el detector de cruce por cero tiene que estar **aislado**
  (optoacoplador). El lado de baja tensión y el de red no comparten masa.
- Trabajá **siempre desconectado** de la red, con la placa en una caja aislante, fusible en la
  línea y triacs dimensionados (con disipador) para la carga.
- No lo uses con cargas inductivas (motores, transformadores) ni con lámparas LED que no sean
  dimerizables: está pensado para cargas resistivas (incandescentes, halógenas).
- Se entrega **sin garantía** (licencia MIT). Lo usás bajo tu responsabilidad.

## Cómo funciona

1. El detector de cruce por cero dispara una interrupción en cada semiciclo y reinicia un
   contador.
2. Un timer interrumpe cada **100 µs** y avanza el contador; con 100 pasos se cubren **10 ms**,
   un semiciclo de 50 Hz.
3. Cada canal tiene un valor `dim[j]` de 0 a 100: cuando el contador llega a ese valor se
   dispara su triac. `dim = 0` → dispara al principio del semiciclo (máximo brillo);
   `dim = 100` → prácticamente no dispara (apagado).
4. El `loop()` cambia los `dim[]` con `millis()` (sin `delay()`) para armar la secuencia.

## Pines

| Placa | Triacs (6 canales) | Cruce por cero | Librería |
|---|---|---|---|
| ESP8266 (NodeMCU / Wemos D1 mini) | GPIO 5, 4, 0, 2, 14, 12 (D1, D2, D3, D4, D5, D6) | GPIO 13 (D7) | `ESP8266TimerInterrupt` |
| Arduino UNO / Nano | D3, D4, D5, D6, D7, D8 | D2 (INT0) | `TimerOne` |

**ESP8266 y "no bootea":** GPIO 0 y GPIO 2 son pines de arranque y tienen que estar en ALTO al
encender. Si el circuito del optoacoplador los tira a masa al energizar, el módulo no arranca.
Revisá cómo quedan esos dos pines en el encendido (y la masa común del lado de baja tensión).

## Sketches

| Carpeta | Placa | Qué hace |
|---|---|---|
| `CRUCE_X_CERO_ESP` | ESP8266 | los 6 canales suben y bajan juntos (paso de 40 ms) |
| `ESP_DIMMER_IGUALES_CRUCE_X_CERO` | ESP8266 | copia idéntica de `CRUCE_X_CERO_ESP` |
| `ESP_SECUENCIA_RAPIDA` | ESP8266 | un canal por vez: sube y baja, y pasa al siguiente (paso de 20 ms) |
| `6_triacs_mismo` | Arduino AVR | los 6 canales juntos (paso de 40 ms) |
| `6triacs_secuencia` | Arduino AVR | un canal por vez (paso de 30 ms) |
| `6_triacs_secuencia_RAPIDA` | Arduino AVR | un canal por vez (paso de 3 ms) |
| `6_tyriac_secuencia_MUY_RAPIDA` | Arduino AVR | un canal por vez (paso de 1 ms) |

Se compilan con el IDE de Arduino o `arduino-cli` (core ESP8266 o AVR, más la librería de timer
de la tabla).

## Limitaciones conocidas (código de 2024, tal cual)

- La interrupción de cruce por cero imprime la frecuencia por `Serial` **desde adentro de la
  ISR** una vez por segundo. Funciona a 9600 baudios en el banco, pero es mala práctica: una
  ISR solo tendría que marcar un flag. Si lo reusás, mové el `Serial.print` al `loop()`.
- `dim[]` no es `volatile` aunque lo lee la ISR del timer.
- No hay control remoto (ni WiFi ni Telegram): las secuencias son fijas en el código.
- El paso de 100 µs × 100 está pensado para 50 Hz; para 60 Hz hay que ajustarlo.

## Licencia

MIT, ver [LICENSE](LICENSE). Hecho por Matías Alegre, Pandemonium (PNDM).
