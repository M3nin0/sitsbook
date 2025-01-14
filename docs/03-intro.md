# Introduction{-}



<a href="https://www.kaggle.com/esensing/introduction-to-sits" target="_blank"><img src="https://kaggle.com/static/images/open-in-kaggle.svg"/></a>


## Who is this book for?{-}

This book is intended for land use change experts and researchers, allowing them to harness the power of big Earth observation data sets. We aim to provide readers with the means to produce high-quality maps of land use and land cover, guiding them through all necessary steps to achieve good results. Given the natural world's complexity and huge variations in human-nature interactions, we consider that only local experts who know their countries and their ecosystems can extract full information from big EO data. 

One group of readers that we are particularly keen to engage with are the national authorities on forest, agriculture, and statistics in developing countries. We aim to foster a collaborative environment where they can use EO data to enhance their national land use and cover estimates, thereby supporting sustainable development policies. To achieve this goal, `sits` has strong backing from the FAO Expert Group on the Use of Earth Observation data (FAO-EOSTAT)[https://www.fao.org/in-action/eostat]. FAO-EOSTAT is at the forefront of using advanced EO data analysis methods for agricultural statistics in developing countries [@DeSimone2022][@DeSimone2022a].

## Why work with satellite image time series?{-}

Satellite images are the most comprehensive source of data about our environment.  Covering a large area of the Earth's surface, images allow researchers to study regional and global changes. Sensors capture data in multiple spectral bands to measure the physical, chemical, and biological properties of the Earth's surface. By observing the same location multiple times, satellites provide data on changes in the environment and survey areas that are difficult to observe from the ground. Given its unique features, images offer essential information for many applications, including deforestation, crop production, food security, urban footprints, water scarcity, and land degradation.

A time series is a set of data points collected at regular intervals over time. Time series data is used to analyze trends, patterns, and changes. Satellite image time series refer to time series obtained from a collection of images captured by a satellite over a period of time, typically months or years. Using time series, experts improve their understanding of ecological patterns and processes. Instead of selecting individual images from specific dates and comparing them, researchers track change continuously [@Woodcock2020]. 

## Time-first, space-later{-}

"Time-first, space-later" is a concept in satellite image classification that takes time series analysis as the first step for analyzing remote sensing data, with spatial information being considered after all time series are classified. The *time-first* part brings a better understanding of changes in landscapes. Detecting and tracking seasonal and long-term trends becomes feasible, as well as identifying anomalous events or patterns in the data, such as wildfires, floods, or droughts. Each pixel in a data cube is treated as a time series, using information available in the temporal instances of the case. Time series classification is pixel-based, producing a set of labeled pixels. This result is then used as input to the *space-later* part of the method. In this phase, a smoothing algorithm improves the results of time-first classification by considering the spatial neighborhood of each pixel. The resulting map thus combines both spatial and temporal information.

## How `sits` works {.unnumbered}

The `sits` package uses satellite image time series for land classification, using  a *time-first, space-later* approach. In the data preparation part, collections of big Earth observation images are organized as data cubes. Each spatial location of a data cube is associated with a time series. Locations with known labels train a machine learning algorithm, which classifies all time series of a data cube, as shown in Figure \@ref(fig:gview).

<div class="figure" style="text-align: center">
<img src="./images/sits_general_view.png" alt="Using time series for land classification (source: authors)." width="70%" height="70%" />
<p class="caption">(\#fig:gview)Using time series for land classification (source: authors).</p>
</div>

The package provides tools for analysis, visualization, and classification of satellite image time series. Users follow a typical workflow for a pixel-based classification:

1.  Select an analysis-ready data image collection from a cloud provider such as AWS, Microsoft Planetary Computer, Digital Earth Africa, or Brazil Data Cube.
2.  Build a regular data cube using the chosen image collection.
3.  Obtain new bands and indices with operations on data cubes.
4.  Extract time series samples from the data cube to be used as training data.
5.  Perform quality control and filtering on the time series samples.
6.  Train a machine learning model using the time series samples.
7.  Classify the data cube using the model to get class probabilities for each pixel.
8.  Post-process the probability cube to remove outliers.
9.  Produce a labeled map from the post-processed probability cube.
10. Evaluate the accuracy of the classification using best practices.

Each workflow step corresponds to a function of the `sits` API, as shown in the Table below and Figure \@ref(fig:api). These functions have convenient default parameters and behaviors. A single function builds machine learning (ML) models. The classification function processes big data cubes with efficient parallel processing. Since the `sits` API is simple to learn, achieving good results do not require in-depth knowledge about machine learning and parallel processing.


<table class="table" style="font-size: 14px; margin-left: auto; margin-right: auto;">
<caption style="font-size: initial !important;">(\#tab:unnamed-chunk-2)The sits API workflow for land classification.</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> API_function </th>
   <th style="text-align:left;"> Inputs </th>
   <th style="text-align:left;"> Output </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_cube() </td>
   <td style="text-align:left;"> ARD image collection </td>
   <td style="text-align:left;"> Irregular data cube </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_regularize() </td>
   <td style="text-align:left;"> Irregular data cube </td>
   <td style="text-align:left;"> Regular data cube </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_apply() </td>
   <td style="text-align:left;"> Regular data cube </td>
   <td style="text-align:left;"> Regular data cube with new bands and indices </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_get_data() </td>
   <td style="text-align:left;"> Data cube and sample locations </td>
   <td style="text-align:left;"> Time series </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_train() </td>
   <td style="text-align:left;"> Time series and ML method </td>
   <td style="text-align:left;"> ML classification model </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_classify() </td>
   <td style="text-align:left;"> ML classification model and regular data cube </td>
   <td style="text-align:left;"> Probability cube </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_smooth() </td>
   <td style="text-align:left;"> Probability cube </td>
   <td style="text-align:left;"> Post-processed probability cube </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_uncertainty() </td>
   <td style="text-align:left;"> Post-processed probability cube </td>
   <td style="text-align:left;"> Uncertainty cube </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_label_classification() </td>
   <td style="text-align:left;"> Post-processed probability cube </td>
   <td style="text-align:left;"> Classified map </td>
  </tr>
  <tr>
   <td style="text-align:left;font-family: monospace;color: RawSienna !important;"> sits_accuracy() </td>
   <td style="text-align:left;"> Classified map and validation samples </td>
   <td style="text-align:left;"> Accuracy assessment </td>
  </tr>
</tbody>
</table>


<div class="figure" style="text-align: center">
<img src="./images/sits_api.png" alt="Main functions of the sits API (source: authors)." width="100%" height="100%" />
<p class="caption">(\#fig:api)Main functions of the sits API (source: authors).</p>
</div>

Additionally, experts can perform object-based image analysis (OBIA) with `sits`. In this case, before classifying the time series, one can use `sits_segments()` to create a set of closed polygons. These polygons are classified using a subset of the time series contained inside each segment. For details, see Chapter [Object-based time series image analysis](https://e-sensing.github.io/sitsbook/object-based-time-series-image-analysis.html).

## Land use and land cover{-}

Since `sits` aims mainly to support land use and land cover classification, this section presents a short discussion on the use of these terms. The UN Food and Agriculture Organization defines land cover as "the observed biophysical cover on the Earth's surface" [@DiGregorio2016]. Land cover can be observed and mapped directly through remote sensing images. In FAO's guidelines and reports, land use is described as "the human activities or purposes for which land is managed or exploited". FAO's land use classifications include classes such as cropland and pasture. Although *land cover* and *land use* denote different approaches for describing the Earth's landscape, in practice there is considerable overlap between these concepts [@Comber2008b]. When classifying remote sensing images, natural areas are classified using land cover types (e.g, forest), while human-modified areas are described with land use classes (e.g., pasture). 

One of the advantages of using image time series for land classification is its capacity of measuring changes in the landscape related to agricultural practices. For example, the time series of a vegetation index in an area of crop production will show a pattern of minima (planting and sowing stages) and maxima (flowering stage). Thus, classification schemas based on image time series data can be richer and more detailed than those associated only with land cover. In what follows, we use the term "land classification" to refer to image classification representing both land cover and land use classes.

## Classes and labels{-}

In this book, we distinguish between the concepts of "class" and "label". A class denotes a group of spatial objects that share similar land cover and land use types, such as urban areas, forests, water bodies, or agricultural fields. Classes are defined based on the specific application or study being conducted, and they help to analyse the vast amount of data obtained from remote sensing imagery. A label is the assignment or identification given to a specific feature or object within an image. Labels are markers that indicate to which class a particular pixel, segment, or object belongs. Labels are essential for supervised classification methods, where a training dataset with known labels is used to train a machine learning algorithm to recognize and classify new, unlabeled data. Thus, a "class" represents the overall category or group of features, while a "label" refers to the specific assignment of a class to a particular feature or object within an image. 

## Creating a data cube {.unnumbered}

There are two kinds of data cubes in `sits`: (a) irregular data cubes generated by selecting image collections on cloud providers such as AWS and Planetary Computer; (b) regular data cubes with images fully covering a chosen area, where each image has the same spectral bands and spatial resolution, and images follow a set of adjacent and regular time intervals. Machine learning applications need regular data cubes. Please refer to Chapter [Earth observation data cubes](https://e-sensing.github.io/sitsbook/earth-observation-data-cubes.html) for further details.

The first steps in using `sits` are: (a) select an analysis-ready data image collection available in a cloud provider or stored locally using `sits_cube()`; (b) if the collection is not regular, use `sits_regularize()` to build a regular data cube.

This section shows how to build a data cube from local images already organized as a regular data cube. The data cube is composed of MODIS MOD13Q1 images for the region close to the city of Sinop in Mato Grosso, Brazil. This region is one of the world's largest producers of soybeans. All images have indexes NDVI and EVI covering a one-year period from 2013-09-14 to 2014-08-29 (we use "year-month-day" for dates). There are 23 time instances, each covering a 16-day period. This data is available in the package `sitsdata`.

To build a data cube from local files, users must provide information about the original source from which the data was obtained. In this case, `sits_cube()` needs the parameters:

(a) `source`, the cloud provider from where the data has been obtained (in this case, the Brazil Data Cube "BDC");
(b) `collection`, the collection of the cloud provider from where the images have been extracted. In this case, data comes from the MOD13Q1 collection 6; 
(c) `data_dir`, the local directory where the image files are stored; 
(d) `parse_info`, a vector of strings stating how file names store information on "tile", "band", and "date". In this case, local images are stored in files whose names are similar to `TERRA_MODIS_012010_EVI_2014-07-28.tif`. This file represents an image obtained by the MODIS sensor onboard the TERRA satellite, covering part of tile 012010 in the EVI band for date 2014-07-28.


``` r
# load package "tibble"
library(tibble)
# load packages "sits" and "sitsdata"
library(sits)
library(sitsdata)
# Create a data cube using local files
sinop_cube <- sits_cube(
  source = "BDC",
  collection = "MOD13Q1-6.1",
  bands = c("NDVI", "EVI"),
  data_dir = system.file("extdata/sinop", package = "sitsdata"),
  parse_info = c("satellite", "sensor", "tile", "band", "date")
)
# Plot the NDVI for the first date (2013-09-14)
plot(sinop_cube,
  band = "NDVI",
  dates = "2013-09-14",
  palette = "RdYlGn"
)
```

<div class="figure" style="text-align: center">
<img src="03-intro_files/figure-html/introndvi-1.png" alt="False color MODIS image for NDVI band in 2013-09-14 from sinop data cube (source: Authors)." width="100%" />
<p class="caption">(\#fig:introndvi)False color MODIS image for NDVI band in 2013-09-14 from sinop data cube (source: Authors).</p>
</div>

The aim of the `parse_info` parameter is to extract `tile`, `band`, and `date` information from the file name. Given the large variation in image file names generated by different produces, it includes designators such as `X1` and `X2`; these are place holders for parts of the file name that is not relevant to `sits_cube()`. 

The R object returned by `sits_cube()` contains the metadata describing the contents of the data cube. It includes data source and collection, satellite, sensor, tile in the collection, bounding box, projection, and list of files. Each file refers to one band of an image at one of the temporal instances of the cube.


``` r
# Show the description of the data cube
sinop_cube
```

```
#> # A tibble: 1 × 11
#>   source collection satellite sensor tile     xmin    xmax    ymin    ymax crs  
#>   <chr>  <chr>      <chr>     <chr>  <chr>   <dbl>   <dbl>   <dbl>   <dbl> <chr>
#> 1 BDC    MOD13Q1-6… TERRA     MODIS  0120… -6.18e6 -5.96e6 -1.35e6 -1.23e6 "PRO…
#> # ℹ 1 more variable: file_info <list>
```

The list of image files which make up the data cube is stored as a data frame in the column `file_info`. For each file, `sits` stores information about spectral band, reference date, size, spatial resolution, coordinate reference system, bounding box, path to file location and cloud cover information (when available). 


``` r
# Show information on the images files which are part of a data cube
sinop_cube$file_info[[1]]
```

```
#> # A tibble: 46 × 13
#>    fid   band  date       nrows ncols  xres  yres      xmin      ymin      xmax
#>    <chr> <chr> <date>     <dbl> <dbl> <dbl> <dbl>     <dbl>     <dbl>     <dbl>
#>  1 1     EVI   2013-09-14   551   944  232.  232. -6181982. -1353336. -5963298.
#>  2 1     NDVI  2013-09-14   551   944  232.  232. -6181982. -1353336. -5963298.
#>  3 2     EVI   2013-09-30   551   944  232.  232. -6181982. -1353336. -5963298.
#>  4 2     NDVI  2013-09-30   551   944  232.  232. -6181982. -1353336. -5963298.
#>  5 3     EVI   2013-10-16   551   944  232.  232. -6181982. -1353336. -5963298.
#>  6 3     NDVI  2013-10-16   551   944  232.  232. -6181982. -1353336. -5963298.
#>  7 4     EVI   2013-11-01   551   944  232.  232. -6181982. -1353336. -5963298.
#>  8 4     NDVI  2013-11-01   551   944  232.  232. -6181982. -1353336. -5963298.
#>  9 5     EVI   2013-11-17   551   944  232.  232. -6181982. -1353336. -5963298.
#> 10 5     NDVI  2013-11-17   551   944  232.  232. -6181982. -1353336. -5963298.
#> # ℹ 36 more rows
#> # ℹ 3 more variables: ymax <dbl>, crs <chr>, path <chr>
```

A key attribute of a data cube is its timeline, as shown below. The command `sits_timeline()` lists the temporal references associated to `sits` objects, including samples, data cubes and models. 


``` r
# Show the R object that describes the data cube
sits_timeline(sinop_cube)
```

```
#>  [1] "2013-09-14" "2013-09-30" "2013-10-16" "2013-11-01" "2013-11-17"
#>  [6] "2013-12-03" "2013-12-19" "2014-01-01" "2014-01-17" "2014-02-02"
#> [11] "2014-02-18" "2014-03-06" "2014-03-22" "2014-04-07" "2014-04-23"
#> [16] "2014-05-09" "2014-05-25" "2014-06-10" "2014-06-26" "2014-07-12"
#> [21] "2014-07-28" "2014-08-13" "2014-08-29"
```
The timeline of the `sinop_cube` data cube has 23 intervals with a temporal difference of 16 days. The chosen dates capture the agricultural calendar in Mato Grosso, Brazil. The agricultural year starts in September-October with the sowing of the summer crop (usually soybeans) which is harvested in February-March. Then the winter crop (mostly Corn, Cotton or Millet) is planted in March and harvested in June-July. For LULC classification, the training samples and the date cube should share a timeline with the same number of intervals and similar start and end dates.

## The time series tibble {-}

To handle time series information, `sits` uses a `tibble`. Tibbles are extensions of the `data.frame` tabular data structures provided by the `tidyverse` set of packages. The example below shows a tibble with 1,837 time series obtained from MODIS MOD13Q1 images. Each series has four attributes: two bands (NIR and MIR) and two indexes (NDVI and EVI). This dataset is available in package `sitsdata`.

The time series tibble contains data and metadata. The first six columns contain the metadata: spatial and temporal information, the label assigned to the sample, and the data cube from where the data has been extracted. The `time_series` column contains the time series data for each spatiotemporal location. This data is also organized as a tibble, with a column with the dates and the other columns with the values for each spectral band. 


``` r
# Load the MODIS samples for Mato Grosso from the "sitsdata" package
library(tibble)
library(sitsdata)
data("samples_matogrosso_mod13q1", package = "sitsdata")
samples_matogrosso_mod13q1
```

```
#> # A tibble: 1,837 × 7
#>    longitude latitude start_date end_date   label   cube     time_series      
#>        <dbl>    <dbl> <date>     <date>     <chr>   <chr>    <list>           
#>  1     -57.8    -9.76 2006-09-14 2007-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  2     -59.4    -9.31 2014-09-14 2015-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  3     -59.4    -9.31 2013-09-14 2014-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  4     -57.8    -9.76 2006-09-14 2007-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  5     -55.2   -10.8  2013-09-14 2014-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  6     -51.9   -13.4  2014-09-14 2015-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  7     -56.0   -10.1  2005-09-14 2006-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  8     -54.6   -10.4  2013-09-14 2014-08-29 Pasture bdc_cube <tibble [23 × 5]>
#>  9     -52.5   -11.0  2013-09-14 2014-08-29 Pasture bdc_cube <tibble [23 × 5]>
#> 10     -52.1   -14.0  2013-09-14 2014-08-29 Pasture bdc_cube <tibble [23 × 5]>
#> # ℹ 1,827 more rows
```

The timeline for all time series associated with the samples follows the same agricultural calendar, starting in September 14th and ending in August 28th. All samples contain 23 values, corresponding to the same temporal interval as those of the `sinop` data cube. Notice that that although the years for the samples are different, the samples for a given year follow the same agricultural calendar. 

The time series can be displayed by showing the `time_series` column.


``` r
# Load the time series for MODIS samples for Mato Grosso
samples_matogrosso_mod13q1[1, ]$time_series[[1]]
```

```
#> # A tibble: 23 × 5
#>    Index       NDVI   EVI   NIR    MIR
#>    <date>     <dbl> <dbl> <dbl>  <dbl>
#>  1 2006-09-14 0.500 0.263 0.230 0.139 
#>  2 2006-09-30 0.485 0.330 0.359 0.161 
#>  3 2006-10-16 0.716 0.397 0.264 0.0757
#>  4 2006-11-01 0.654 0.415 0.332 0.124 
#>  5 2006-11-17 0.591 0.433 0.400 0.172 
#>  6 2006-12-03 0.662 0.439 0.348 0.125 
#>  7 2006-12-19 0.734 0.444 0.295 0.0784
#>  8 2007-01-01 0.739 0.502 0.348 0.0887
#>  9 2007-01-17 0.768 0.526 0.351 0.0761
#> 10 2007-02-02 0.797 0.550 0.355 0.0634
#> # ℹ 13 more rows
```


The distribution of samples per class can be obtained using the `summary()` command. The classification schema uses nine labels, four associated to crops (`Soy_Corn`, `Soy_Cotton`, `Soy_Fallow`, `Soy_Millet`), two with natural vegetation (`Cerrado`, `Forest`) and one to `Pasture`.


``` r
# Load the MODIS samples for Mato Grosso from the "sitsdata" package
summary(samples_matogrosso_mod13q1)
```

```
#> # A tibble: 7 × 3
#>   label      count   prop
#>   <chr>      <int>  <dbl>
#> 1 Cerrado      379 0.206 
#> 2 Forest       131 0.0713
#> 3 Pasture      344 0.187 
#> 4 Soy_Corn     364 0.198 
#> 5 Soy_Cotton   352 0.192 
#> 6 Soy_Fallow    87 0.0474
#> 7 Soy_Millet   180 0.0980
```


It is helpful to plot the dispersion of the time series. In what follows, for brevity, we will filter only one label (`Forest`) and select one index (NDVI). Note that for filtering the label we use a function from `dplyr` package, while for selecting the index we use `sits_select()`. We use two different functions for selection because of they way metadata is stored in a samples files. The labels for the samples are listed in column `label` in the samples tibble, as shown above. In this case, one can use functions from the `dplyr` package to extract subsets. In particular, the function `dplyr::filter` retaining all rows that satisfy a given condition. In the above example, the result of `dplyr::filter` is the set of samples associated to the "Forest" label. The second selection involves obtaining only the values for the NDVI band. This operation requires access to the `time_series` column, which is stored as a list. In this case, selection with `dplyr::filter` will not work. To handle such cases, `sits` provides `sits_select()` to select subsets inside the `time_series` list. 


``` r
# select all samples with label "Forest"
samples_forest <- dplyr::filter(
  samples_matogrosso_mod13q1,
  label == "Forest"
)
# select the NDVI band for all samples with label "Forest"
samples_forest_ndvi <- sits_select(
  samples_forest,
  band = "NDVI"
)
plot(samples_forest_ndvi)
```

<div class="figure" style="text-align: center">
<img src="03-intro_files/figure-html/timeseriesforest-1.png" alt="Joint plot of all samples in band NDVI for label Forest (source: authors)." width="80%" />
<p class="caption">(\#fig:timeseriesforest)Joint plot of all samples in band NDVI for label Forest (source: authors).</p>
</div>
  
The above figure shows all the time series associated with label `Forest` and band NDVI (in light blue), highlighting the median (shown in dark red) and the first and third quartiles (shown in brown). The spikes are noise caused by the presence of clouds.



## Training a machine learning model {.unnumbered}

The next step is to train a machine learning (ML) model using `sits_train()`. It takes two inputs, `samples` (a time series tibble) and `ml_method` (a function that implements a machine learning algorithm). The result is a model that is used for classification. Each ML algorithm requires specific parameters that are user-controllable. For novice users, `sits` provides default parameters that produce good results. Please see Chapter [Machine learning for data cubes](https://e-sensing.github.io/sitsbook/machine-learning-for-data-cubes.html) for more details.

Since the time series data has four attributes (EVI, NDVI, NIR, and MIR) and the data cube images have only two, we select the NDVI and EVI values and use the resulting data for training. To build the classification model, we use a random forest model called by `sits_rfor()`. Results from the random forest model can vary between different runs, due to the stochastic nature of the algorithm, For this reason, in the code fragment below, we set the seed of R's pseudo-random number generation explicitly to ensure the same results are produced for documentation purposes.


``` r
set.seed(03022024)
# Select the bands NDVI and EVI
samples_2bands <- sits_select(
  data = samples_matogrosso_mod13q1,
  bands = c("NDVI", "EVI")
)
# Train a random forest model
rf_model <- sits_train(
  samples = samples_2bands,
  ml_method = sits_rfor()
)
# Plot the most important variables of the model
plot(rf_model)
```

<div class="figure" style="text-align: center">
<img src="03-intro_files/figure-html/rfintro-1.png" alt="Most relevant variables of trained random forest model (source: authors)." width="80%" />
<p class="caption">(\#fig:rfintro)Most relevant variables of trained random forest model (source: authors).</p>
</div>


## Data cube classification {.unnumbered}

After training the machine learning model, the next step is to classify the data cube using `sits_classify()`. This function produces a set of raster probability maps, one for each class. For each of these maps, the value of a pixel is proportional to the probability that it belongs to the class. This function has two mandatory parameters: `data`, the data cube or time series tibble to be classified; and `ml_model`, the trained ML model. Optional parameters include: (a) `multicores`, number of cores to be used; (b) `memsize`, RAM used in the classification; (c) `output_dir`, the directory where the classified raster files will be written. Details of the classification process are available in "Image classification in data cubes".


``` r
# Classify the raster image
sinop_probs <- sits_classify(
  data = sinop_cube,
  ml_model = rf_model,
  multicores = 2,
  memsize = 8,
  output_dir = "./tempdir/chp3"
)
# Plot the probability cube for class Forest
plot(sinop_probs, labels = "Forest", palette = "BuGn")
```

<div class="figure" style="text-align: center">
<img src="03-intro_files/figure-html/pbforintro-1.png" alt="Probability map for class Forest (source: authors)." width="90%" />
<p class="caption">(\#fig:pbforintro)Probability map for class Forest (source: authors).</p>
</div>

After completing the classification, we plot the probability maps for class `Forest`. Probability maps are helpful to visualize the degree of confidence the classifier assigns to the labels for each pixel. They can be used to produce uncertainty information and support active learning, as described in Chapter [Image classification in data cubes](https://e-sensing.github.io/sitsbook/image-classification-in-data-cubes.html).

## Spatial smoothing {.unnumbered}

When working with big Earth observation data, there is much variability in each class. As a result, some pixels will be misclassified. These errors are more likely to occur in transition areas between classes. To address these problems, `sits_smooth()` takes a probability cube as input and  uses the class probabilities of each pixel's neighborhood to reduce labeling uncertainty. Plotting the smoothed probability map for class Forest shows that most outliers have been removed.


``` r
# Perform spatial smoothing
sinop_bayes <- sits_smooth(
  cube = sinop_probs,
  multicores = 2,
  memsize = 8,
  output_dir = "./tempdir/chp3"
)
plot(sinop_bayes, labels = "Forest", palette = "BuGn")
```

<div class="figure" style="text-align: center">
<img src="03-intro_files/figure-html/byforintro-1.png" alt="Smoothed probability map for class Forest (source: authors)." width="90%" />
<p class="caption">(\#fig:byforintro)Smoothed probability map for class Forest (source: authors).</p>
</div>

## Labeling a probability data cube {.unnumbered}

After removing outliers using local smoothing, the final classification map can be obtained using `sits_label_classification()`. This function assigns each pixel to the class with the highest probability.


``` r
# Label the probability file
sinop_map <- sits_label_classification(
  cube = sinop_bayes,
  output_dir = "./tempdir/chp3"
)
plot(sinop_map)
```

<div class="figure" style="text-align: center">
<img src="03-intro_files/figure-html/mapintro-1.png" alt="Classification map for Sinop (source: authors)." width="100%" />
<p class="caption">(\#fig:mapintro)Classification map for Sinop (source: authors).</p>
</div>

The resulting classification files can be read by QGIS. Links to the associated files are available in the `sinop_map` object in the nested table `file_info`.


``` r
# Show the location of the classification file
sinop_map$file_info[[1]]
```

```
#> # A tibble: 1 × 12
#>   band  start_date end_date   ncols nrows  xres  yres      xmin     xmax    ymin
#>   <chr> <date>     <date>     <dbl> <dbl> <dbl> <dbl>     <dbl>    <dbl>   <dbl>
#> 1 class 2013-09-14 2014-08-29   944   551  232.  232. -6181982.  -5.96e6 -1.35e6
#> # ℹ 2 more variables: ymax <dbl>, path <chr>
```

To simplify the process of importing your data to QGIS, the color palette used to display classified maps in `sits` can be exported as a QGIS style using `sits_colors_qgis`. The function takes two parameters: (a) `cube`, a classified data cube; and (b) `file`, the file where the QGIS style in XML will be written to. In this case study, it suffices to do the following command.


``` r
# Show the location of the classification file
sits_colors_qgis(sinop_map, file = "./tempdir/chp3/qgis_style.xml")
```

## Plotting{-}

The `plot()` function produces a graphical display of data cubes, time series, models, and SOM maps. For each type of data, there is a dedicated version of the `plot()` function. See `?plot.sits` for details. Plotting of time series, models and SOM outputs uses the `ggplot2` package; maps are plotted using the `tmap` package. When plotting images and classified maps, users can control the output with three main parameters:

- `pallete`: color scheme to be used for false color maps, which should be one of the `RColorBrewer` palettes. These palettes have been designed to be effective for map display by Prof Cynthia Brewer as described at the [Brewer website](http://colorbrewer2.org). To see the available palettes, please run `RColorBrewer::display.brewer.all(type = "seq")` and `RColorBrewer::display.brewer.all(type = "div")`. By default, optical images use the `RdYlGn` scheme, SAR images use `Greys`, and DEM cubes use `Spectral`. 
- `rev`: whether the color palette should be reversed; `TRUE` for DEM cubes, and `FALSE` otherwise.
- `scale`: global scale parameter used by `tmap`. All font sizes, symbol sizes, border widths, and line widths are controlled by this value. Default is 0.75; users should vary this parameter and see the results.
- `first_quantile`: 1st quantile for stretching images (default = 0.05)
- `last_quantile`: last quantile for stretching images (default = 0.95)
- `max_cog_size`: for cloud-oriented geotiff files (COG), sets the maximum number of lines or columns of the COG overview to be used for plotting. 

COG overviews are reduced-resolution versions of the main image, stored within the same file. Overviews allow for quick rendering at lower zoom levels, improving performance when dealing with large images. Usually, a single GeoTIFF will have many overviews, to match different zoom levels. The parameter `max_cog_size` controls the size of the overview which will be used for visualisation.

The following optional parameters are available to allow for detailed control over the plot output:


3. `graticules_labels_size`: size of coordinates labels (default = 0.8)
4. `legend_title_size`: relative size of legend title (default = 1.0)
5. `legend_text_size`: relative size of legend text (default = 1.0)
6. `legend_bg_color`: color of legend background (default = "white")
7. `legend_bg_alpha`: legend opacity (default = 0.5)


## Visualization of data cubes in interactive maps {.unnumbered}

 Data cubes and samples can also be shown as interactive maps using `sits_view()`. This function creates tiled overlays of different kinds of data cubes, allowing comparison between the original, intermediate and final results. It also includes background maps. The following example creates an interactive map combining the original data cube with the classified map.


``` r
sits_view(sinop, band = "NDVI", class_cube = sinop_map)
```

<img src="./images/view_sinop.png" width="90%" style="display: block; margin: auto;" />
