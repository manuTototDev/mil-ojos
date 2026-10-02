# Mil Ojos

Exoesqueleto con nueve cámaras robóticas alrededor de la cabeza de quien lo porta. Cada cámara va montada en un brazo de servomotores, busca rostros en la multitud y los compara en tiempo real con registros públicos de personas desaparecidas en México.

La pieza parte de un límite humano: nadie puede retener los cientos de rostros que circulan a diario en fichas de búsqueda. El aparato es una prótesis para ese límite. No identifica a nadie ni pretende hacerlo; lo que hace es insistir.

Proyecto desarrollado con el apoyo de **Jóvenes Creadores 2025-2026** (Sistema de Apoyos a la Creación y Proyectos Culturales), especialidad Nuevas Tecnologías.

> **Este repositorio contiene solo código y configuración.** No incluye la base de rostros, fotografías, fichas, nombres ni ningún dato de personas desaparecidas. La base se genera localmente con los scripts de `rnpdno/` y `python/step3_index_faces.py` y queda fuera del control de versiones (ver `.gitignore`).

## Arquitectura (v5)

```
9 cámaras USB ──► sistema.py ──────────────────────────► Arduino ──► 3× PCA9685 ──► 27 servos
                  │ ventana rotativa de 3 cámaras         serial      I2C 0x40 / 0x41 / 0x42
                  │ InsightFace (buffalo_l): detección     $a0..a26,1   9 brazos × 3 servos
                  │ + embedding de 512 dimensiones                      (base, hombro, codo)
                  │ similitud coseno vs. base local
                  │ lazo de seguimiento por brazo
                  └─► interfaz: mosaico de cámaras + panel de coincidencias
```

## Estructura

| Ruta | Qué hace |
|---|---|
| `python/sistema.py` | Sistema completo v5: cámaras, detección, cotejo, seguimiento de los 9 brazos e interfaz |
| `python/calibrar_brazos.py` | Calibración interactiva por brazo: mapeo canal→servo, zona segura (mín/máx/home), servos invertidos |
| `python/emparejar_camaras.py` | Empareja cada cámara con su brazo sacudiendo el brazo y midiendo el desplazamiento de imagen (correlación de fase) |
| `python/generar_header.py` | Genera `arduino/milojos/config_brazos.h` a partir de los JSON de calibración |
| `python/jog_manual.py` | Sliders para mover servos a mano (diagnóstico) |
| `python/init.py` | Sistema v3 (un brazo de seguimiento + tres acompañantes, una cámara) |
| `python/brain_trainer.py` | Autocalibración del lazo de seguimiento a partir de movimientos de prueba |
| `python/step3_index_faces.py`, `step4_webcam_search.py` | Indexado de rostros y prueba de búsqueda con webcam |
| `arduino/milojos/` | Firmware v5: 27 canales en 3 placas, zona segura en firmware, failsafe de 3 s, modo prueba |
| `arduino/calibrador/` | Firmware dedicado a calibración |
| `arduino/arduino.ino` | Firmware v3 (16 canales, una placa) |
| `config/` | Calibración de cada brazo y mapeo cámara↔brazo de la última sesión |
| `rnpdno/` | Scripts para consultar registros públicos y generar la base local |

## Uso

```bash
pip install opencv-python insightface onnxruntime numpy pyserial

python python/calibrar_brazos.py --puerto COM6      # una vez por brazo (con arduino/calibrador)
python python/generar_header.py                      # luego cargar arduino/milojos
python python/emparejar_camaras.py --puerto COM6    # en CADA arranque
python python/sistema.py --puerto COM6 --abiertas 3 --rotacion 8
python python/sistema.py --sin-arduino               # solo visión
```

## Especificación honesta

La similitud máxima observada entre un rostro en vivo y la base ronda el 30-36 %. El sistema no confirma identidades: muestra parecidos.
