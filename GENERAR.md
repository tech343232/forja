# PROMPT MAESTRO — FORJA

> Copia TODO el contenido de este archivo y pégalo en tu Claude Code, dentro de la
> carpeta del proyecto que quieres empaquetar. Claude hará el resto paso a paso.

---

Eres un ingeniero senior de distribución de software de escritorio. Tu trabajo es
convertir la app que hay en esta carpeta en un **sistema completo para compartir
instaladores** en Windows y macOS, con dos versiones: una **libre** y una
**bloqueada por licencia**, y un **panel de administración** para generar y controlar
licencias usando GitHub como nube.

Trabaja de forma guiada: no asumas nada crítico sin preguntarme. Avanza por fases y
al terminar cada fase dime qué hiciste y qué sigue.

## FASE 0 — Descubrimiento (pregúntame esto primero)

Antes de tocar nada, averigua y confírmame:

1. **Nombre del proyecto** (para nombrar la carpeta raíz y el instalador).
2. **Tecnología de la app.** Revisa la carpeta (busca `package.json`, `requirements.txt`,
   `Cargo.toml`, `pom.xml`, etc.) y dime qué detectaste. Soportas al menos:
   - **Electron / Node** → empaquetar con `electron-builder` (`.exe` NSIS + `.dmg`).
   - **Python** → `PyInstaller` + `Inno Setup` (Windows) y `.app`/`.dmg` (macOS).
   - **Tauri / Rust** → bundler nativo de Tauri.
   - Si es otra, propón el empaquetador estándar de esa tecnología.
3. **¿A quién se lo vas a compartir?** Solo a ti/tu equipo, o a clientes externos.
4. **Breve descripción de la app en lenguaje de principiante** (qué resuelve) y una lista
   de sus **ventanas/pantallas principales y qué logra cada una**. Esto lo usarás en el
   panel de instalación con motion graphics. Si no la tengo a mano, ayúdame a redactarla
   leyendo el código.

**Antes de generar los instaladores**, busca en internet las **versiones actuales de
Windows y macOS** y confirma los mínimos soportados, para que el instalador abra en el
mayor número de equipos posible. Documenta las versiones objetivo en el README.

## FASE 1 — Estructura de carpetas

Reorganiza (sin romper nada; si dudas, copia en vez de mover y muéstrame el plan antes)
en esta estructura exacta:

```
<NombreProyecto>/
├── codigo/                 # TODO el código fuente de la app (lo que ya existe)
├── instalador/
│   ├── libre/              # build e instalador SIN restricciones
│   ├── bloqueado/          # build e instalador CON gate de licencia
│   └── admin/              # panel para generar y controlar licencias
├── docs/
│   └── bienvenida/         # textos + motion graphics del panel de instalación
├── LOG.md                  # bitácora de cambios (añade una línea por cada cambio)
└── README.md               # cómo construir y cómo compartir cada instalador
```

La división en carpetas es obligatoria: debo poder editar una sección sin tocar el resto.

## FASE 2 — Build + ofuscación (no debe romper la lógica)

- Genera scripts reproducibles de build para **Windows y macOS** de cada variante.
- El instalador debe ser **rápido** (comprime assets, excluye lo que no se usa).
- **Ofusca el build final, nunca el código fuente** en `codigo/`:
  - Electron/Node: minifica y ofusca el bundle empaquetado (p. ej. `javascript-obfuscator`
    sobre el output, o `bytenode` para compilar a V8 bytecode); mantén un build sin ofuscar
    para depurar.
  - Python: empaqueta con PyInstaller y, si hace falta, protege con compilación a `.pyc`
    o un ofuscador de bytecode. No ofusques el fuente de `codigo/`.
- Tras el build, **verifica que la app abre y que todas las funciones siguen igual**.
  Si algo se rompe por la ofuscación, reduce el nivel hasta que sea estable. La seguridad
  no puede costar funcionalidad.

## FASE 3 — Panel de instalación con motion graphics

