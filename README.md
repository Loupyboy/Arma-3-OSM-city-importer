# Arma 3 OSM City Importer

Generate an Arma 3 Eden SQF file from real-world OpenStreetMap (OSM) building data.

The importer is a small browser-based tool: select an area on a map, download its OSM data, convert building footprints into approximate Arma 3 buildings, and download a `generated_city.sqf` file that can be executed in Eden's Debug Console.

> **Important:** This does **not** create a new Arma 3 terrain and it does **not** import real 3D building models. It creates Arma 3 objects using existing class names, positioned and rotated to roughly match OSM building footprints.

## What it does

The workflow is:

```
Search for a place
      ↓
Draw an area on the map
      ↓
Download OSM building/object data
      ↓
Analyse building footprints
      ↓
Choose approximate Arma 3 building classes
      ↓
Generate SQF
      ↓
Run SQF in Eden
      ↓
Buildings appear as Eden objects
```

The generated objects can then be edited normally in Eden: move them, delete them, add roads/props/AI, or save the mission.

## Requirements

### On your computer

- A modern web browser (Chrome, Edge, Firefox, etc.)
- Arma 3 with the Eden Editor
- An internet connection while generating the city
- Access to OpenStreetMap/Overpass services

### No web server is required

You can open `arma_3_city_importer.html` directly in your browser.

The tool loads Leaflet from a CDN and uses public geocoding/OSM services. If your browser blocks requests when opening a local `file://` page, serve the HTML from a simple local HTTP server instead.

## Quick start

1. Download or clone this repository.
2. Open `arma_3_city_importer.html` in your browser.
3. Search for a city or place.
4. Select the search result.
5. Click **Draw Area**.
6. Drag a rectangle around the neighbourhood you want.
7. Adjust **Spacing scale** and **Min gap** if necessary.
8. Enable/disable the optional extra objects.
9. Click **Generate SQF**.
10. Wait for the OSM data to be downloaded and processed.
11. Your browser downloads **`generated_city.sqf`**.
12. Open Arma 3 and start the Eden Editor.
13. Select the terrain where you want to place the city.
14. Put the Eden camera at the **centre of the area you selected**.
15. Point the camera in the direction you want the imported city to face.
16. Open the Eden **Debug Console**.
17. Paste the contents of `generated_city.sqf`.
18. Execute it.
19. The generated objects are created and selected in Eden.
20. Save your mission.

## Selecting an area

The tool has a hard maximum selection area of **10 km²**.

For your first test, use something much smaller — for example:

- a few city blocks
- a neighbourhood
- roughly 0.25–1 km²

Large areas can produce very large SQF files and hundreds or thousands of Eden objects.

The importer works from OSM data, so the quality of the result depends heavily on how well the location is mapped in OpenStreetMap.

## Search

The search box uses OpenStreetMap's Nominatim geocoder.

Enter something like:

```
Cambridge, UK
```

or:

```
Manchester, UK
```

Select the appropriate result, then draw the exact area you want to import.

If Nominatim fails, the tool attempts to use Photon as a fallback for the search.

## Spacing scale

**Default: 1.0**

This multiplies the distances between generated objects.

- `1.0` = approximately real-world spacing
- `0.5` = buildings are packed to half the original spacing
- `2.0` = buildings are spread to twice the original spacing

The building models themselves are **not resized**.

Start with `1.0`.

## Min gap

**Default: 12 metres**

This prevents buildings from being placed too close together.

The importer sorts buildings by footprint area and keeps larger buildings first. Smaller buildings whose centres are closer than the configured gap to an already-kept building are skipped.

Examples:

- `0` = keep everything; no gap filtering
- `5` = allow denser placement
- `12` = default
- larger values = increasingly sparse placement

If your imported city looks too empty, try reducing this value.

## What gets imported

### Buildings

The importer queries OSM for:

```
way["building"]
```

It uses closed building footprints and ignores very small footprints (less than 15 m²).

The footprint is analysed to determine:

- approximate centre
- approximate area
- approximate orientation

The tool then chooses an Arma 3 building class based primarily on footprint area.

The generated buildings come from a list of existing Arma 3 classes such as:

```
Land_House_1W01_F
Land_House_1W02_F
Land_House_1W03_F
...
Land_House_2W01_F
Land_House_2W02_F
...
```

This means the result is an **approximation**, not a 1:1 reconstruction of the real architecture.

### Optional extra objects

The UI can also import selected OSM nodes for:

| OSM type | Default Arma class | Default |
|---|---|---|
| Street lamps | `Land_LampStreet_F` | On |
| Benches | `Land_Bench_F` | On |
| Waste baskets | `Land_GarbageBin_01_F` | On |
| Trees | User supplied | Off |
| Fire hydrants | User supplied | Off |

For objects without a default class, paste an Arma 3 class name into the corresponding field.

The tool removes characters other than letters, numbers and underscores from class names.

## Custom object classes

The extra-object class fields are editable.

For example, if you have an Arma 3 mod containing a preferred street lamp, you can enter that object's class name instead of the default:

