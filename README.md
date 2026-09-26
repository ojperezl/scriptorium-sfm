# scriptorium_sfm_code

Repositorio de **código** del proyecto **Scriptorium SFM**: preservación del legado del
Seminario de Filosofía de las Matemáticas (23 semestres, 2012-1 a 2024-2, ~290 sesiones).
Cubre la curaduría de material audiovisual, transcripción, publicación de tomos y el
desarrollo de un LLM con RAG.

Este repositorio está **separado de la carpeta de datos** (`scriptorium_sfm/`). El código
accede a los datos únicamente vía la variable de entorno `SFM_DATA_ROOT`. Nunca se usan
rutas hardcodeadas a los datos y nunca se incluyen datos ni credenciales en el repo.

## Estructura

```
common/                  Utilidades compartidas: config, DB (SQLAlchemy), nomenclatura,
                         checksums, logging + registro de ejecuciones.
fase1_setup/             Creación idempotente del árbol de datos e inicialización de la DB.
fase2_catalogacion/      Catalogación local de 01_raw/, validación de integridad y
                         reporte de cobertura.
fase3_procesamiento_av/  (placeholder) Limpieza de audio, mejora de imagen, OCR de pizarras.
fase4_transcripcion/     (placeholder) Transcripción automática (ASR).
fase5_edicion_revision/  (placeholder) Edición y revisión humana de transcripciones.
fase6_dataset_llm/       (placeholder) Construcción de datasets para el LLM / RAG.
fase7_interfaz/          (placeholder) Backend y frontend de consulta.
fase8_multimodal/        (placeholder) Modelo de visión para diagramas de pizarra.
docs/adr/                Registros de decisiones de arquitectura (ADR).
tests/                   Tests con pytest (SQLite en memoria, sin datos reales ni Drive).
```

## Setup

Requiere Python 3.11+.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

### Definir `SFM_DATA_ROOT`

Copia `.env.example` a `.env` y apunta a tu carpeta local de datos:

```bash
cp .env.example .env
# editar .env:
# SFM_DATA_ROOT=/ruta/a/scriptorium_sfm
```

Alternativamente exporta la variable en tu shell:

```bash
export SFM_DATA_ROOT=/ruta/a/scriptorium_sfm
```

Los scripts cargan `.env` automáticamente (python-dotenv); la variable de entorno del
shell tiene prioridad sobre `.env`.

## Uso inicial

```bash
# 1. Crear (idempotente) el árbol de carpetas de datos que falte
python -m fase1_setup.crear_estructura_datos

# 2. Crear las tablas de la base de metadatos si no existen
python -m fase1_setup.init_db

# 3. Catalogar el material crudo ya presente en 01_raw/ (ver fase2_catalogacion/README.md)
python -m fase2_catalogacion.catalogar_raw_local --dry-run
```

## Tests

```bash
pytest
```

Los tests no necesitan Google Drive ni datos reales: usan SQLite en memoria y
fixtures sintéticos.

## Formato y lint

```bash
black .
ruff check .
```

## Convenciones

- **Nomenclatura de archivos de datos**: `{semestre}_{sesion}_{tipo}_{secuencia}_{estado}.{ext}`,
  p. ej. `2017-1_s04_audio_01_raw.mp3`. Todo en minúsculas, sin espacios, sin tildes, sin ñ.
  Los archivos nunca se sobreescriben entre estados: cada cambio de estado genera un archivo nuevo.
- **Base de metadatos**: SQLite en `{SFM_DATA_ROOT}/00_metadata/db/scriptorium.db`, accedida
  siempre vía SQLAlchemy ORM (migración futura a PostgreSQL cambiando la cadena de conexión;
  ver `docs/adr/ADR-002`).
- **Trazabilidad**: todo script que produce artefactos registra en la tabla `ejecucion_log`
  el nombre del script, el commit de git, los parámetros (JSON) y la fecha UTC ISO-8601.
- Rutas siempre con `pathlib.Path`; fechas siempre UTC ISO-8601.
