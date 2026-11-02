# FORJA 🔨

**Convierte cualquier app de escritorio en instaladores listos para compartir — libre y bloqueado por licencia — con un solo prompt.**

FORJA no es una app: es un **generador**. Lo copias, lo pegas en tu Claude Code, y él te
construye todo el sistema de distribución para tu propia aplicación (Windows y macOS):

- ✅ Instalador **libre** (abre y funciona directo).
- 🔒 Instalador **bloqueado** con licencia por dispositivo (un código = un equipo).
- 🛠️ **Panel admin** para generar licencias, pasárselas a tus clientes y ver quién las usó.
- ☁️ Verificación de licencias usando **GitHub** como nube gratis (repo privado + token).
- 🎬 Panel de instalación con **motion graphics** que explica la app al usuario nuevo.
- 🛡️ Build **ofuscado** sin romper la lógica.

---

## Cómo se usa (2 pasos)

1. Abre **tu proyecto** (la app que quieres empaquetar) en Claude Code.
2. Copia **todo** el contenido de [`GENERAR.md`](GENERAR.md) y pégalo en el chat.

Claude empezará preguntándote lo básico (nombre, tecnología, a quién se lo compartes) y
construirá el sistema por fases. Si usas Claude Code con comandos, también puedes copiar
[`.claude/commands/forja.md`](.claude/commands/forja.md) a tu proyecto y escribir `/forja`.

> No necesitas saber programar. FORJA te va guiando y te explica cada paso.

---

## Qué te va a generar

```
<TuProyecto>/
├── codigo/              # tu código fuente (intacto)
├── instalador/
│   ├── libre/           # instalador sin restricciones
│   ├── bloqueado/       # instalador con gate de licencia
│   └── admin/           # panel para crear y controlar licencias
├── docs/bienvenida/     # textos y animaciones del instalador
├── LOG.md
└── README.md
```

---

## El sistema de licencias, en simple

GitHub es tu nube gratis. **Una sola vez** creas un repo **privado** (p. ej. `mis-licencias`)
con un archivo `licencias.json`. A partir de ahí:

| Tú (dueño) | El cliente |
|---|---|
| Abres el **panel admin** y generas un código único | Instala el **instalador bloqueado** |
| Copias el código y se lo envías | Pega el código la primera vez |
| En el panel ves que ya lo activó (y en qué equipo) | La app queda ligada **solo a su equipo** |
| Puedes **revocar** un código cuando quieras | Si lo revocas, deja de abrir |

Un código solo sirve para **un dispositivo**. No es reutilizable en otra máquina.

Hay una plantilla del archivo en [`plantillas/licencias.json`](plantillas/licencias.json).

---

## ⚠️ Seguridad

Este repo es **público**: aquí NUNCA van tokens, llaves ni licencias reales.
Tus secretos (token de GitHub, repo de licencias) viven solo en tu máquina / variables de
entorno y en tu repo **privado** de licencias. El `.gitignore` ya los bloquea.

---

## Preguntas frecuentes

**¿Funciona en Mac y Windows?** Sí, genera instaladores para ambos.

**¿Sirve para cualquier app?** Sí: Electron/Node, Python, Tauri/Rust y otras. FORJA detecta
tu tecnología y elige el empaquetador correcto.

**¿Qué es "ofuscar"?** Proteger el programa final para que sea difícil de copiar. FORJA lo
hace sobre el build, no sobre tu código fuente, y verifica que todo siga funcionando.

**¿GitHub cobra por esto?** No. Un repo privado y un token personal son gratis.
