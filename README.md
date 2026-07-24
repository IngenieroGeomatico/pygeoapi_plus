# pygeoapi_plus

> Fork de [geopython/pygeoapi](https://github.com/geopython/pygeoapi) con extensiones para infraestructuras de datos espaciales (IDE).

[![Build](https://github.com/geopython/pygeoapi/actions/workflows/main.yml/badge.svg)](https://github.com/geopython/pygeoapi/actions/workflows/main.yml)
[![Docker](https://github.com/geopython/pygeoapi/actions/workflows/containers.yml/badge.svg)](https://github.com/geopython/pygeoapi/actions/workflows/containers.yml)

`pygeoapi_plus` mantiene el 100% de la funcionalidad de **pygeoapi** —un servidor Python que implementa la familia de estándares [OGC API](https://ogcapi.ogc.org) (OpenAPI, GeoJSON y HTML)— y le suma un conjunto de **complementos** empaquetados en el submódulo [`pygeoapi_complementos`](https://github.com/IngenieroGeomatico/pygeoapi_complementos): nuevos *providers* vectoriales y ráster, *proxy-providers* para servir fuentes remotas dinámicas, salida en TopoJSON, conectividad IoT e integración de interfaz con **API-IDEE** / OpenLayers.

---

## Índice

- [¿Qué añade este fork?](#qué-añade-este-fork)
- [Instalación](#instalación)
- [Inicializar los complementos](#inicializar-los-complementos)
- [Uso en la configuración](#uso-en-la-configuración)
- [Docker](#docker)
- [Documentación](#documentación)
- [Créditos y licencia](#créditos-y-licencia)

---

## ¿Qué añade este fork?

Los complementos viven en el submódulo `pygeoapi/pygeoapi_complementos` y se cargan como plugins de pygeoapi. Añaden:

| Categoría | Componente | Descripción |
|---|---|---|
| **Provider vectorial** | `OGR_DataProvider` | Acceso a datos vectoriales vía GDAL/OGR (local o remoto): filtros por atributo, `bbox`, selección de capa (`__layer__`), paginación. |
| **Provider vectorial** | `OGCVectorProxyProvider` | *Proxy* que sirve datasets vectoriales remotos definidos dinámicamente por petición (`__url__`), sin fijarlos en el YAML. |
| **Provider vectorial** | `Sonoff_DataProvider` / `Sonoff_DataProvider_OGR` | Conectores IoT (Sonoff/eWeLink, Tuya Smart Life) expuestos como capas vectoriales. |
| **Provider ráster** | `GDALRasterProvider` | OGC API - Coverages sobre GDAL: selección de bandas, `subset` por coordenadas, resolución (`width`/`height`). |
| **Provider ráster** | `GDALProxyRasterProvider` | *Proxy* de coberturas ráster remotas definidas dinámicamente (`__url__`). |
| **Formatter** | `TopoJSON_DataFormatter` | Salida en **TopoJSON** (`application/topo+json`) mediante `f=topojson`. |
| **Proceso** | `bufferDataProcessor` | Cálculo de áreas de influencia (buffer) — OGC API - Processes. |
| **Proceso** | `RandomNumberProcessor` | Proceso de demostración (números aleatorios). |
| **Interfaz** | Plantillas + estáticos | `templates/` y `static/` personalizados con integración **API-IDEE** y OpenLayers. |

La librería base de acceso a datos (PostgreSQL/PostGIS vía GDAL/OGR y conectores IoT Sonoff/Tuya) se encapsula en el subsubmódulo [`pygdal_PG_datasource`](https://github.com/IngenieroGeomatico/pygdal_PG_datasource), con sus módulos de conexión en `conex/` (`PG_conex.py`, `Vector_conex.py`, `Raster_conex.py`, `sonoff_conex.py`, `tuyaSmartLife_conex.py`).

---

## Instalación

### Requisitos

- Python 3.10+
- **GDAL** con sus *bindings* de Python (se instala desde el gestor de paquetes del sistema, no solo con `pip`).
- Acceso a PostgreSQL/PostGIS si vas a usar ese origen de datos.

### Clonado con submódulos

```bash
git clone --recurse-submodules https://github.com/IngenieroGeomatico/pygeoapi_plus.git
cd pygeoapi_plus

# Si ya lo clonaste sin submódulos:
git submodule update --init --recursive
```

### Dependencias

```bash
pip install -r requirements.txt
pip install -e .

# Dependencias de los complementos (GDAL requiere bindings del sistema):
pip install gdal psycopg2-binary topojson
```

---

## Inicializar los complementos

El submódulo `pygeoapi/pygeoapi_complementos` debe estar presente y ser importable. Asegúrate de que la raíz del proyecto esté en el `PYTHONPATH` para que pygeoapi resuelva las rutas de los plugins:

```bash
# Linux / macOS
export PYTHONPATH=$(pwd)

# Windows (PowerShell)
$env:PYTHONPATH = (Get-Location).Path
```

---

## Uso en la configuración

Los plugins se referencian en `pygeoapi-config.yml` por su **ruta completa** dentro del paquete.

### Formatter TopoJSON

```yaml
format:
    topojson:
        mimetype: application/topo+json
        name: pygeoapi.pygeoapi_complementos.Vector_format.topojson_DataFormat.TopoJSON_DataFormatter
        title: TopoJSON
```

### Provider vectorial OGR

```yaml
providers:
    - type: feature
      name: pygeoapi.pygeoapi_complementos.Vector_provider.OGR_DataProvider.OGR_DataProvider
      data: /ruta/al/dato.gpkg
      options:
          collection_id: OGR_1   # asegura el href en los self-links
```

### Rutas completas de los plugins

| Componente | Ruta para `name:` en el config |
|---|---|
| `OGR_DataProvider` | `pygeoapi.pygeoapi_complementos.Vector_provider.OGR_DataProvider.OGR_DataProvider` |
| `OGCVectorProxyProvider` | `pygeoapi.pygeoapi_complementos.Vector_provider.OGRProxy_DataProvider.OGCVectorProxyProvider` |
| `Sonoff_DataProvider` | `pygeoapi.pygeoapi_complementos.Vector_provider.Sonoff_DataProvider.Sonoff_DataProvider` |
| `Sonoff_DataProvider_OGR` | `pygeoapi.pygeoapi_complementos.Vector_provider.Sonoff_DataProvider.Sonoff_DataProvider_OGR` |
| `GDALRasterProvider` | `pygeoapi.pygeoapi_complementos.Raster_provider.GDAL_DataProvider.GDALRasterProvider` |
| `GDALProxyRasterProvider` | `pygeoapi.pygeoapi_complementos.Raster_provider.GDALProxy_DataProvider.GDALProxyRasterProvider` |
| `TopoJSON_DataFormatter` | `pygeoapi.pygeoapi_complementos.Vector_format.topojson_DataFormat.TopoJSON_DataFormatter` |
| `bufferDataProcessor` | `pygeoapi.pygeoapi_complementos.procesos.areaDeInfluencia.bufferDataProcessor` |
| `RandomNumberProcessor` | `pygeoapi.pygeoapi_complementos.procesos.numeros_aleatorios.RandomNumberProcessor` |

### Arrancar el servidor

```bash
export PYGEOAPI_CONFIG=pygeoapi-config.yml
export PYGEOAPI_OPENAPI=openapi.yml

pygeoapi openapi generate $PYGEOAPI_CONFIG --output-file $PYGEOAPI_OPENAPI
pygeoapi serve
```

---

## Docker

El `Dockerfile` incluido construye una imagen completa de pygeoapi con librerías para todos los *providers* (GDAL, rasterio, psycopg2, etc.) y arranca con gunicorn:

```bash
docker build -t pygeoapi_plus .
docker run -p 5000:80 pygeoapi_plus
```

> Para que los complementos estén disponibles dentro de la imagen, asegúrate de haber inicializado el submódulo **antes** de construir (`git submodule update --init --recursive`), ya que el `Dockerfile` copia el árbol del proyecto con `ADD . /pygeoapi`.

---

## Documentación

- pygeoapi (base): <https://docs.pygeoapi.io>
- Complementos: [README de `pygeoapi_complementos`](https://github.com/IngenieroGeomatico/pygeoapi_complementos)
- Acceso a datos: [`pygdal_PG_datasource`](https://github.com/IngenieroGeomatico/pygdal_PG_datasource)

---

## Créditos y licencia

Este proyecto es un fork de [pygeoapi](https://pygeoapi.io), desarrollado por la comunidad de geopython y publicado bajo licencia [MIT](LICENSE.md). Las extensiones de este fork son mantenidas por [IngenieroGeomatico](https://github.com/IngenieroGeomatico) y se distribuyen bajo la misma licencia.