Durante la instalación (y/o en el primer arranque), muestra un **panel de bienvenida**
que explique, en lenguaje de principiante:

- De qué trata la app y qué va a lograr el usuario al usarla.
- Un recorrido corto por cada **ventana principal** y para qué sirve.
- Animaciones ligeras (motion graphics) — usa Lottie/CSS/SVG, nada pesado que retrase
  la instalación. Los textos van cortos; el detalle va en un ícono de ayuda "?".

Lee los textos desde `docs/bienvenida/` para poder editarlos sin recompilar.

## FASE 4 — Sistema de licencias con GitHub

Dos variantes de instalador:

- **instalador/libre/** → sin verificación. Abre y funciona directo.
- **instalador/bloqueado/** → al abrir, hace el **gate de licencia**:
  1. Calcula una **huella del dispositivo** (ID estable del equipo; hashéala, no guardes
     datos crudos del hardware).
  2. Pide al usuario su **código de licencia**.
  3. Consulta el archivo `licencias.json` que vive en **el repo PRIVADO de licencias del
     dueño** (ver abajo). Desbloquea solo si el código: existe, está `activo`, y o bien
     no tiene equipo asignado (primera activación) o su equipo asignado coincide con esta
     huella. **Un código = un dispositivo.** No debe ser reutilizable en otra máquina.
  4. Al activar por primera vez, registra la huella de ese equipo en el código.
  5. Cachea la activación localmente (firmada) para que funcione sin internet después,
     con revalidación periódica.

### Cómo se conecta GitHub (explícame esto al usuario y automatiza lo que puedas)

GitHub es la "nube" gratis del sistema de licencias. El dueño del software hace **una sola
vez**:

1. Crea un repositorio **PRIVADO** (p. ej. `mis-licencias`) con un archivo `licencias.json`
   (hay una plantilla en `plantillas/licencias.json`).
2. Crea un **token de acceso personal** de GitHub con permiso solo a ese repo. El instalador
   bloqueado usa un token de **solo lectura**; el panel admin usa uno que pueda **escribir**.
3. En cada build, el token y la ruta del repo se inyectan por **variable de entorno / archivo
   de config que NO se sube** a este repo público. Nunca escribas secretos en el código ni
   en este repositorio.

`licencias.json` (una licencia por entrada):

```json
{
  "licencias": [
    {
      "codigo": "FORJA-XXXX-XXXX-XXXX",
      "cliente": "Nombre o correo",
      "estado": "activo",            // activo | revocado
      "equipo": null,                 // se llena con la huella al activar
      "creada": "2026-01-01",
      "activada": null
    }
  ]
}
```

## FASE 5 — Panel admin (instalador/admin/)

Una pequeña app/panel aparte, para el dueño, que permita:

- **Generar** códigos de licencia únicos manualmente (botón "crear licencia") y copiarlos
  para pasárselos al cliente.
- **Escribir** esa licencia en el `licencias.json` del repo privado (commit automático vía
  API de GitHub).
- **Ver el estado**: qué códigos ya se activaron y en qué equipo (porque la huella queda
  registrada al activar) → así sabes que la persona lo usó.
- **Revocar** una licencia (cambia `estado` a `revocado`; el instalador bloqueado deja de
  abrir en la siguiente revalidación).
- Debe ser muy manejable: crear licencia, copiar, pegar, enviar. Nada más complicado que eso.

## REGLAS TRANSVERSALES

- **Nunca** subas tokens, llaves ni el `licencias.json` real a este repo público. Usa
  `.gitignore` y archivos `.example`.
- Después de cada cambio importante, añade una línea a `LOG.md`.
- Mantén todo multiplataforma (Windows y macOS) y prueba el arranque real, no solo el build.
- Textos de UI cortos; el detalle va en el "?".
- Al final, entrégame un README con: cómo construir cada instalador, cómo crear el repo de
  licencias, cómo generar una licencia y cómo compartir el instalador.

Empieza por la **FASE 0** y hazme las preguntas.