```
Land_LampStreet_F
```

The generated SQF checks whether each class exists in `CfgVehicles`.

If a class is missing, the script does not create that object and reports the missing class when it finishes.

## How the generated SQF works

The generated file contains compact data describing each object:

```
[classIndex, eastMetres, northMetres, headingDegrees]
```

When you execute the SQF in Eden it:

1. Reads the centre of the Eden camera.
2. Reads the camera direction.
3. Uses that position as the anchor for the imported OSM area.
4. Converts each generated local coordinate into an Eden world position.
5. Creates each object with `create3DENEntity`.
6. Applies the calculated rotation.
7. Selects the newly created objects.

The generated objects are therefore normal Eden objects rather than runtime-only objects.

## Positioning the import in Eden

This is important.

The SQF does **not** contain a fixed Arma map position.

Instead, the centre of your OSM selection becomes the centre of the generated layout, and the Eden camera determines where that centre is placed.

Before executing the SQF:

1. Move the Eden camera to the location where the centre of the imported area should be.
2. Point the camera in the direction you want the city to face.
3. Run the generated SQF.

The importer uses the camera's position and heading as the anchor/orientation.

## Terrain and roads

This project does **not** generate an Arma terrain.

It also does **not** currently import:

- roads
- road markings
- pavements
- terrain height/elevation
- interiors
- real building meshes
- building textures
- underground structures
- traffic systems

Think of it as an **OSM building/object layout generator for Eden**, not a complete terrain-making pipeline.

You will normally want to add or build the road network and other terrain features separately.

## Data sources

The tool uses:

- **OpenStreetMap / Nominatim** for place search
- **Overpass API** for the main OSM data download
- **OpenStreetMap API** as a fallback if Overpass requests fail
- **Leaflet** for the map UI
- **Esri map tiles** and OpenStreetMap tiles for map display

The importer tries several Overpass servers in parallel and uses the first successful response. If all Overpass requests fail, it falls back to downloading the selected area in smaller OSM API cells.

## Troubleshooting

### "Area is too large"

The selection exceeds the 10 km² limit.

Click **Clear** and draw a smaller area.

### Search does not work

Try a more specific query:

```
Town name, country
```

For example:

```
Oxford, UK
```

The search uses public geocoding services, so temporary service/rate-limit failures are possible.

### Generation takes a long time

OSM/Overpass is external infrastructure.

The importer tries multiple Overpass servers and waits for a successful response. Large selections also contain much more OSM data.

Try a smaller area first.

### Overpass fails

The importer automatically attempts the OpenStreetMap API as a fallback.

If that also fails, wait a while and try again with a smaller selection.

### Very few buildings appear

Possible causes:

- The OSM area has incomplete building data.
- **Min gap** is too high.
- The selected area is very small.
- Many OSM building footprints are invalid/open.
- The OSM data simply does not contain the features you expected.

Try lowering **Min gap** or checking the area in OpenStreetMap.

### Buildings overlap

Lowering **Min gap** increases density, but also makes overlaps more likely because the generated Arma models do not exactly match the real OSM footprints.

Increasing **Min gap** can reduce this problem.

### The city looks wrong

Remember that the importer does not recreate real buildings. It chooses from a relatively small set of Arma 3 building classes based mainly on footprint size.

It is intended to provide a fast starting layout that can then be edited in Eden.

### Objects are facing the wrong way

The importer uses the orientation calculated from the OSM footprint and then rotates the entire generated layout according to the Eden camera heading.

Try changing the camera direction before executing the SQF.

### A custom extra object does not appear

Check that:

1. The class name is correct.
2. The required mod is loaded in Arma 3.
3. The class exists under `CfgVehicles`.

The generated SQF reports missing classes instead of creating invalid objects.

## Recommended first test

For a first run, use:

```
Area:        ~0.25–1 km²
Spacing:     1.0
Min gap:     10–12 m

Street lamps: On
Benches:      On
Bins:         On
Trees:        Off
Hydrants:     Off
```

Generate the SQF, run it in a blank Eden mission, and confirm that the workflow works before attempting a larger area.

## Limitations

This project is intentionally simple. It is best suited to quickly getting a rough city layout into Eden.

It does not currently attempt to:

- create a terrain
- import elevation data
- create roads
- match real building models
- match building heights
- import building interiors
- reproduce textures
- reproduce exact real-world architecture
- guarantee that every OSM building becomes an Arma object

OSM coverage and tagging also vary significantly between locations.

## Credits / attribution

OpenStreetMap data is provided by OpenStreetMap contributors.

When using OSM data, follow the current OpenStreetMap attribution and usage requirements:

https://www.openstreetmap.org/copyright

The map interface uses Leaflet:

https://leafletjs.com/

Map tiles/search/data services are provided by their respective services. Please respect their usage policies and rate limits.

## License

See [LICENSE](LICENSE).

## Repository

[Arma 3 OSM City Importer](https://github.com/Poppadomus/Arma-3-OSM-city-importer)
