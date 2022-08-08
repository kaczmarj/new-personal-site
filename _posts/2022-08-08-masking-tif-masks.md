---
title:  "Making pyramidal TIF masks for segmentation"
excerpt: "Convert polygons in GeoJSON to a TIF whole slide image mask."
---

![Image of nuclei labels overlaid on histology.](https://raw.githubusercontent.com/kaczmarj/geojson2tif/main/sample.png)

This post will explain how I created multi-resolution (pyramidal) TIF images of binary
masks for use with histology. **The goal of this**
is to create a whole slide image of segmentations, which can be accessed in the same
way as the original histology image. The segmentation image is the same size (width, length)
as the original histology image, and they have the same physical units (i.e., microns per pixel).

The segmentations can be of many different things, including nuclei, cell boundaries, tumor regions, stroma, etc.

As part of this, I created a command-line tool to convert
a GeoJSON representation of polygons to a binary mask TIF image. Find it here:
[https://github.com/kaczmarj/geojson2tif/](https://github.com/kaczmarj/geojson2tif/).

The main workhorse is the [ASAP whole slide image viewer](https://computationalpathologygroup.github.io/ASAP/)
and the [`wholeslidedata` Python package](https://github.com/DIAGNijmegen/pathology-whole-slide-data).
These packages implement the conversion of a set of polygons to a whole slide image format.

## GeoJSON

GeoJSON is a powerful format to represent geometries, like polygons. Below I show an
example of GeoJSON with one polygon. This is simply a JSON object that conforms to a schema.
As such, it is highly flexible. You can use common JSON tools (like Python's `json` module in the standard library)
with GeoJSON. Additionally, QuPATH is able to consume GeoJSON (just drag and drop into the viewer).

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [0, 0],
            [1, 0],
            [1, 1],
            [0, 1],
            [0, 0]
          ]
        ]
      }
    }
  ]
}
```

## GeoJSON to TIF

Once we have our data in GeoJSON format, we can use it with other tools. For example,
we can use it directly with the `geojson2tif` command-line tool I wrote. Below, I demonstrate
how to use it with `apptainer` (formerly Singularity).

```
apptainer pull docker://kaczmarj/geojson2tif:latest
apptainer exec --pwd $(pwd) geojson2tif_latest.sif \
    --wsi image.tif --geojson polygons.json --output mask.tif
```

Here is an explanation of the arguments:
- `--wsi image.tif` : the original whole slide image with which the labels are associated.
This will often be a hematoxylin and eosin slide.
- `--geojson polygons.json` : the GeoJSON file with the polygon coordinates. This can
be gzipped.
- `--output mask.tif` : the path of the output TIF image containing the binary mask.


This process can take some time... perhaps 20 to 30 minutes. Perhaps more. Maybe less. But probably more.

The script that converts the GeoJSON to TIF is relatively straightforward. The difficult
work of constructing the pyramidal TIF is done by ASAP and `wholeslidedata`.

Here are the operations:
1. Convert the GeoJSON format to a [format specific to `wholeslidedata`](https://github.com/DIAGNijmegen/pathology-whole-slide-data/blob/e6ba4338ed2528e4fd40552edaee3c845973e7f8/wholeslidedata/annotation/parser.py#L18-L47).
    ```json
    {
    "index": 0,
    "type": "polygon",
    "coordinates": [[0, 0], [1, 0], [1, 1], [0, 1], [0, 0]],
    "label": {
        "name": "nucleus",
        "value": 1
      }
    }
    ```
2. Identify the physical units of the whole slide image. This is microns per pixel.
If this is not correct, the segmentations will not overlap properly with the original slide image.
3. Decide on a tile size for the tiled TIF (1024 is a reasonable choice).
4. Write the mask with `wholeslidedata.accessories.asap.write_mask2.convert_annotations_to_mask`

For a complete example, see [this section of `geojson2tif.py`](https://github.com/kaczmarj/geojson2tif/blob/1be6b99ea1f323e55f2c032905cb2cadd6b6af0d/geojson-to-tif.py#L29-L50).


## Why Apptainer (containers in general?)

ASAP is an incredibly powerful tool, but in my experience, the installation is awkward.
Building from source requires lots of different dependencies (which is expected), and
thankfully the maintainers provide `.deb` packages for installation on Debian-based systems.
On my laptop, I run Debian Testing, but I did not want to install ASAP globally (not to mention its dependencies).
Containers to the rescue. I created [a Dockerfile](https://github.com/kaczmarj/geojson2tif/blob/main/Dockerfile)
that builds a Docker image based on Ubuntu and installs ASAP. I chose to build it as a Docker image instead
of directly as Apptainer/Singularity because of personal preference. I then pushed it to DockerHub,
and Apptainer/Singularity will happilly pull and convert this image. Long story short,
I am able to use ASAP without having to install it globally on my laptop. I can even
use it headless on a high performance computing cluster, where I don't have root access.

## Using the images in a machine learning pipeline

I will cover this in another blog post.
