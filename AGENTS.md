# Post-Cronos: Córdoba 2060

## Descripción
Proyecto participativo donde niños y adolescentes imaginan y dibujan Córdoba en 2060. Completan un formulario + dibujo, se escanea, se sube a GCS, Gemini extrae datos estructurados, se guarda en BigQuery y se muestra en una galería web.

## Stack
- **Google Cloud Storage** — bucket `post-cronos-dibujos-2060` (imágenes de dibujos)
- **BigQuery** — tabla `laboratorio-cordoba-2060.post_cronos.encuestas`
- **Vertex AI Gemini** — modelo `gemini-2.5-flash-lite` para extraer datos de formularios
- **GitHub Pages** — `index.html` deployado como galería pública

## Pipeline de procesamiento
`process_with_gemini.py` — Lee imágenes de GCS, envía a Gemini, inserta en BigQuery.
- El prompt debe indicar a Gemini que busque **X o tildes** en lugar de inferir
- El campo `genero` admite: `Masculino`, `Femenino`, `No binario`, `Otro`, `No responder`
- El campo `eje` admite **múltiples valores** separados por coma (ej: `"Moda, Tecnología"`)
- Para actualizar géneros existentes en BQ: `update_generos.py`

## Scripts útiles
- `process_with_gemini.py` — procesar nueva tanda de encuestas desde GCS
- `update_generos.py` — corregir géneros en BigQuery
- `update_urls_bq.py` — migrar URLs de gs:// a https://
- `add_genero.py` / `add_eje_column.py` — agregar columnas a la tabla BQ
- `generate_data.py` — lee BigQuery y genera `data.js` (`window.DATA`) para la web
- `analisis_tendencias.py` — manda los datos a Gemini (Vertex AI) y genera `resumen_ia.js` (`window.RESUMEN`) con análisis de tendencias

## Web (`index.html`)
- Estática (GitHub Pages), consume `data.js` y `resumen_ia.js` (ambos generados por scripts)
- Dos vistas: **Dashboard** (gráficos + panel de Tendencias IA + tabla) y **Galería de dibujos**
- Cada entrada de `DATA`: `id, edad, genero, ciudad_barrio, colegio, eje, img, interp, diseno`
- `eje` puede ser múltiple separado por coma — filtro usa `split+includes`
- Filtros por género, eje temático y búsqueda por texto
- Para actualizar tras procesar encuestas: correr `generate_data.py` y `analisis_tendencias.py`, luego commit + push

## Correcciones aplicadas (sesión 2026-07-02)
- Santa Ana: las 7 encuestas (IMG_5334 a 5340) son todas **Femenino**
- Se corrigió `index.html` y BigQuery vía `update_generos.py`
- Se mejoró prompt de Gemini para leer X/tildes en género y ejes
