# Validation and accuracy measurements{-}




## Case study{-}

To show how to do validation and accuracy assessment in`sits`, we show an example of land classification in the Cerrado biome, the second largest biome in Brazil with 1.9 million km$^2$. The Brazilian Cerrado is a tropical savanna ecoregion with a rich ecosystem ranging from grasslands to woodlands. It is home to more than 7000 species of plants with high levels of endemism [@Klink2005]. It includes three major types of natural vegetation: Open Cerrado, typically composed of grasses and small shrubs with a sporadic presence of small tree vegetation; Cerrado, a typical savanna formation with the presence of low, irregularly branched, thin-trunked trees; and Cerradão, areas of medium-sized trees (up to 10--12 m)  [@Del-Claro2019]. Its natural areas are being converted to agriculture at a fast pace, as it is one of the world's fast-moving agricultural frontiers [@Walter2006]. The main agricultural land uses include cattle ranching, crop farms, and planted forests. The classification follows the work by Simoes et al. [@Simoes2021].

The data comprises 67 Landsat-8 tiles from the Brazil Data Cube (BDC), with 23 time steps covering 2017-08-29 to 2018-08-29. Since the data is available in the Brazil Data Cube, users should first obtain access to the BDC by obtaining an access key. After obtaining the access key, they should include their credentials using environment variables, as shown below. Obtaining a BDC access key is free. To obtain the key, users need to register at the [BDC site](https://brazildatacube.dpi.inpe.br/portal/explore).

After obtaining the BDC access key, we can now create a data cube for the Cerrado biome. 


``` r
# Files are available in the Brazil Data Cube
#
# Obtain the region of interest covering the Cerrado biome
roi_cerrado_shp <- system.file(
  "extdata/shapefiles/cerrado_border/cerrado_border.shp",
  package = "sitsdata"
)
# Read the shapefile as an object of the "sf" package
roi_cerrado <- sf::st_read(roi_cerrado_shp, quiet = TRUE)
# Create a data cube for the entire Cerrado biome
cerrado_cube <- sits_cube(
  source = "BDC",
  collection = "LANDSAT-OLI-16D",
  roi = roi_cerrado,
  start_date = "2017-08-29",
  end_date = "2018-08-29",
  multicores = 3
)
```

To classify the Cerrado, we use a training dataset produced by Simoes et al. [@Simoes2021]. The authors carried out a systematic sampling using a $5 \times 5$ km grid throughout the Cerrado biome, collecting 85,026 samples. The training data labels were extracted from three sources: the 2018 pastureland map from Parente et al. [@Parente2019], MapBiomas Collection 5 for 2018 [@SouzaJr2020], and Brazil's National Mapping Agency IBGE land maps for 2016--2018. Out of the 85,026 samples, the authors selected those without disagreement between the labels assigned by the three sources. The final training set consists of 48,850 points from which the authors extracted the time series using the Landsat-8 data cube available in the BDC. The classes for this training set are: `Annual Crop`, `Cerradao`,  `Cerrado`, `Open Cerrado`, `Nat_NonVeg (Dunes)`, `Pasture`, `Perennial_Crop`, `Silviculture (Planted Forests)`, `Sugarcane`, and `Water`.

The dataset is available in the package `sitsdata` as `samples_cerrado_lc8`. 


``` r
library(sitsdata)
data("samples_cerrado_lc8")
# Show the class distribution in the new training set
summary(samples_cerrado_lc8)
```

```
#> # A tibble: 10 × 3
#>    label          count     prop
#>    <chr>          <int>    <dbl>
#>  1 Annual_Crop     6887 0.141   
#>  2 Cerradao        4211 0.0862  
#>  3 Cerrado        16251 0.333   
#>  4 Nat_NonVeg        38 0.000778
#>  5 Open_Cerrado    5658 0.116   
#>  6 Pasture        12894 0.264   
#>  7 Perennial_Crop    68 0.00139 
#>  8 Silviculture     805 0.0165  
#>  9 Sugarcane       1775 0.0363  
#> 10 Water            263 0.00538
```

## Cross-validation of training set{-}



The first step in analysing the results is to perform cross-validation. Since the dataset is big and highly imbalanced, we use `sits_reduce_imbalance()` to reduce the size and produce a smaller, more balanced sample dataset for the validation examples.


``` r
# Reduce imbalance in the dataset
# Maximum number of samples per class will be 1000
# Minimum number of samples per class will be 500
samples_cerrado_bal <- sits_reduce_imbalance(
  samples = samples_cerrado_lc8,
  n_samples_over = 500,
  n_samples_under = 1000,
  multicores = 4
)

# Show new sample distribution
summary(samples_cerrado_bal)
```


```
#> # A tibble: 10 × 3
#>    label          count   prop
#>    <chr>          <int>  <dbl>
#>  1 Annual_Crop     1000 0.124 
#>  2 Cerradao         884 0.110 
#>  3 Cerrado          980 0.121 
#>  4 Nat_NonVeg       500 0.0619
#>  5 Open_Cerrado     960 0.119 
#>  6 Pasture          972 0.120 
#>  7 Perennial_Crop   500 0.0619
#>  8 Silviculture     805 0.0997
#>  9 Sugarcane        972 0.120 
#> 10 Water            500 0.0619
```

The following code does a five-fold validation using the random forest algorithm. 


``` r
# Perform a five-fold validation for the Cerrado dataset
# Random forest machine learning method using default parameters
val_rfor <- sits_kfold_validate(
  samples = samples_cerrado_bal,
  folds = 5,
  ml_method = sits_rfor(),
  multicores = 5
)

# Print the validation statistics
summary(val_rfor)
```

```
#> Overall Statistics                            
#>  Accuracy : 0.8785          
#>    95% CI : (0.8712, 0.8855)
#>     Kappa : 0.864
```

One useful function of `sits` is the capacity to compare different validation methods and store them in an XLS file for further analysis. The following example shows how to do this using the Cerrado dataset. We take the models: random forest (`sits_rfor()`), extreme gradient boosting (`sits_xgboost()`), temporal CNN (`sits_tempcnn()`), and lightweight temporal attention encoder (`sits_lighttae()`). After computing the confusion matrix and the statistics for each model, we also store the result in a list. When the calculation is finished, the function `sits_to_xlsx()` writes all the results in an Excel-compatible spreadsheet. 


``` r
# Compare different models for the Cerrado dataset
# Create a list to store the results
results <- list()
# Give a name to the results of the random forest model (see above)
val_rfor$name <- "rfor"
# Store the rfor results in a list
results[[length(results) + 1]] <- val_rfor
# Extreme Gradient Boosting
val_xgb <- sits_kfold_validate(
  samples = samples_cerrado_bal,
  ml_method = sits_xgboost(),
  folds = 5,
  multicores = 5
)
# Give a name to the SVM model
val_xgb$name <- "xgboost"
# store the results in a list
results[[length(results) + 1]] <- val_xgb
# Temporal CNN
val_tcnn <- sits_kfold_validate(
  samples = samples_cerrado_bal,
  ml_method = sits_tempcnn(
    optimizer = torch::optim_adamw,
    opt_hparams = list(lr = 0.001)
  ),
  folds = 5,
  multicores = 5
)
# Give a name to the result
val_tcnn$name <- "TempCNN"
# store the results in a list
results[[length(results) + 1]] <- val_tcnn
# Light TAE
val_ltae <- sits_kfold_validate(
  samples = samples_cerrado_bal,
  ml_method = sits_lighttae(
    optimizer = torch::optim_adamw,
    opt_hparams = list(lr = 0.001)
  ),
  folds = 5,
  multicores = 5
)
# Give a name to the result
val_ltae$name <- "LightTAE"
# store the results in a list
results[[length(results) + 1]] <- val_ltae
# Save to an XLS file
xlsx_file <- "./model_comparison.xlsx"

sits_to_xlsx(results, file = xlsx_file)
```

The resulting Excel file can be opened with R or using spreadsheet programs. Figure \@ref(fig:xls) shows a printout of what is read by Excel. Each sheet corresponds to the output of one model. For simplicity, we show only the result of TempCNN, which has an overall accuracy of 90%. 

<div class="figure" style="text-align: center">
<img src="images/k_fold_validation_xlsx.png" alt="Result of 5-fold cross-validation of Mato Grosso data using LightTAE (Source: Authors)." width="90%" height="90%" />
<p class="caption">(\#fig:xls)Result of 5-fold cross-validation of Mato Grosso data using LightTAE (Source: Authors).</p>
</div>

The scores for overall accuracy are similar between the models. However, the models have significant differences, as shown by comparing their F1 scores below.    


``` r
model_acc <- tibble::tibble(
  "Random Forest" = val_rfor$overall[["Accuracy"]],
  "XGBoost"       = val_xgb$overall[["Accuracy"]],
  "TempCNN"       = val_tcnn$overall[["Accuracy"]],
  "LightTAE"      = val_ltae$overall[["Accuracy"]]
)

options(digits = 3)
model_acc
```


```
#> # A tibble: 1 × 4
#>   `Random Forest` XGBoost TempCNN LightTAE
#>             <dbl>   <dbl>   <dbl>    <dbl>
#> 1           0.879   0.889   0.897    0.894
```

The table below shows the F1-scores of all classes for each model, as produced by the k-fold validation. The F1-scores are the harmonic mean between user's accuracy and precision accuracy for each class. The results show that, although deep learning models such TempCNN and LightTAE have similar overall accuracies to random forest or XGBoost, their F1-scores per class are generally better.  


``` r
f1_score_rfor <- unname(val_rfor$byClass[, "F1"])
f1_score_xgb <- unname(val_xgb$byClass[, "F1"])
f1_score_tcnn <- unname(val_tcnn$byClass[, "F1"])
f1_score_ltae <- unname(val_ltae$byClass[, "F1"])

f1_scores <- tibble::tibble(
  "Classes"  = sits_labels(samples_cerrado_bal),
  "RandFor"  = f1_score_rfor,
  "XGBoost"  = f1_score_xgb,
  "TempCNN"  = f1_score_tcnn,
  "LightTAE" = f1_score_ltae
)

f1_scores
```

```
#> # A tibble: 10 × 5
#>    Classes        RandFor XGBoost TempCNN LightTAE
#>    <chr>            <dbl>   <dbl>   <dbl>    <dbl>
#>  1 Annual_Crop      0.909   0.903   0.924    0.912
#>  2 Cerradao         0.878   0.889   0.877    0.882
#>  3 Cerrado          0.746   0.759   0.755    0.748
#>  4 Nat_NonVeg       0.823   0.835   0.838    0.833
#>  5 Open_Cerrado     0.824   0.847   0.854    0.859
#>  6 Pasture          0.917   0.933   0.947    0.931
#>  7 Perennial_Crop   0.999   0.999   0.998    1    
#>  8 Silviculture     0.977   0.976   0.990    0.995
#>  9 Sugarcane        0.998   0.998   1        0.998
#> 10 Water            0.890   0.911   0.945    0.936
```

The cross-validation results have to be interpreted carefully. Cross-validation measures how well the model fits the training data. Using these results to measure classification accuracy is only valid if the training data is a good sample of the entire dataset. In practice, training data is subject to various sources of bias. In most cases of land classification, some classes are much more frequent than others, and as such, the training dataset will be imbalanced. For large areas, regional differences in soil and climate condition will lead the same classes to have different spectral responses. When collecting samples for large areas, field analysts may be restricted to areas where they have access (e.g., along roads). An additional problem is that of mixed pixels. Expert interpreters tend to select samples that stand out in fieldwork or reference images. Border pixels are unlikely to be chosen as part of the training data. For all these reasons, cross-validation results should not be considered indicative of accuracy measurement over the entire dataset. 


## Accuracy assessment of classified images{-}

To measure the accuracy of classified images, `sits_accuracy()` uses an area-weighted technique, following the best practices proposed by Olofsson et al. [@Olofsson2013]. The need for area-weighted estimates arises because the land classes are not evenly distributed in space. In some applications (e.g., deforestation) where the interest lies in assessing how much of the image has changed, the area mapped as deforested is likely to be a small fraction of the total area. If users disregard the relative importance of small areas where change is taking place, the overall accuracy estimate will be inflated and unrealistic. For this reason, Olofsson et al.  argue that "mapped areas should be adjusted to eliminate bias attributable to map classification error, and these error-adjusted area estimates should be accompanied by confidence intervals to quantify the sampling variability of the estimated area" [@Olofsson2013].

With this motivation, when measuring the accuracy of classified images, `sits_accuracy()` follows the procedure set by Olofsson et al. [@Olofsson2013]. Given a classified image and a validation file, the first step calculates the confusion matrix in the traditional way, i.e., by identifying the commission and omission errors. Then it calculates the unbiased estimator of the proportion of area in cell $i,j$ of the error matrix

$$
\hat{p_{i,j}} = W_i\frac{n_{i,j}}{n_i},
$$
where the total area of the map is $A_{tot}$, the mapping area of class $i$ is $A_{m,i}$ and the proportion of area mapped as class $i$ is $W_i = {A_{m,i}}/{A_{tot}}$. Adjusting for area size allows producing an unbiased estimation of the total area of class $j$, defined as a stratified estimator
$$
\hat{A_j} = A_{tot}\sum_{i=1}^KW_i\frac{n_{i,j}}{n_i}.
$$

This unbiased area estimator includes the effect of false negatives (omission error) while not considering the effect of false positives (commission error). The area estimates also allow for an unbiased estimate of the user's and producer's accuracy for each class. Following Olofsson et al. @Olofsson2013, we provide the 95% confidence interval for $\hat{A_j}$. 

To produce the adjusted area estimates, `sits_accuracy()` must get the classified image together with a csv file containing a set of well-selected labeled points. The csv file should have the same format as the one used to obtain samples, as discussed earlier.

The labeled points should be based on a random stratified sample. All areas associated with each class should contribute to the test data used for accuracy assessment. 

Because of the biases inherent in cross-validation of training data, an independent validation dataset should be used to measure classification accuracy. In this case study, Simoes et al.  did a systematic sampling of the Cerrado biome using a $20 \times 20$ km grid with a total of 5,402 points [@Simoes2021]. These samples are independent of the training set used in the classification. They were interpreted by five specialists using high-resolution images from the same period of the classification. This resulted in 5,286 evaluation samples thus distributed: `Annual Crop` (553), `Cerrado` (3,155), `Natural Non Vegetated` (44), `Pasture` (1,246), `Perennial Crop` (38), `Silviculture` (94), `Sugarcane` (77), and `Water` (79). This dataset is available in package `sitsdata`, as described below. In this validation file, all samples belonging to classes `Cerrado`, `Open Cerrado`, and `Cerradao`  (`Woody Savanna`) have been grouped together in a single class. 

The first step is to obtain the classification map. The code for the full classification of the Cerrado biome, using the TempCNN algorithm, is shown below. Because of the large data size, the code will not be executed. 


``` r
# This code shows the classification of the Cerrado biome
# It is included for information purposes
# It takes a long time to run
tcnn_model <- sits_train(
  samples = samples_cerrado_lc8,
  ml_method = sits_tempcnn()
)

# Using the tempCNN model to classify the Cerrado
# This example should be run on a large virtual machine
cerrado_probs_cube <- sits_classify(
  cube = cerrado_cube,
  ml_model = tcnn_model,
  memsize = 128,
  multicores = 64,
  output_dir = "./tempdir/chp11"
)

cerrado_bayes_cube <- sits_smooth(
  cube = cerrado_probs_cube,
  memsize = 128,
  multicores = 64,
  output_dir = "./tempdir/chp11"
)

cerrado_classif <- sits_label_classification(
  cube = cerrado_bayes_cube,
  memsize = 128,
  multicores = 64,
  output_dir = "./tempdir/chp11"
)
```

Since the above code is included for information only, we use the labeled cube available in the `sitsdata` package to perform the accuracy assessment. First, we retrieve the metadata for the cube.


``` r
# Retrieve the metadata for the classified cube
# The files are stored in the sitsdata package
data_dir <- system.file("extdata/Cerrado", package = "sitsdata")
# labels for the classification
labels <- c(
  "1" = "Annual_Crop", "2" = "Cerrado", "3" = "Cerrado", "4" = "Nat_NonVeg",
  "5" = "Cerrado", "6" = "Pasture", "7" = "Perennial_Crop",
  "8" = "Silviculture", "9" = "Sugarcane", "10" = "Water"
)
# Read the cube metadata
cerrado_classif <- sits_cube(
  source = "USGS",
  collection = "LANDSAT-C2L2-SR",
  bands = "class",
  labels = labels,
  data_dir = data_dir,
  parse_info = c("X1", "tile", "band", "start_date", "end_date", "version"),
  progress = FALSE
)
# Plot one tile of the classification
plot(cerrado_classif, tiles = "044048")
```

<div class="figure" style="text-align: center">
<img src="11-validation_files/figure-html/unnamed-chunk-14-1.png" alt="Classification of tile 044048 from the Landsat data cube for the Brazilian Cerrado in 2017/2018 (Source: Authors)." width="100%" />
<p class="caption">(\#fig:unnamed-chunk-14)Classification of tile 044048 from the Landsat data cube for the Brazilian Cerrado in 2017/2018 (Source: Authors).</p>
</div>
The next step is to provide a csv file with the validation points, as described above.


``` r
# Get ground truth points
valid_csv <- system.file("extdata/csv/cerrado_lc8_validation.csv",
  package = "sitsdata"
)
# Calculate accuracy according to Olofsson's method
area_acc <- sits_accuracy(cerrado_classif,
  validation_csv = valid_csv
)
# Print the area estimated accuracy
area_acc
```


```
#> $error_matrix
#>                 
#>                  Annual_Crop Cerrado Nat_NonVeg Pasture Perennial_Crop
#>   Annual_Crop            469      13          0      47              0
#>   Cerrado                  4    2813          0     191              3
#>   Nat_NonVeg               0       2         43       0              0
#>   Pasture                 67     287          0     999              5
#>   Perennial_Crop           0      23          0       2             26
#>   Silviculture             0      16          0       2              4
#>   Sugarcane               13       0          0       5              0
#>   Water                    0       1          1       0              0
#>                 
#>                  Silviculture Sugarcane Water
#>   Annual_Crop               0         2     0
#>   Cerrado                  12         2     4
#>   Nat_NonVeg                0         0     2
#>   Pasture                   3         2     4
#>   Perennial_Crop            2         0     0
#>   Silviculture             77         0     0
#>   Sugarcane                 0        71     0
#>   Water                     0         0    69
#> 
#> $area_pixels
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>       3.12e+07       2.02e+08       9.97e+05       1.09e+08       1.62e+06 
#>   Silviculture      Sugarcane          Water 
#>       6.96e+06       1.06e+07       1.42e+07 
#> 
#> $error_ajusted_area
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>       3.47e+07       2.14e+08       1.11e+06       9.58e+07       1.67e+06 
#>   Silviculture      Sugarcane          Water 
#>       6.51e+06       8.84e+06       1.44e+07 
#> 
#> $stderr_prop
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>       0.002328       0.004194       0.000542       0.004386       0.000735 
#>   Silviculture      Sugarcane          Water 
#>       0.001060       0.001282       0.000931 
#> 
#> $stderr_area
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>         876650        1579665         204161        1651840         276851 
#>   Silviculture      Sugarcane          Water 
#>         399372         483007         350471 
#> 
#> $conf_interval
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>        1718233        3096142         400156        3237606         542628 
#>   Silviculture      Sugarcane          Water 
#>         782769         946693         686924 
#> 
#> $accuracy
#> $accuracy$user
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>          0.883          0.929          0.915          0.731          0.491 
#>   Silviculture      Sugarcane          Water 
#>          0.778          0.798          0.972 
#> 
#> $accuracy$producer
#>    Annual_Crop        Cerrado     Nat_NonVeg        Pasture Perennial_Crop 
#>          0.794          0.880          0.820          0.830          0.474 
#>   Silviculture      Sugarcane          Water 
#>          0.831          0.954          0.956 
#> 
#> $accuracy$overall
#> [1] 0.861
#> 
#> 
#> attr(,"class")
#> [1] "sits_area_assessment" "list"
```
This example shows that it is important to correct area estimates in land classification to reduce the bias effect of misclassification and to consider the different producer's accuracies associated with each class. It also shows that actual overall accuracy is generally lower than the result of cross-validation.  