# Geovisualizador Urbano de Machala

Proyecto STEAM — mapa web interactivo del uso de suelo urbano de Machala.

## Estructura del proyecto

```
geovisualizador-machala/
├── data/
│   ├── raw/            ← archivos originales tal cual los recibiste (no tocar)
│   └── processed/       ← capas convertidas a GeoJSON (WGS84), listas para web
├── web/                 ← esto es lo que se publica en GitHub Pages
│   ├── index.html
│   └── data/            ← copia de los GeoJSON que usa el sitio
├── qgis/                 ← aquí va el .qgz que te falta pedir a tu compañero
├── docs/                 ← memoria técnica, informe de análisis (entregables 1 y 4)
└── README.md
```

Regla simple: **nunca edites nada dentro de `data/raw`**. Si necesitas volver a exportar
o corregir algo, hazlo sobre una copia y guárdala en `data/processed`.

## Qué se hizo con tus archivos

De los 18 archivos que subiste:

- **`machala_uso_suelo.gpkg`** — este es el que realmente importa. Tenía un problema de
  nombres (los archivos `-wal` y `-shm` no calzaban con el nombre del `.gpkg` principal,
  posiblemente por cómo los renombró el sistema al subirlos), lo cual impedía abrirlo
  directamente. Una vez corregido el nombre, adentro había **muchas más capas de las que
  esperábamos** — tu compañero ya avanzó bastante:
  - `lotes_urbanos` (67 lotes: uso de suelo, manzana, habitantes)
  - `red_vial` (46 tramos)
  - `hidrografia` (el estero)
  - `equipamientos` (salud/educación)
  - `areas_verdes`
  - `Lotes_Riesgo_Inundacion` + `buffer_inundacion`
  - `buffer_salud_1000m` / `buffer_educacion_500m` (los buffers de accesibilidad)
  - `PARCELAS` y `PARCELAS CATASTRO` (1247 parcelas catastrales — ver advertencia abajo)
  - Tablas de análisis: `matriz_de_distancia`, `centroides`, `estadsticas_por_categora`

  De ahí exporté a GeoJSON (en `data/processed/`) las 9 capas listas para publicar:
  uso de suelo, red vial, hidrografía, equipamientos, áreas verdes, riesgo de inundación
  y los 3 buffers. Ya están reproyectadas de UTM 17S a WGS84 (lat/lon), que es lo que
  necesita Leaflet para la web.

- **`AREA_DEL_POLIGONO`, `PUNTOS_LOTE`, `VERTICES`, `SPLIT_BY_LENGT_ORIGINAL.gpkg`** —
  estos parecen ser de un ejercicio topográfico aparte (coordenadas de vértices, azimut,
  distancia — como el ejemplo de cálculo de accesibilidad del brief), no del mapa de
  ciudad completo. Confírmalo con tu compañero antes de descartarlos o integrarlos.

- **`Machala.jgw`** — es un archivo de georreferenciación (world file) para una imagen
  raster, pero **la imagen (`Machala.jpg` o `.tif`) no vino incluida**. Solo no sirve de
  nada; pide también el archivo de imagen.

## ⚠️ Importante: datos personales antes de publicar

La tabla `PARCELAS CATASTRO` incluye campos con **nombres de propietarios** y otros datos
personales (`propietari`, `propieta_1/2/3`, `observacio`). Como el proyecto ya toca la
LOPDP, ten esto presente: **no publiques esa tabla tal cual en el mapa web público.**
Antes de usarla, hay que quitar o anonimizar esos campos — para el geovisualizador
probablemente te alcanza con `PARCELAS` (tiene área, uso de suelo, valor y % de
ocupación, sin nombres).

## Calidad de datos a corregir con tu compañero

El campo `uso` de `lotes_urbanos` y `tipo_via` de `red_vial` tienen inconsistencias de
captura: mayúsculas mezcladas (`COMERCIAL` vs `Comercial`), errores de tecleo
(`Princiapl`, `Princiapal`), espacios de más (`Centro de Slud `). El sitio web ya
normaliza esto automáticamente para que la leyenda no se fragmente, pero **lo correcto
es que se limpie en el GeoPackage de origen** antes de la entrega final — si no, tu
informe de análisis (entregable 4) va a tener categorías duplicadas en las estadísticas.

## Cómo publicar en GitHub Pages — paso a paso

1. Crea un repositorio en GitHub (público, para que Pages funcione gratis).
2. Sube el **contenido de la carpeta `web/`** al repositorio (no la carpeta `geovisualizador-machala`
   completa — solo lo que está dentro de `web/`, así `index.html` queda en la raíz del repo).
3. En el repositorio → **Settings → Pages** → Branch: `main`, carpeta `/root` → Save.
4. Espera 1-2 minutos y abre `https://tu-usuario.github.io/nombre-repo/`.
5. Cada vez que actualices un GeoJSON en `web/data/`, solo haz commit y push — no necesitas
   volver a configurar nada.

## Cómo previsualizar localmente antes de subir

No abras `index.html` haciendo doble clic (el navegador bloquea la carga de los GeoJSON
por seguridad, CORS). Usa un servidor local:

- **VS Code**: instala la extensión "Live Server", clic derecho sobre `web/index.html` →
  "Open with Live Server".
- **O por terminal**, dentro de la carpeta `web/`:
  ```
  python3 -m http.server 8000
  ```
  y abre `http://localhost:8000` en el navegador.

## Qué le falta pedir a tu compañero

- [ ] El archivo `.qgz` del proyecto QGIS (simbología, nombres de capas, popups configurados)
- [ ] Confirmar cuál tabla de parcelas es la definitiva (`PARCELAS` vs `PARCELAS CATASTRO`
      vs `PARCELAS CATASTRA` — esta última está vacía, probablemente se puede borrar)
- [x] ~~La imagen raster que acompaña a `Machala.jgw`~~ — recibida. Es un mapa topográfico
      de referencia (Machala–El Guabo–Pasaje, formato IGM), ya integrado en `web/img/` como
      capa de contexto opcional (desactivada por defecto, pesa ~4.3 MB).
- [ ] Limpieza de las inconsistencias de texto en `uso` y `tipo_via`
- [ ] Confirmación sobre `AREA_DEL_POLIGONO`, `PUNTOS_LOTE`, `VERTICES` — ¿son parte del
      mapa de ciudad o son el ejercicio topográfico del entregable de matemáticas?

## Sobre el mapa topográfico (Machala.jpg + .jgw)

Es una cartografía base escaneada y georreferenciada (no vectorial), útil como capa de
**contexto regional** — muestra la relación de Machala con El Guabo, Pasaje y el océano
Pacífico a escala cantonal. No sirve para análisis (no tiene atributos), solo como fondo
visual. Por su peso (~4.3 MB) la dejé **desactivada por defecto** en el sitio; el usuario
la activa si quiere verla. Los archivos `.aux.xml` son metadata de estadísticas de imagen
generada por ArcGIS/QGIS — no aportan nada a la web, quedan solo como respaldo en `data/raw`.
