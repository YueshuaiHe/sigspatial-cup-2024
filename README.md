# ACM SIGSPATIAL Cup 2024

The code for the 13th SIGSPATIAL Cup competition (GISCUP 2024) [https://sigspatial2024.sigspatial.org/giscup/index.html](https://sigspatial2024.sigspatial.org/giscup/index.html).

The goal is to optimize the placement of EV charging stations. Our UMN-UL team proposes an approach that analyzes traffic demands and leverages point-of-interest contextual embeddings using a language model to predict EV registration rates per census block. 

## 1. Data preparation 

### Description
We prepare related data for the following steps.

### Usage
- Input
  - Census block shapefile for GA
  `./data/GA_cb_2018_13_bg_500k`
  - Census block shapefile for NC
  `./data/NC_cb_2018_37_bg_500k`
  - Justice 40 shapefile for GA
  `./data/Georgia-Justice40-map`
  - Existing charging stations for GA
  `./data/Existing ev charging stations.csv`


- Code

- Output


## 2. Learning region embeddings to predict EV registration rate for each census block
### Description
We contextualize regions using one of the spatial language models, [SpaBERT](https://github.com/knowledge-computing/spabert). We use publicly available open map data, [Overture Maps](https://overturemaps.org/), to retrieve points of interest (POI) and infrastructure data for North Carolina and Georgia.

### Usage
- Input
  - Overture Maps data in North Carolina and Georgia
    `./data/overturemap_{STATE_NAME}_{place/infrastructure}.csv`

  - SpaBERT's pretrained weights using North Carolina data
    `./poi_contextual/weights`

  - Learned region embeddings for each zip code in North Carolina and census block in Georgia
    `./poi_contextual/embeddings`
    
- Code `python ./src/train_predict_ev_count.py`

- Output
  - Prediction results of EV count for each census block in Georgia


## 3. Predicting EV charging demand at the census block for Georgia

### Description
We used the OD matrix from GA DOT to estimate the EV charging demand per census block. Based on the EV registration estimated from step 2, we know how many EV trips originated from each census block. Then we proportionally assigned those EV trips according to the trip distribution from the OD matrix. Since the OD matrix is sparse, we only kept top 5 destinations with the most trips per origin. At last, the EV charging demand is the maximum of the EV registration number and the EV trips, considering the charging needs for both home and travel.  

### Usage
- Input
  - OD Matrix
  `googledrive`

- Code `python ./src/charging_demand_prediction.ipynb`

- Output
  - Estimated EV demand per census block in Georgia
    `./data/output/ev_demand.csv`


## 4. Assigning EV charging stations based on demand for each census block

### Description
We assign EV charging station locations based on estimated demand and a ranked list of relevant POI types, ordered as follows: `EV stations, Parking, Shopping Centers, Offices, Institutes, Hotels, Parks, Restaurants, Companies, Museums, and Golf Courses` EV charging stations are categorized into three capacity levels—1, 4, and 8—based on EV demand quantiles: 50%, 60%-80%, and 90%, respectively. 

### Usage
- Input
  - Estimated EV demand per census block in Georgia
    `./data/output/ev_demand.csv`

  - Existing EV charging stations in Georgia
    `./data/ev_stations_georgia.csv`

  - Filtered Overture Maps data in Georgia
    `./data/overturemap_georgia_potential_ev_stations.csv`

- Code `python ./src/assign_ev_station.py`

- Output
  - Assigned EV charging stations (GPKG format)
    `./data/output/ev_charging_stations_UMN_UL.gpkg`


## 5. Adjusting EV charging stations for disadvantaged communities

### Description
We estimated the charging station distributions between disadvantaged communities (DACs) and non-disadvantaged communities (Non-DACs) according to the definition of Justice40. We found that the number of charging stations are (Mean/std):
- DAC: 2.45/4.70
- Non-DAC: 2.62/5.08
The comparison result indicates a reasonable charging station distribution. 

### Usage
- Input
  - Justice 40
  `./data/Georgia-Justice40-map`

- Code `python ./src/charging_demand_prediction.ipynb`

- Output
