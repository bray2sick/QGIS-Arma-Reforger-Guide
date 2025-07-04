# QGIS Arma Reforger Guide

## 🛠️ Requirements

- [QGIS](https://qgis.org/download/) (QGIS Desktop 3.44.0)
- Elevation data in GeoTIFF or ASC (ASCII Grid) format

## 📚 Resources

- [QGIS Documentation](https://docs.qgis.org/)
- [Arma Reforger](https://reforger.armaplatform.com/)
- [Bohemia Interactive Community Wiki](https://community.bistudio.com/wiki/Main_Page)
- [Arma Platform Discord](https://discord.com/invite/arma)
- [OpenDEM](https://www.opendem.info/opendemsearcher.html)
- [EPSG.io](https://epsg.io/)

This guide walks you through the full process of preparing elevation (heightmap) and satellite (satmap) data in QGIS for use with Arma Reforger’s Terrain Tool. It’s designed for new and experienced modders alike, using free tools and open data.

Alongside heightmaps, satellite imagery is one of the main data sources used in terrain creation. It serves as reference material for shaping your environment and is often imported as a temporary satmap before generating a final version from in-game terrain materials.

To obtain satellite imagery, we will use QGIS's XYZ Tiles feature, which allows loading online map tiles (like Google Earth or Bing) directly into your project.

### Satmap Data (Satellite Imagery)

Satellite imagery is used as a reference layer and optionally as a temporary texture for your terrain. We use QGIS’s **XYZ Tiles** feature to add online map tiles like Google Satellite.

### 🔌 Setting Up Tile Sources

1. Open the **Browser panel** in QGIS.
2. Right-click on **XYZ Tiles** and select **New Connection**.
3. Give it a name (e.g., `Google Satellite`).
4. Paste one of the URLs below.

| Name                 | Type         | URL |
|----------------------|--------------|-----|
| **Google Satellite** | Satellite    | <https://mt1.google.com/vt/lyrs=s&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D> |
| **Bing Satellite**   | Satellite    | <http://ecn.t3.tiles.virtualearth.net/tiles/a{q}.jpeg?g=1> |
| **ArcGIS Imagery**   | Satellite    | <https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D> |
| **OpenStreetMap**    | Topographic  | <https://tile.openstreetmap.org/{z}/{x}/{y}.png> |
| **Google Maps**      | Topographic  | <https://mt1.google.com/vt/lyrs=m&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D> |

> ✅ You only need to set these up once — they’ll remain in your QGIS Browser panel for all future projects.

Once added, **double-click** the tile service to add it as a layer to your map.

### Heightmap Data (DEM)

Heightmaps are grayscale images where pixel brightness represents terrain elevation. The most common format is **GeoTIFF**.

#### 🔍 Where to Find Elevation Data

The easiest way to find high-quality DEMs is via the [**OpenDEM**](https://opendem.info/) website:

- **Red areas** show high-resolution coverage.
- If unavailable, switch to **middle-resolution** (bottom-left).
- If no data exists, search for national geoportals (e.g., “Australia DEM geoportal”).

If the **OpenDEM Searcher** shows data for your location, click the red-highlighted area. This will bring up more information about the available datasets. In some regions, you may be able to choose between multiple alternatives or resolutions.

Click the **Link DTM** to go to the data provider’s website. Some links may be outdated — if so, go to the provider’s homepage and look for a **DEM** or **DTM** section manually (e.g., “Elevation”, “Terrain”, or “Geospatial downloads”).

> ⚠️ Some areas are not indexed in OpenDEM, but governments may still offer public DEMs — search online for terms like "`[Country Name] elevation data`" or "`[Country Name] geoportal`".

As there are many different providers, exact steps will vary. However, most sites offer an **interactive map** where you can select tiles covering your area of interest. These are typically provided as `.tif` files, but you may also find **ASCII Grid files** (`.asc`), which are fully supported by QGIS.

Once downloaded, **extract the files** (if zipped) and move the `.tif` or `.asc` files into your project folder for use in QGIS.

### Merging Raster Tiles

Depending on your location, elevation data may come as a single large file (common for smaller islands or compact areas) or as multiple tiles covering the region.

After adding your DEM tiles as layers in QGIS, merge them into a single raster for easier processing:

1. In the menu bar, go to **Raster > Miscellaneous > Merge**.
2. Select all your DEM tiles under **Input layers**.
3. Choose an **Output file** location inside your project folder.
4. Click **Run** to merge the tiles.
5. After merging, remove the individual tile layers to declutter your workspace.

Merging simplifies further processing such as clipping, reprojection, and exporting.

### Setting a Projection (CRS)

Before defining your map boundaries, it's important to set an appropriate **map projection** (also called a **Coordinate Reference System**, or CRS).

> **What is a map projection?**
>
> *“All projections of a sphere on a plane necessarily distort the surface in some way and to some extent. Depending on the purpose of the map, some distortions are acceptable and others are not; therefore, different map projections exist in order to preserve some properties of the sphere-like body at the expense of other properties.”*
>
> — [Wikipedia: Map Projection](https://en.wikipedia.org/wiki/Map_projection)

In short: All projections distort the earth’s surface, but different projections distort in different ways. Distortions increase further from the projection center or axis.

#### Why Choose a Suitable Projection?

For creating a realistic terrain based on real-world data, minimizing distortion in your area of interest is key. Choosing a CRS designed for your local region will help keep the data accurate.

#### How to Find a Suitable Projection (CRS)

- **Check your heightmap’s existing CRS**:  In QGIS, open the **Layer Properties** of your DEM layer. Under **Information > Coordinate Reference System (CRS)**, you will see its current projection. This is often already suitable for the region.
- **Use online tools to find local CRS**: Visit [EPSG.io](https://epsg.io/) and search for your country or region. Many countries have multiple zones — pick one that roughly matches your area. Don’t worry about absolute perfection; switching from a global CRS (like EPSG:4326 or EPSG:3857) to a country-specific CRS will greatly reduce distortion.

#### Setting the CRS in QGIS

1. Copy the EPSG code of your chosen CRS (e.g., `EPSG:28354` or `ESRI:102100`).
2. In QGIS, click the **CRS Status** button in the bottom-right corner of the window.
3. In the dialog, paste or search for the EPSG code and select it.
4. Click **OK** to set the project CRS.

Your project will now use this CRS as the **project CRS**. Whenever you run tools that allow CRS selection, choose this same CRS to maintain consistency.

### Creating a Boundary Box Layer

Once your project CRS is set, it's helpful to create a visual reference box over your terrain. This makes it easier to define clipping areas, track working extents, and maintain a clean layout during terrain preparation.

Here’s how to do it using the extent of your merged DEM:

1. **Select your merged DEM layer** in the Layers panel.
2. Go to the top menu and click:  
   **Layer > Create Layer > New Temporary Scratch Layer from Extent**  
   (Or right-click the DEM and choose **Create Layer from Extent**).
3. A new layer will appear, usually called `Extent`.
4. Press **F2** to rename the layer — name it something meaningful like `buffer`.
5. **Right-click** the new `buffer` layer and go to **Properties > Symbology**.
6. In the top dropdown, select **"Colorful"** and choose **"outline red"** from the **Project Styles** list (or set a red stroke manually).
7. Click **OK** to apply the style.

### Exporting RGB Data (Satmap Imagery)

This method is for exporting RGB imagery (like satellite maps) from **XYZ Tiles** or **WMS/WMTS** layers in QGIS.  
> ⚠️ This method is **not suitable** for heightmaps or grayscale elevation data — it is only for **8-bit RGB** imagery.

#### 🎯 Before You Begin

1. Choose the satellite layer you want to export (e.g., Google Satellite or ArcGIS Imagery).
2. Hide all other layers to avoid exporting unnecessary data.
3. If desired, you can export **multiple satellite sources** for comparison or blending later.

#### 🖼️ Exporting the Satmap

1. In the menu bar, go to: **Project > Import/Export > Export Map to Image**
2. In the dialog box, click **Calculate from Layer**, then select your `buffer` layer (bounding box).
3. QGIS will calculate the export **extent** and show an **Output Width / Height**.

#### 🧮 Adjusting Export Resolution

- The image resolution is based on your current zoom level in the QGIS map view.
- To increase or decrease output resolution:
  - Zoom in to increase pixel density.
  - Zoom out to lower resolution.
  - Repeat the export dialog until you're satisfied.
- For best results, aim for around **0.5–2 meters per pixel**, depending on your final terrain size and detail needs.

#### 💾 Final Export

- Set the output format to **PNG**.
- Choose a save location inside your project folder.
- Click **Save** to export the image.

You now have an RGB satellite image ready to use as a **satmap** in Arma Reforger, or for further reference and alignment during terrain painting and detailing.

### Exporting Local Grayscale Raster Data (Heightmap)

When exporting a heightmap, it's critical to preserve **grayscale elevation data quality**. This workflow ensures the result is suitable for terrain import into Arma Reforger Workbench.

#### 🗺️ Step 1: Clip Raster by Extent

1. Go to **Raster > Extraction > Clip Raster by Extent**.
2. Set your **merged DEM** as the **Input layer**.
3. On the **Clipping extent** line, click the small arrow → **Calculate from Layer > buffer** (your red outline box).
4. Choose an output file path (or leave it temporary) and **Run** the tool.
5. A new layer like `Clipped (extent)` will appear.

#### 📐 Step 2: Reproject (if needed)

If the **CRS of your clipped layer** doesn't match your **project CRS** (shown in the bottom-right of QGIS), you must reproject it:

1. Go to **Raster > Projections > Warp (Reproject)**.
2. Set the **Input layer** to your clipped DEM.
3. Set the **Target CRS** to your **project CRS** (e.g., `EPSG:28354`).
4. Choose an output path (or temporary file), then **Run**.

#### 📊 Step 3: Record Elevation Range

To export the heightmap correctly, you'll need the **minimum and maximum elevation values** of your reprojected layer:

1. Right-click the layer → **Properties > Symbology**.
2. Note the **Minimum** and **Maximum** values under the stretch settings — these represent the lowest and highest elevation in meters.

You’ll use these values in the next step to scale the terrain properly.

#### 📤 Step 4: Export Heightmap as PNG

1. Go to **Raster > Conversion > Translate (Convert Format)**.
2. Set the **Input layer** to your reprojected DEM.
3. Set **Output Data Type** to `UInt16` (16-bit grayscale).
4. Under **Additional command-line parameters**, enter: `-scale MIN MAX 0 65535`
5. Replace `MIN` and `MAX` with the elevation values you copied earlier.
6. Set the **Output file** path to a `.png` inside your project folder (e.g., `heightmap.png`).
7. Disable **Open output file after running the algorithm** if desired.
8. Click **Run**.

You now have a properly scaled, 16-bit grayscale PNG heightmap ready to be imported into Arma Reforger’s Workbench using the **Terrain Tool**. Make sure to also remember the **MIN/MAX elevation values** — you'll need them for the **Resample Heights** setting when importing.

## 🧩 Importing Into Arma Reforger

Once you've exported your **16-bit heightmap PNG** from QGIS, you can import it directly into Arma Reforger using the Terrain Tool.

### 🛠️ Steps to Import Heightmap

1. **Open Arma Reforger Workbench**.
2. In your world project, **create or open** the `.ent` world file (your terrain scene).
3. Open the **Terrain Tool**  
   - From the top menu: `Tools > Terrain Tool`  
   - Or use the shortcut: `Ctrl + T`
4. In the Terrain Tool panel, click **Import Heightmap**.
5. Browse to your exported `heightmap.png` file.

### ⚙️ Set Resample Heights

In the **Import Heightmap** dialog:

- ✅ **Enable**: `Resample Heights`
- Set the **Minimum Height** — use the `MIN` elevation value from QGIS.
- Set the **Maximum Height** — use the `MAX` elevation value from QGIS.

```plaintext
Minimum Height: 12.3
Maximum Height: 248.7
```

6. Click **Import**.

Your heightmap will now be loaded into your world with the correct elevation scale.

### 🙏 Credits

This guide is heavily inspired by the work of **Til**, author of the original:

**Advanced QGIS Usage for Terrain Preparation**  
📄 [Read Til’s Original Guide on Google Docs](https://docs.google.com/document/d/1FESiDx2g2nlun22TS31H3Gb0_BhEz9MsfHbOJk9EWog/edit?tab=t.0#heading=h.rxqplz62jktr)

Much of the content in this guide is adapted from Til’s Google Docs, rewritten and reformatted for clarity and structure. Full credit goes to Til for making this knowledge accessible to the Arma terrain community.

> If you found this guide useful, please support the original author and the modding community.

### 🧠 Tips & Troubleshooting

- **QGIS Export Looks Blurry or Low-Res?**
  - Zoom in more before exporting satmap. QGIS exports based on screen resolution.

- **DEM Data Won’t Load or Looks Corrupt?**
  - Ensure files are unzipped and not renamed.
  - Try converting `.asc` to `.tif` using QGIS: *Raster > Conversion > Translate*.

- **Can't Find Local CRS for Your Country?**
  - Use `EPSG:4326` (WGS 84) as a fallback, then resample later in Workbench.

- **Heightmap Doesn't Import Correctly?**
  - Ensure it’s 16-bit grayscale (`UInt16`) PNG with correct scaling.
  - Double-check MIN/MAX elevation values used in the `-scale` parameter.
