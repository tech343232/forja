# Configurar GitHub como nube de licencias (una sola vez)

Esto lo hace **el dueño del software**, no el cliente.

## 1. Crea el repo privado de licencias
- En GitHub: **New repository** → nombre `mis-licencias` → **Private** → Create.
- Sube el archivo `licencias.json` (copia la plantilla de `plantillas/licencias.json`).

## 2. Crea un token de acceso
- GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
- **Repository access:** solo `mis-licencias`.
- **Permisos:** `Contents` → *Read and write* (para el panel admin).
  - Para el instalador bloqueado basta *Read-only*; puedes crear un segundo token solo-lectura.
- Copia el token (empieza con `github_pat_...`). **Guárdalo; no lo subas a ningún lado.**

## 3. Guarda la config localmente (fuera del repo público)
Crea un archivo `.env` en tu máquina (ya está en `.gitignore`):

```
GITHUB_REPO=tu-usuario/mis-licencias
GITHUB_TOKEN_ADMIN=github_pat_xxx     # panel admin (lee y escribe)
GITHUB_TOKEN_LECTURA=github_pat_yyy   # instalador bloqueado (solo lee)
```

El panel admin y el build del instalador bloqueado leen estos valores de aquí.
**Nunca** escribas el token dentro del código ni en el repo público de FORJA.

## 4. Listo
Desde el panel admin ya puedes generar licencias, enviarlas y ver quién las activó.
