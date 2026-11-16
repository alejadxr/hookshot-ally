# Hookshot para la ROG Ally: guía

[English](GUIDE.md) · **Español**

Fork no oficial de Hookshot `15646d1` (la rama principal de Hookshot del 11 de noviembre de 2026) con parches y ajustes
para jugar en una **ROG Ally (Ryzen Z1 Extreme, 16 GB, Windows 11)**. **Es la versión que se juega a diario en la Ally**,
y todo lo de aquí se midió en una ROG Ally RC71L.

Hookshot ya no se mantiene. Su continuación es Longshot, y el mismo fork sobre Longshot es
[longshot-ally](https://github.com/alejadxr/longshot-ally).

> **No incluye el juego.** En este repositorio no hay archivos del juego, ni claves de consola, ni shaders, ni cachés, ni
> partidas. Para compilarlo necesitas tu propia copia del juego, sacada de tu propia consola.

## Índice

1. [Qué trae este fork](#1-qué-trae-este-fork)
2. [Requisitos](#2-requisitos)
3. [Inicio rápido](#3-inicio-rápido)
4. [Compilar](#4-compilar)
5. [Instalar en la Ally](#5-instalar-en-la-ally)
6. [El archivo de ajustes](#6-el-archivo-de-ajustes)
7. [Ajustes de Windows y de la Ally](#7-ajustes-de-windows-y-de-la-ally)
8. [Ajustes del menú del juego](#8-ajustes-del-menú-del-juego)
9. [Qué rendimiento esperar](#9-qué-rendimiento-esperar)
10. [Problemas y soluciones](#10-problemas-y-soluciones)
11. [Ajustes que salieron peor](#11-ajustes-que-salieron-peor)
12. [Pasar a Longshot](#12-pasar-a-longshot)
13. [Desinstalar](#13-desinstalar)
14. [Licencia y créditos](#14-licencia-y-créditos)

## 1. Qué trae este fork

| Repositorio | Base | Estado |
|---|---|---|
| **hookshot-ally** (este) | Hookshot `15646d1` (11 nov 2026) + los parches de la Ally (v5) | La versión que se juega a diario en la Ally. |
| [longshot-ally](https://github.com/alejadxr/longshot-ally) | Longshot `bbe7a07` (15 nov 2026) + los mismos parches | Compila sin errores; todavía no se ha probado jugando en la Ally. |

### Los parches

| Cambio | Archivo | Cómo se activa | Por qué |
|---|---|---|---|
| Hilos de trabajo a prioridad normal | `gpu/vk/vk_jobs.cpp` | `RX_VK_JOB_PRIO=normal` | Iban por debajo de lo normal y, al entrar en una zona nueva, otros programas les quitaban CPU. |
| Número de hilos de trabajo | `gpu/vk/vk_jobs.cpp` | `RX_VK_JOB_WORKERS=<n>` | Hookshot usaba 4 y no se podía cambiar. En los 16 hilos de la Ally usamos 6. |
| Hilos y prioridad de la preparación de shaders | `gpu/vk/vk_draw.cpp` | `RX_VK_PREPARE_WORKERS=auto` y `RX_VK_PREPARE_PRIO=high` | Hookshot usaba la mitad de las CPUs para la preparación del primer arranque, que en la Ally tardaba 20–45 minutos. `auto` usa todos los hilos menos `RX_VK_PREPARE_SPARE` (2 por defecto). |
| Guardado de la caché de pipelines | `gpu/vk/vk_draw.cpp` | siempre | En juego, el archivo de la caché (~900 MB) se reescribía entero unas 15 veces por sesión. Ahora solo se guarda cuando ha crecido bastante. |
| Fallback de task/mesh optimizado | `gpu/vk/vk_draw.cpp` | siempre (`RX_VK_FALLBACK_NOOPT=1` lo apaga) | Sin optimizador, la geometría del mundo costaba ~21 ms de GPU por fotograma en la 780M. |
| Superficie nueva si la swapchain sigue fallando | `gpu/vk/vk_main.cpp` | siempre | Al pulsar F11 dos veces la imagen se quedaba congelada (más de 400 reintentos). |
| Fallback de present en la generación de fotogramas | `gpu/vk/vk_framegen.cpp` | siempre | La pantalla se congelaba mientras el juego seguía (pasó tocando la ocarina). |
| Giroscopio de la Ally a 10 ms | `host/hos_imu_rate.c` | siempre (solo Windows) | Windows solo mandaba las lecturas del sensor Bosch al pasar un umbral: al apuntar despacio se movía a saltos y el juego apagaba el giroscopio cada pocos segundos. |
| Cuarentena de fallos del driver sin serializar | `gpu/vk/vk_draw.cpp` | `RX_VK_DRIVER_FAULTS=3` | **Experimental, no lo uses:** en la Ally dio DEVICE_LOST. |

### Qué hay en el repositorio

- `GUIDE.md` y `GUIA.md`: esta guía, en inglés y en español.
- `ally/hookshot_env.txt`: el archivo de ajustes de la Ally ([apartado 6](#6-el-archivo-de-ajustes)).
- `ally/compilar/`: la compilación con Docker para Linux y macOS.
- `ally/windows/`: un lanzador con prioridad más alta y los ajustes del registro.
- `ally/patches/hookshot-ally.diff`: los parches en un solo archivo.
- `README.hookshot.md`: el readme original de Hookshot.

El código original y el nuestro van en commits separados: `git show 75422e5` enseña exactamente los parches. Lo único
que se quitó de Hookshot es `.github/`, la CI y las plantillas de issues del repositorio original.

## 2. Requisitos

- **Una ROG Ally o Ally X** con Ryzen Z1 o Z1 Extreme. También valen otros chips con AVX2: mira `RX_MARCH` en el
  [apartado 4](#4-compilar).
- **Windows 11** con el **driver AMD 32.0.32015 (octubre de 2026) o más nuevo**.
- **Tu propia copia del juego**, en la versión que soporta Hookshot: programa `0400486024DEC000`, update `v131072`, build
  `e0f3ccfa`. Puede ser un archivo `.nup`, o las carpetas `romfs` y `exefs` (`--romfs` y `--exefs` en lugar de `--nup`).
  Añade `--check` al comando de compilar para comprobar tu copia sin compilar nada.
- **Espacio en disco:** la carpeta del juego ocupa ~26 GB, más la carpeta de trabajo de la compilación. Deja al menos
  40 GB libres.
- **Dónde compilar:** un PC con Windows (vale la propia Ally), o Linux o macOS con Docker.

## 3. Inicio rápido

1. Compila el juego a partir de tu copia ([apartado 4](#4-compilar)).
2. Copia la carpeta de salida a la Ally, por ejemplo a `C:\Games\Hookshot`.
3. Copia `ally/hookshot_env.txt` y `ally/windows/lanzar.bat` a esa carpeta, junto a `image.bin`.
4. Haz doble clic una vez en `ally/windows/ajustes_ally.reg` y reinicia Windows.
5. En Armoury Crate, pon la memoria de GPU a 4 GB y el modo Turbo. Enchufa el cargador.
6. Abre el juego con `lanzar.bat`. La primera vez prepara los shaders y tarda un rato.

## 4. Compilar

La compilación lee tu copia del juego y genera la carpeta del juego para Windows.

### Opción A: en Windows (la propia Ally u otro PC)

Hazlo en un Símbolo del sistema (`cmd`). Son los pasos de Hookshot para compilar desde el código, con este repositorio:

1. Instala [Python 3](https://www.python.org/downloads/) y [Git](https://git-scm.com/download/win).
2. Clona el repositorio e instala los paquetes de Python:
   ```bat
   git clone https://github.com/alejadxr/hookshot-ally.git
   cd hookshot-ally
   py -m pip install -r requirements.txt
   ```
3. Descarga `llvm-mingw-20261104-ucrt-x86_64.zip` de
   [mstorsjo/llvm-mingw](https://github.com/mstorsjo/llvm-mingw/releases/tag/20261104). Descomprímelo dentro del clon y
   cambia el nombre de la carpeta a `llvm-mingw`, de modo que exista `llvm-mingw\bin\clang.exe`.
4. Descarga el paquete `cpm` para Windows (las cabeceras de Vulkan y SPIR-V, VMA y FFmpeg). La descarga propia de
   Hookshot desapareció con su repositorio, pero Longshot mantiene el mismo paquete con dav1d añadido:
   [cpm-windows-x64](https://git-cdn.magicbean.dev/api/packages/MagicBeanDev/generic/cpm-windows-x64/hookshot-v2.1.0-dav1d-1.5.4/4eeae3a8d50118dd5c15f2c18dcdc452fbf1dfafceeffcad3bca97aadffe829b-cpm.zip).
   Descomprímelo dentro del clon, de modo que exista `cpm\vulkanheaders`.
5. Compila:
   ```bat
   set HOOKSHOT_MINGW=%CD%\llvm-mingw
   set RX_CPM=%CD%\cpm
   py -I nx2/make_game.py --nup "C:\ruta\a\tu-juego.nup" --out "C:\Games\Hookshot"
   ```
   En la propia Ally no hace falta nada más: la compilación elige la CPU sola. Si compilas en **otro** PC para la Ally,
   antes pon también `set RX_MARCH=x86-64-v3` (es lo que usamos nosotros).

### Opción B: Linux o macOS con Docker (lo que usamos nosotros)

```sh
git clone https://github.com/alejadxr/hookshot-ally.git
cd hookshot-ally
ally/compilar/compilar.sh /ruta/a/tu-juego.nup ~/HookshotAlly
```

El script descarga llvm-mingw y `cpm` y comprueba su SHA-256, crea la imagen de Docker y compila el juego para Windows.
Las opciones están al principio de [`ally/compilar/compilar.sh`](ally/compilar/compilar.sh):

- `RX_MARCH`: la CPU para la que se compila el juego. Aquí, por defecto, es `x86-64-v3`, que funciona en cualquier CPU
  con AVX2 (todas las ROG Ally) y es la compilación que se juega a diario. `znver4` apunta justo a los Z1 y Z1 Extreme,
  pero no lo hemos medido con Hookshot.
- `DOCKER_MEM`: la memoria del contenedor. En una Mac Pro con `DOCKER_MEM=44g` y 22 hilos tardó 1 h 50 min más o menos.
- `WORK`: la carpeta de trabajo (por defecto `<salida>-work`). Consérvala: la siguiente compilación es mucho más rápida.

Docker Desktop en Mac arranca con poca memoria: súbela en sus ajustes. En un Mac con Apple Silicon el contenedor corre
emulado en x86 y tarda mucho más.

## 5. Instalar en la Ally

1. Copia la carpeta de salida entera a la Ally, por ejemplo a `C:\Games\Hookshot`.
2. Copia [`ally/hookshot_env.txt`](ally/hookshot_env.txt) a esa carpeta, junto a `image.bin`. Si la compilación ya creó
   un `hookshot_env.txt`, añade nuestras líneas al final en lugar de sustituirlo.
3. Copia [`ally/windows/lanzar.bat`](ally/windows/lanzar.bat) a la misma carpeta y abre el juego con él. Así arranca con
   prioridad por encima de lo normal.
4. **La primera vez prepara los shaders.** Tarda un rato y es normal. Las siguientes veces arranca rápido.
5. **Para actualizar** a una compilación nueva de este fork: copia solo `hookshot.exe` y `hookshot_vk.dll` encima de los
   viejos. Si copias también `hookshot_shaders.bin`, el juego vuelve a preparar los shaders.

### Partidas

Las partidas van en `user\save\` dentro de la carpeta del juego. Haz una copia de esa carpeta antes de actualizar o de
mover el juego. Longshot usa el mismo formato, así que te las puedes llevar cuando cambies.

## 6. El archivo de ajustes

`hookshot_env.txt` va en la carpeta del juego, junto a `image.bin`. El juego lo lee al arrancar, así que tras cambiarlo
reinicia el juego.

- **Formato:** un `NOMBRE=VALOR` por línea. Las líneas que empiezan por `#` no cuentan.
- **Sin comentarios al final de una línea:** formarían parte del valor.
- **Prioridad:** si una variable ya está puesta en el entorno, gana sobre el archivo.

| Línea | Qué hace |
|---|---|
| `RX_VK_PRECOMPILE=0` | No compila al arrancar las pipelines de todas las zonas: 13 GB de memoria comprometida al arrancar en vez de 20, sin paginar, y 80 °C en vez de 96 °C. El precio es un tirón corto (300–500 ms) la primera vez que ves algo nuevo. |
| `RX_VK_PCACHE_LOAD=0` | No carga en RAM la caché de pipelines del juego (~900 MB); se queda la caché en disco del propio driver AMD. La RAM libre en juego pasó de ~30 MB a 2–3,6 GB. |
| `RX_VK_KEEP_RETIRED=0` | Suelta la memoria importada que el juego ya no usa. |
| `RX_VK_IMPORT_HIGH=85` | Desaloja los imports ociosos cuando el heap de imports llega al 85 %. Con 16 GB y sin estas dos líneas, la imagen acaba congelada (`present/submit failed (-2)`). |
| `RX_VK_VRAM_MB=2560` | El presupuesto de texturas. **No lo subas a 3072:** dio DEVICE_LOST y pantallazo azul `0x119` al cargar zonas. |
| `RX_VK_PASS_BARRIER=game` | Unos 10 % menos de trabajo de GPU. |
| `RX_VK_MESH_OPT=1` | Optimizador de mesh shaders. Necesita el driver AMD de octubre de 2026 o más nuevo. |
| `RX_VK_NO_READBACK_WAIT=1` | No espera a las lecturas de la GPU. |
| `RX_VK_MESH_NOCULL=2c56dc1006942f74` | El arreglo de Hookshot para el cuelgue en el Bosque Kokiri con GPUs AMD de portátil (`README.hookshot.md`, Steam Deck). |
| `RX_VK_JOB_WORKERS=6` | (parche) Seis hilos de trabajo. |
| `RX_VK_JOB_PRIO=normal` | (parche) Hilos de trabajo a prioridad normal. |
| `RX_VK_PREPARE_WORKERS=auto` | (parche) En el primer arranque preparan shaders todos los hilos menos dos. |
| `RX_VK_PREPARE_PRIO=high` | (parche) Preparación de shaders con prioridad alta. |

**En una ROG Ally X (24 GB)** sobra más RAM: puedes probar a borrar `RX_VK_PRECOMPILE=0` y `RX_VK_PCACHE_LOAD=0` para tener
menos tirones. No lo hemos probado.

## 7. Ajustes de Windows y de la Ally

Esto cambia el rendimiento más que cualquier parche:

- **Driver AMD 32.0.32015 (octubre de 2026) o más nuevo.** Con el de mayo, los mesh shaders colgaban la GPU al terminar
  la cinemática del principio (DEVICE_LOST). Si tienes que quedarte con un driver viejo, añade
  `RX_VK_TASK_MESH_FALLBACK=1` y quita `RX_VK_MESH_OPT=1` del `hookshot_env.txt`.
- **Memoria de GPU a 4 GB en Armoury Crate.** Con 16 GB de RAM, si le das más a la GPU el juego empieza a tirar del disco
  y va a tirones. Con 4 GB, Windows ve ~12 GB.
- **[`ally/windows/ajustes_ally.reg`](ally/windows/ajustes_ally.reg)** (doble clic y reinicia) hace dos cosas:
  - apaga HAGS (la programación de GPU acelerada por hardware), que dio un pantallazo azul `0x119` del driver AMD;
  - le da a la GPU 10 s en vez de 2 antes de que Windows la reinicie.

  `deshacer_ajustes_ally.reg` deja las dos cosas como estaban.
- **Modo Turbo en Armoury Crate y siempre enchufada.** Con batería el chip baja a 14–25 W.
- **En la tele con un hub USB-C**, la Ally se queda en ~25 W diga lo que diga el perfil. Pon un cargador de 65 W o más
  (mejor 100 W) en el puerto PD del hub.
- **Escritorio de Windows a 1920×1080.** Si está a otra resolución, el juego sale a esa y la estira.
- **Cierra la Xbox Game Bar y lo que tengas en segundo plano.** La RAM va muy justa.

## 8. Ajustes del menú del juego

Lo que mejor nos fue:

- salida a 1920×1080;
- render al 50–58 % con FSR 3.1;
- generación de fotogramas del juego activada;
- filtrado anisótropo 8x.

La AFMF del driver AMD se nota peor que la generación de fotogramas del propio juego: déjala apagada.

## 9. Qué rendimiento esperar

Medido con esta versión y los ajustes de arriba:

- **Enchufada en Turbo (30–45 W):** 32–37 fps reales en zonas normales y 20–26 en las más cargadas (~2000 draws). La
  generación de fotogramas lo dobla. El límite suele ser la GPU (la 780M a 2,4–2,6 GHz).
- **Con hub a 25 W, a 960×540:** 21–36 fps reales.
- **Al entrar en una zona nueva:** un tirón corto mientras compila sus pipelines. Es el precio de `RX_VK_PRECOMPILE=0`.

## 10. Problemas y soluciones

| Qué pasa | Por qué | Qué hacer |
|---|---|---|
| El juego se cierra nada más abrirlo | Se compiló para una CPU más nueva que la tuya | Vuelve a compilar con `RX_MARCH=x86-64-v3`. |
| El primer arranque tarda muchísimo | Está preparando los shaders | Normal, solo la primera vez. |
| Un tirón corto en cada zona nueva | Compila las pipelines de esa zona (`RX_VK_PRECOMPILE=0`) | Normal. |
| La imagen se congela mientras el juego sigue | Se acaba la memoria para imports | Revisa `RX_VK_KEEP_RETIRED=0`, `RX_VK_IMPORT_HIGH=85` y la memoria de GPU a 4 GB, y cierra lo que tengas en segundo plano. |
| DEVICE_LOST al terminar la cinemática del principio | Driver AMD viejo y mesh shaders | Actualiza el driver, o usa `RX_VK_TASK_MESH_FALLBACK=1` (ver apartado 7). |
| Pantallazo azul `0x119`, o DEVICE_LOST al cargar una zona | Presupuesto de texturas demasiado alto, o HAGS encendido | `RX_VK_VRAM_MB=2560` y `ajustes_ally.reg`. |
| DEVICE_LOST al cargar la partida | `RX_VK_DRIVER_FAULTS=3` | Quita esa línea. |
| Se queda en ~25 W con un hub | El cargador no da bastante | 65 W o más en el puerto PD del hub. |
| Imagen borrosa o estirada | El escritorio de Windows no está a 1920×1080 | Pon el escritorio a 1920×1080. |
| Algunas texturas salen negras un momento | El presupuesto de texturas (2560 MB) está lleno | Es lo que hay. Subirlo provoca los cierres de arriba. |

El juego escribe su log en `hookshot_log.txt`, en la carpeta del juego (solo de la última vez que lo abriste): adjúntalo
si informas de un problema.

## 11. Ajustes que salieron peor

| Ajuste | Qué pasó |
|---|---|
| `RX_VK_VRAM_MB=3072` | DEVICE_LOST y pantallazo azul `0x119` al cargar zonas o partidas. Con 2560 no ha vuelto a pasar. |
| `RX_VK_DRIVER_FAULTS=3` | DEVICE_LOST al cargar la partida. |
| `RX_VK_RECORDER=thread` | El replay de CPU subió a 38 ms por fotograma. |
| `RX_VK_PER_QUEUE_FENCES=0` + `RX_VK_UPLOAD_BUDGET_MS=8` + `RX_VK_TEX_CAPTURE=0` + `RX_VK_MIRROR_MB=256` | Menos de 1 fps. |
| `RX_VK_UPLOAD_BUDGET_MS=16` | Texturas que parpadean. |
| `RX_VK_IMAGE_BLOCK_MB=64` | No ahorró memoria. |
| La AFMF del driver AMD | Se nota peor que la generación de fotogramas del juego. |

## 12. Pasar a Longshot

Hookshot ya no recibe actualizaciones. Cuando quieras cambiar:

1. Compila [longshot-ally](https://github.com/alejadxr/longshot-ally) con la misma copia del juego. Lleva los mismos
   parches.
2. Copia allí tu carpeta `user\save`. El formato es el mismo.

Su archivo de ajustes se llama `longshot_env.txt`. Las líneas son las mismas salvo `RX_VK_JOB_WORKERS` y
`RX_VK_PREPARE_WORKERS`, que Longshot ya gestiona solo.

## 13. Desinstalar

1. Haz una copia de `user\save\` de la carpeta del juego.
2. Borra la carpeta del juego. Las cachés van dentro.
3. Para deshacer los ajustes del registro, haz doble clic en `ally/windows/deshacer_ajustes_ally.reg` y reinicia.

## 14. Licencia y créditos

GPL-3.0, como Hookshot (ver [LICENSE](LICENSE)).

- Hookshot es de NotRexed y sus colaboradores. Este fork solo añade lo descrito arriba.
- Es un fork no oficial: los problemas de estos parches van aquí.
