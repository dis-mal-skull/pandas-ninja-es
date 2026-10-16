# 🥷 Pandas Ninja — edición en español

Juego para **aprender [pandas](https://pandas.pydata.org/)** (la librería de Python para datos)
resolviendo retos con **una sola expresión**. Todo el código Python se ejecuta **dentro de tu
navegador** gracias a [Pyodide](https://pyodide.org/) (Python compilado a WebAssembly), así que
**no necesita backend ni instalar Python**.

> Basado en el proyecto original **[dridk/pandas-ninja](https://github.com/dridk/pandas-ninja)** de
> [@dridk](https://github.com/dridk).
> Esta edición traduce la interfaz y los tutoriales al español y corrige varios errores de la versión original.

---

## ✨ ¿Cómo se juega?

1. Arriba a la derecha ves los **datos de entrada** (el DataFrame `df`).
2. Abajo a la derecha ves los **datos esperados**: lo que debes conseguir.
3. Escribe tu código Python en el editor y pulsa **Ejecutar** (o `Ctrl + Enter`).
4. Cuantas **menos líneas** uses, más estrellas ganas (3 estrellas = 1 sola línea).

Reglas:

- `pandas` ya está importado como `pd`, y los datos están en la variable `df`.
- **No** puedes crear un `DataFrame` ni una `Series` desde cero.
- Se premia el *method chaining* (encadenar métodos en una sola expresión).

---

## 🚀 Correr en local

Necesitas [Node.js](https://nodejs.org/) 18 o superior.

```bash
# 1. Instalar dependencias
npm install

# 2. Arrancar el servidor de desarrollo
npm run dev
```

Abre la URL que aparece en la terminal (normalmente <http://localhost:5173/>).

### Otros comandos

```bash
npm run build     # genera la versión de producción en dist/
npm run preview   # sirve localmente la versión compilada
npm run check     # revisa tipos y accesibilidad (svelte-check)
```

---

## ➕ Crear tus propios retos

Los retos son archivos JSON dentro de `public/challenges/`. Cada reto tiene:

- `title`: nombre que se muestra.
- `file`: nombre del archivo JSON.
- `level`: `easy`, `medium` o `hard`.

Y cada archivo de reto contiene:

```json
{
  "name": "mi reto",
  "author": "@tu_usuario",
  "placeholder": "# Instrucciones para el jugador",
  "input":    [ { "x": 1 }, { "x": 2 } ],
  "expected": [ { "x": 1, "y": 1 }, { "x": 2, "y": 4 } ]
}
```

- `placeholder`: es el texto que aparece en el editor (usa comentarios `#` para las instrucciones).
- `input`: los datos de partida (el DataFrame `df`).
- `expected`: el resultado que el jugador debe obtener.

> El archivo `challenge.py` incluye la clase `Challenge`, una ayuda para generar retos con pandas
> (requiere `pandas` y `simplejson`).

---

## 🗂️ Estructura del proyecto

```
public/
  challenges/          # definición de cada reto (JSON)
  challenges.json      # lista de retos que se cargan en el juego
  shuriken.svg         # favicon
src/
  App.svelte           # lógica principal (carga de retos, ejecución de Python, victoria)
  CodeEditor.svelte    # editor de código (Ace)
  Console.svelte       # panel de salida/consola
  DataFrame.svelte     # tabla que muestra los datos
  ChallengeList.svelte # lista lateral de retos
  Grid.svelte          # distribución de paneles
  Header.svelte        # barra superior
  ...
```

Tecnologías: [Svelte](https://svelte.dev/) + [Vite](https://vitejs.dev/) +
[Tailwind CSS](https://tailwindcss.com/) + [daisyUI](https://daisyui.com) +
[Pyodide](https://pyodide.org/).

---

## ☁️ Desplegar

Es un sitio estático. Compila con `npm run build` y publica la carpeta `dist/`.

Por ejemplo, con [Vercel](https://vercel.com/):

```bash
npx vercel --prod
```

---

## 🙏 Créditos

- Proyecto original: **[dridk/pandas-ninja](https://github.com/dridk/pandas-ninja)** (autor de los
  retos originales y de la idea).
- Traducción al español y correcciones de esta edición: [@dis-mal-skull](https://github.com/dis-mal-skull).

Hecho con cariño para practicar pandas. 🐼
