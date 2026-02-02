# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QGIS plugin for importing LiDAR DSM (Digital Surface Model) data from the Campania region (Italy). Users select geographic quadrants from a boundary layer, and the plugin downloads raster tiles directly from http://sit.cittametropolitana.na.it/ and merges them into a GDAL Virtual Raster (VRT).

## Development Commands

```bash
# Compile Qt resources after modifying resources.qrc
pyrcc5 -o resources.py resources.qrc

# Run linting
pylint --rcfile=pylintrc *.py

# Build documentation (from help/ directory)
cd help && make html
```

## Architecture

**Plugin Entry Flow:**
1. `__init__.py::classFactory()` → instantiates `LidarCampania`
2. `lidar_campania.py::initGui()` → creates toolbar/menu actions
3. `lidar_campania.py::run()` → opens `LidarCampaniaDialog`

**Key Files:**
- `lidar_campania.py` - Main plugin class, QGIS lifecycle management
- `lidar_campania_dialog.py` - Dialog with download/merge logic
- `lidar_campania_dialog_base.ui` - Qt Designer UI definition
- `quadro_unione_lidar_dsm.shp` - Bundled shapefile with LiDAR tile boundaries

**Data Flow in Dialog (`lidar_campania_dialog.py`):**
1. `on_pushButton_2_pressed()` - Loads the boundary shapefile layer
2. User selects features in map canvas
3. `on_pushButton_pressed()` - Iterates selected features, extracts URLs from `url` attribute
4. `gdal.BuildVRT()` - Merges remote raster URLs into a single VRT file
5. Resulting VRT is added to the QGIS project

## Dependencies

- QGIS 3.0+ (compatible with QGIS 4.x via Qt6 support)
- PyQt5/PyQt6 (via qgis.PyQt compatibility layer)
- GDAL Python bindings (osgeo.gdal)

## Qt5/Qt6 Compatibility

The plugin supports both Qt5 (QGIS 3.x) and Qt6 (QGIS 4.x):
- Uses `QMessageBox.StandardButton.Ok` instead of `QMessageBox.Ok`
- Uses `exec()` instead of `exec_()`
- Imports `QAction` from `QtGui` (Qt6 location, qgis.PyQt provides compatibility)

## Coordinate System

The plugin uses EPSG:32633 (WGS 84 / UTM zone 33N) for the Campania region data.

## Packaging

Create a clean zip for QGIS plugin repository (excludes macOS metadata):
```bash
cd /path/to/plugins
zip -r lidar_campania.zip lidar_campania -x "*.pyc" -x "*__pycache__*" -x "*.git*" -x "*__MACOSX*" -x "*.DS_Store" -x "*.idea*"
```
