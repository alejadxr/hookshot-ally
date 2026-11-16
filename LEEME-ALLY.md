# Hookshot para la ROG Ally

Fork no oficial de Hookshot `15646d1` (la rama principal de Hookshot del 11 de noviembre de 2026) con los parches v5 para
la **ROG Ally (Ryzen Z1 Extreme, 16 GB, Windows 11)**. **Es la versión que se juega a diario en la Ally.**

Hookshot ya no se mantiene. Su continuación es Longshot, y el mismo fork sobre Longshot está en
[**longshot-ally**](https://github.com/alejadxr/longshot-ally).

> **No incluye el juego.** Aquí no hay archivos del juego, ni claves de consola, ni shaders, ni cachés, ni partidas. Para
> compilar necesitas tu propia copia del juego sacada de tu consola.

## La guía

La [guía completa está en longshot-ally](https://github.com/alejadxr/longshot-ally/blob/main/LEEME-ALLY.md) y vale para
los dos forks: compilar, instalar, ajustes de Windows y de la Ally, rendimiento medido y lo que no conviene usar.

Lo que cambia en Hookshot:

- **Archivo de ajustes:** se llama `hookshot_env.txt`. Usa [`ally/hookshot_env.txt`](ally/hookshot_env.txt).
- **Parches extra respecto a longshot-ally:**
  - `RX_VK_JOB_WORKERS=<n>`: número de hilos de trabajo. Longshot ya trae el suyo, `RX_VK_WORKERS`.
  - `RX_VK_PREPARE_WORKERS=auto`: todos los hilos menos `RX_VK_PREPARE_SPARE` (2 por defecto) preparan shaders.
  - `RX_VK_DRIVER_FAULTS=3`: experimental. En la Ally dio DEVICE_LOST, así que no lo uses.
- **Compilar con Docker:**
  ```sh
  ally/compilar/compilar.sh /ruta/a/tu-juego.nup ~/HookshotAlly
  ```
  Nosotros la compilamos con `RX_MARCH=x86-64-v3 RX_MTUNE=znver4`.
- **Compilar en Windows sin Docker:** los pasos de la guía, con `HOOKSHOT_MINGW` y `HOOKSHOT_JOBS` en lugar de
  `LONGSHOT_*`.
- **Partidas:** van en `user\save\` dentro de la carpeta del juego. Longshot usa el mismo formato, así que puedes
  llevártelas cuando cambies.
- **Lo quitado:** `.github/` (la CI y las plantillas de issues del repositorio original).

Los dos primeros commits separan el código original de lo nuestro: `git show HEAD~1` enseña exactamente los parches,
que también están sueltos en [`ally/patches/hookshot-ally.diff`](ally/patches/hookshot-ally.diff).

## Licencia y créditos

GPL-3.0 (ver [LICENSE](LICENSE)). Hookshot es de NotRexed y sus colaboradores; este fork solo añade lo descrito arriba.
