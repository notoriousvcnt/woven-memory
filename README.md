# woven-memory

Procesador de audio en vivo hecho con [Pure Data](https://puredata.info/) para Raspberry Pi. Toma cuatro entradas de audio, aplica a cada una una cadena de efectos (delay, vocoder, reverb y pitch shift) y mezcla el resultado en una salida estéreo.

▶️ **[Ver demo](https://vimeo.com/1112075305)**

## Cómo funciona

```
adc~ 1 ─┐                         ┌─ delay~ ──────┐
adc~ 2 ─┤  cada entrada alimenta  ├─ vocoder~ ────┤
adc~ 3 ─┤  su propia cadena  ───► ├─ reverb~ ─────┼─► busMixer~ ─┐
adc~ 4 ─┘                         └─ pitchshift~ ─┘              │
                                                    (x4 cadenas) ├─► busMixer~ (master) ─► dac~ 3 4
                                                                 ┘
```

- Cada una de las 4 entradas (`adc~ 1` a `adc~ 4`) tiene su cadena de efectos en paralelo.
- Cada cadena termina en un `busMixer~` con fader, mute y vúmetro.
- Un `busMixer~` maestro suma las cuatro cadenas y las envía a `dac~ 3 4`.
- Los parámetros de cada efecto se guardan en el propio parche (`#A saved ...`), así que el estado se conserva entre sesiones.

## Estructura del repositorio

```
woven-memory/
├── puredata/
│   ├── _MAIN.pd               # Parche principal (4 entradas → 4 cadenas → salida)
│   ├── soundchain~_test.pd    # Banco de pruebas con audio de muestra
│   ├── delay~.pd              # Delay con feedback y mezcla dry/wet
│   ├── combDelay~.pd          # Delay tipo filtro peine
│   ├── reverb~.pd             # Reverb (roomsize, damping, width, freeze, bypass)
│   ├── vfreeverb~.pd          # Freeverb en Pd vanilla (Katja Vetter, 2012)
│   ├── pitchshift~.pd         # Pitch shift granular
│   ├── pitchShiftGrain~.pd    # Grano individual usado por pitchshift~
│   ├── vocoder~.pd            # Vocoder basado en FFT
│   ├── noiseRamp~.pd          # Rampas de ruido (versión señal)
│   ├── noiseRamp.pd           # Rampas de ruido (versión control)
│   ├── busMixer~.pd           # Fader + mute + vúmetro
│   ├── speech.wav             # Audio de prueba (voz)
│   └── piezo.wav              # Audio de prueba (piezo)
└── archivos-raspi/
    └── autostart/
        ├── qjackctl.desktop   # Abre QjackCtl al iniciar la Pi
        └── pd.desktop         # Abre Pd con _MAIN.pd tras 10 s
```

## Requisitos

- [Pure Data](https://puredata.info/downloads) (vanilla)
- La librería externa **cyclone** (se usan `cyclone/edge~`, `cyclone/equals~`, `cyclone/cartopol~`, etc.). Se instala desde *Ayuda → Buscar externals* en Pd.
- [JACK](https://jackaudio.org/) y, opcionalmente, QjackCtl para enrutar el audio.
- Una interfaz de audio con al menos 4 entradas y 4 salidas (el parche usa `adc~ 1-4` y `dac~ 3 4`). Ajusta los números de canal en `_MAIN.pd` según tu hardware.

## Uso

### En un computador (pruebas)

1. Abre `puredata/_MAIN.pd` en Pd. Mantén los demás `.pd` en la misma carpeta, porque son abstracciones que se cargan por nombre.
2. Activa el DSP (*Media → Audio ON*).
3. Para probar sin hardware de entrada, abre `puredata/soundchain~_test.pd`, que reproduce `speech.wav` y `piezo.wav` por la cadena de efectos.

### En la Raspberry Pi

1. Instala Pd, JACK, QjackCtl y cyclone.
2. Copia el contenido del repositorio a la Pi. Los autostart esperan el parche en `/home/vcnt/pi/Londres38/_MAIN.pd`. Si usas otra ruta o usuario, edita `pd.desktop`.
3. Copia los `.desktop` a `~/.config/autostart/`:
   ```bash
   mkdir -p ~/.config/autostart
   cp archivos-raspi/autostart/*.desktop ~/.config/autostart/
   ```
4. Al reiniciar, se abre QjackCtl y, 10 segundos después, Pd corre en modo tiempo real sobre JACK:
   ```bash
   pd -rt -jack /ruta/a/_MAIN.pd
   ```

## Notas

- Los nombres con `~` indican abstracciones que procesan señal de audio.
- Los parámetros de `delay~`, `reverb~`, `pitchshift~` y `vocoder~` se pueden ajustar con los sliders de cada abstracción.
- `vfreeverb~.pd` es obra de Katja Vetter (2012). Conserva los créditos si la redistribuyes.

## Licencia

Pendiente de definir. Si quieres que otras personas puedan reutilizar el código, añade un archivo `LICENSE`.
