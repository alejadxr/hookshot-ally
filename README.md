<h1 align="center">hookshot-ally</h1>

<p align="center">
  <b>Hookshot for the ROG Ally</b> · <b>Hookshot para la ROG Ally</b><br>
  <a href="GUIDE.md">📘 Guide (English)</a> · <a href="GUIA.md">📗 Guía (español)</a>
</p>

---

## English

An unofficial fork of Hookshot `15646d1` with patches and settings for the **ROG Ally (Ryzen Z1 Extreme, 16 GB,
Windows 11)**:

- worker and shader-preparation threads and priorities;
- a pipeline cache that isn't rewritten over and over;
- an optimized task/mesh fallback;
- swapchain and frame-generation recovery;
- the Ally's gyroscope at full rate.

It comes with the settings file and the Windows and Ally settings we measured. **This is the version played every day on
the Ally.**

> **The game is not included.** No game files, no console keys, no shaders, no caches, no saves. You need your own copy
> of the game, dumped from your own console.

Hookshot is no longer maintained. The same fork on top of its continuation, Longshot, is
[longshot-ally](https://github.com/alejadxr/longshot-ally).

**Quick start:**

1. Build from your copy: on Windows, or with `ally/compilar/compilar.sh` and Docker.
2. Copy the output folder, `ally/hookshot_env.txt` and `ally/windows/lanzar.bat` to the Ally.
3. Run `ally/windows/ajustes_ally.reg`.
4. Set GPU memory to 4 GB and Turbo, then start the game with `lanzar.bat`.

All the details are in the **[guide](GUIDE.md)**.

## Español

Fork no oficial de Hookshot `15646d1` con parches y ajustes para la **ROG Ally (Ryzen Z1 Extreme, 16 GB, Windows 11)**:

- hilos y prioridades de trabajo y de la preparación de shaders;
- una caché de pipelines que no se reescribe una y otra vez;
- un fallback de task/mesh optimizado;
- recuperación de la swapchain y de la generación de fotogramas;
- el giroscopio de la Ally a su frecuencia máxima.

Trae el archivo de ajustes y los ajustes de Windows y de la Ally que medimos. **Es la versión que se juega a diario en la
Ally.**

> **No incluye el juego.** Ni archivos del juego, ni claves de consola, ni shaders, ni cachés, ni partidas. Necesitas tu
> propia copia del juego, sacada de tu propia consola.

Hookshot ya no se mantiene. El mismo fork sobre su continuación, Longshot, es
[longshot-ally](https://github.com/alejadxr/longshot-ally).

**Inicio rápido:**

1. Compila a partir de tu copia: en Windows, o con `ally/compilar/compilar.sh` y Docker.
2. Copia a la Ally la carpeta de salida, `ally/hookshot_env.txt` y `ally/windows/lanzar.bat`.
3. Ejecuta `ally/windows/ajustes_ally.reg`.
4. Pon la memoria de GPU a 4 GB y el modo Turbo, y abre el juego con `lanzar.bat`.

Todos los detalles están en la **[guía](GUIA.md)**.

---

Hookshot's original readme: [README.hookshot.md](README.hookshot.md) · License: GPL-3.0 ([LICENSE](LICENSE)) ·
Hookshot by NotRexed and contributors.
