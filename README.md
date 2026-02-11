# Satellite Poverty Mapping in India (2015)
Replication and adaptation of Jean et al. (*Science*, 2016)

This project replicates and extends the methodology from:

**Jean et al. (2016)** — *Combining satellite imagery and machine learning to predict poverty*  
Science, DOI: 10.1126/science.aaf7894

The replication method is from: Pytorch implementation: https://github.com/jmather625/predicting-poverty-replication

All notebooks run on Google colab and all data are stored at /content/drive/MyDrive/

---

## Project Goal
The objective is to predict proverty from satellite imagery:

1) Train a CNN model on satellite image chips  
2) Extract learned feature embeddings  
3) Use Ridge regression to predict DHS wealth index  
4) Produce a poverty prediction map for India
---

## Data for the baseline replication
- Download nightlights data from https://www.ngdc.noaa.gov/eog/viirs/download_dnb_composites.html. Use the 2015 annual composite in the 75N/060W tile and the 00N/060W tile. Choose the .tif file that has "vcm-orm-ntl" in the name. Save them to viirs_2015_<tile_descriptor>.tif, where tile_descriptor is 75N/060W or 00N/060W.

- Get the LSMS survey data from the world bank. Download the 2016-2017 Malawi survey data, 2015-2016 Ethiopia data, and the 2015-2016 Nigeria data from https://microdata.worldbank.org/index.php/catalog/lsms. The World Bank wants to know how people use their data, so you will have to sign in and explain why you want their data. Make sure to download the CSV version. Unzip the downloaded data into countries/<country name>/LSMS/. Country name should be either malawi_2016, ethiopia_2015, or nigeria_2015.
---

## Replication 
Once the data is properly downloaded and placed in the correct directories, the replication process closely follows the structure of the original repository. The provided notebooks are already configured to handle the survey data, download satellite image patches, train the night-light classification model, extract features, and fit the final regression.

As long as the night-light .tif files and LSMS survey folders match the naming conventions described above, the notebooks should run with little or no modification. The key step is ensuring the scripts can locate the correct tiles and country datasets.

After that, training the CNN and running the prediction workflow should reproduce results that are within a few percentage points of those reported in the paper.

For a direct comparison with the original paper, refer to the table below:

| Country  | Their Year | Their R² | Our Year | Our R² |
|----------|------------|----------|----------|--------|
| Malawi   | 2013       | 0.37     | 2016     | 0.41   |
| Nigeria  | 2013       | 0.42     | 2015     | 0.34   |

---

## Data for the adaption
Due to data access agreements, users need to independently download data files from the Demographic and Health Surveys and the Earth Observation Group websites. These two data sources require the user to register an account and fill in a Data User Agreement form.

- Download DHS data from https://dhsprogram.com/Data/. Download the Standard DHS surveys of India (2015-16). For each survey, download its corresponding Household Recode files in Stata format as well as its corresponding geographic datasets. Zip the survey data and GPS data and name it as "IN_2015_DHS".
- Get an api key from Google Earth Engine API service. This key is required for downloading the Landsat image patches.
- Download the VIIRS Nighttime Lights (2015) data from https://eogdata.mines.edu/nighttime_light/annual/v10/2015/. Choose the tile of 75N060E.
---

## Pipeline Overview

```text
Landsat Chips → CNN (trained with VIIRS nightlight proxy) → 512-d Embeddings → Ridge Regression → Poverty Map
```
---

## Adaption for the India data
Run the Jupyter files in the following order:
1. scripts/process_survey_data_india_2015.ipynb
2. scripts/download_images_india_2015.ipynb
3. scripts/train_cnn_india_2015.ipynb
4. scripts/feature_extract_india_2015.ipynb
5. scripts/predict_india_2015.ipynb

The adaptation aims to address SDG 1 – No Poverty.

For adapting to the India 2015 data, I used DHS cluster-level wealth as the final prediction target and treated VIIRS nighttime light intensity as a proxy supervision signal to train the CNN. Landsat 8 image chips were downloaded from Google Earth Engine with cloud masking, and a ResNet18 model was trained to predict binned nightlight levels, allowing it to learn visual features related to economic development. After training, the network was used to extract 512 dimensional embeddings for each image, which were then averaged at the DHS cluster level. In the final stage, Ridge regression was applied to predict wealth directly from these learned satellite embeddings, evaluated under both random and spatial cross validation. The resulting predictions generate a plausible poverty map that captures broad regional patterns of development across India.

---

## Results 

| model     | cv      |   r2_mean |   mae_mean |
|:----------|:--------|----------:|-----------:|
| Embedding | Random  |  0.460579 |   0.633763 |
| Embedding | Spatial |  0.227704 |   0.722774 |


<img src="Adaption/Results/Predicted%20poverty.png" width="500">

<img src="Adaption/Results/Mapping%20across%20models.png" width="500">

Embeddings on their own explained a reasonable share of the variation in DHS wealth. Under Random CV, the model reached an R² of about 0.46 (MAE ≈ 0.63). When evaluated with the stricter Spatial CV split, performance dropped to an R² of around 0.23 (MAE ≈ 0.72), which suggests that some of the predictive power in the random setting comes from nearby areas looking similar, and that generalizing to new regions is harder. Even so, the spatial results show that the image embeddings still carry useful information related to socio-economic conditions. The predicted poverty patterns broadly match known regional differences across India, with higher deprivation concentrated in parts of the north and central regions. Overall, the results suggest that satellite-based embeddings can provide a meaningful, though moderate, signal for mapping poverty at scale.
