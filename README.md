# NASA-GIBS-Downloader
NASA-GIBS-Downloader is a command-line tool which facilitates the downloading of NASA satellite imagery and offers different functionalities in order to prepare the images for training in a machine learning pipeline. The tool currently provides support for downloading the following products: `MODIS_Terra_CorrectedReflectance_TrueColor`, `VIIRS_SNPP_CorrectedReflectance_TrueColor`. You can read more about these products [here](https://wiki.earthdata.nasa.gov/display/GIBS/GIBS+Available+Imagery+Products#expand-CorrectedReflectance17Products).  
\
For example, to download imagery of the Bay Area in California from September to October, the tool can be used as follows:  
`gdl 2020-09-01 2020-10-31 37.003277,-124.328539 40.353784,-120.253964`. Note that if the latitude of either the bottom left or top right coordinates is negative, then it will need to be enclosed by quotation marks (ie `"-23.6319752,-46.6173164"`) or else you might get an error.

## Dependencies 
This package depends on the GDAL translator library. Unfortunately, GDAL is not pip installable. Before installing the GIBSDownloader package and thus the GDAL Python binding, you have to install GDAL on your machine. I have found that one of the easiest ways to do this is create a virtual environment in which you will use the GIBSDownloader, and then install GDAL with conda as follows: ``conda install -c conda-forge gdal=3.2.0``.

## Installation
Once GDAL is installed on your machine, the GIBSDownloader package can be installed using: `pip install git+https://github.com/spaceml-org/NASA-GIBS-Downloader.git@fernando/cli-tool#egg=GIBSDownloader`  
Once installed, the packaged can be referenced as `gdl` on the command-line.  
\
**NOTE:** this package must be installed in the same virtual environment in which you installed GDAL.

## Usage
### Positional Arguments
There are four required positional arguments which are as follows:
`start-date`, `end-date`, `bottom-left-coords`, `top-right-coords`. The first two arguments establish a range of dates to download the images, and the last two arguments form the bottom left and top right coordinates of the desired rectangular region to be downloaded. Note that the bottom left and top right coordinate pairs should be entered as `latitude,longitude`, separated by a comma with no space.

### Optional Parameters
As well as the required positional arguments, the GIBSDownloader also offers some optional parameters for increased customizability.  
* `--tile`: when set to true, each downloaded image will be tiled, and the tiles will be outputted as jpegs.  
* `--tile-width`: specifies the width of each tile (defaults to 512 px).  
* `--tile-height`: specifies the height of each tile (defaults to 512 px).  
* `--tile-overlap`: determines the overlap between consecutive tiles while tiling (defaults to 0.5).  
* `--boundary-handling`: determines what the tiling function should do when it reaches a tile that extends past the boundary of the image. There are three options: 
    - `complete-tiles-shift` guarantees that the edges of the images will be included in the tiles, but it performs a shift such that `tile-overlap` may not be respected (defaults to `complete-tiles-shift`)
    - `include-incomplete-tiles` includes the tiles which extend past the boundary and are thus missing data values for portions of the image
    - `discard-incomplete-tiles` simply removes the images which extend past the boundaries . 
* `--generate-tfrecords`: when set to true, the tiles are used to generate 100 MB TFRecord files which contain the tiles as well as the coordinates of the top left and bottom right corner of each tile (defaults to false).    
* `--remove-originals`: when set to true, the original downloaded images will be deleted and only the tiled images and TFRecords will be saved (defaults to false).  
* `--verbose`: when set to true, prints additional information about downloading process to console (defaults to false).
* `--product`: selects which NASA imagery product to download from. There is currently support for two products:
    - `modis`: downloads `MODIS_Terra_CorrectedReflectance_TrueColor`
    - `viirs`: downloads `VIIRS_SNPP_CorrectedReflectance_TrueColor` (defaults to `viirs`)

### Example 
Say we want to download images of the Bay Area in California from 15 September 2020 to 30 September 2020, while also tiling the downloaded images and writing to TFRecords.  
\
This can be done with the following command:  
`gdl 2020-09-15 2020-09-30 37.003277,-124.328539 40.353784,-120.253964 --tile=true --generate-tfrecords=true`.  
\
If we wanted specify the tile size and overlap, while also removing the original downloaded images, the command would be:  
`gdl 2020-09-15 2020-09-30 37.003277,-124.328539 40.353784,-120.253964 --tile=true --tile-width=256 --tile-height=256 --tile-overlap=0 --remove-originals=true --generate-tfrecords=true`  
\
These will create the following directory structure: 

```
product_lower-lat_left-lon_start-date_end-date/
      |> original_images/
           |> product_date.tif
      |> tiled_images/
           |> width_height_overlap/
                |> date_coordinates.jpeg
      |> tfrecords
           |> width_height_overlap/
                |> product_tf.tfrecord
```

## FAQ
#### How can I find the coordinates of the bottom left the top right corners of the rectangular region that I want to download?
On [Google Maps](https://www.google.com/maps), you can right click at any point on the map, and you will be able to copy that point's latitude and longitude. You can then right click on two points that would form the bottom left and top right corners of a rectangular region and copy those coordinates.

#### What is tiling?
The GeoTiff files for the downloaded regions can potentially be very large images that you might not be able to work with directly (think images of the whole world). Tiling makes smaller "tiles" from the large image, which are essentially smaller images that combine to form the larger one.

#### Can I tile images and write to TFRecords after already having downloaded them?
If you initially download a region for range of dates without electing to tile the images, you can call the command again with the same coordinates for the region and same range of dates but with the tiling flag set to true, and the package will tile the already downloaded images. You can also call the same command multiple times with varying tile sizes and overlaps, and the package will create new folders in `tiled_images/` for each specified combination of tile size and overlap. It will not download the tiff files for the same region and dates twice. Note that if you select `--remove-originals`, you will not be able to perform these additional tilings after the initial command, as the original images will be deleted.

#### I want to download imagery of the entire Earth. What do I need to know?
To download the entire Earth, the coordinates you need to enter are: `"-90,-180" 90,180`. The tiff file for one day of the entire Earth is approximately 38 GB.

## Citation
If you find GIBSDownloader useful in your research, please consider citing
```
@article{lisboa2020democratizing,
  title={Democratizing Earth Science Research with Accessible Data High-Performance Training Pipelines},
  author={Lisboa, Fernando and Verma, Shivam and Koul, Anirudh and Kasam, Meher Anand and Ganju, Siddha},
  journal={Committee on Space Research Cloud Computing Workshop},
  year={2021}
}
```