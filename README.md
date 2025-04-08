# Domain-informed Models for Degradation and Prognostics in Hydro Components

**Domain-informed Models for Degradation and Prognostics in Hydro Components** is an open-source prototype tool for degradation and prognostics for hydro components. Example usages are demonstrated on pilot component - bearings. 

![image](https://github.com/ANL-CEEESA/Domain-informed-Models-for-Degradation-and-Prognostics-in-Hydro-Components/assets/62155196/4b264571-4c2b-42f6-94f8-6abd3737016d)

![image](https://github.com/ANL-CEEESA/Domain-informed-Models-for-Degradation-and-Prognostics-in-Hydro-Components/assets/62155196/f7b34382-cfe6-4aa4-8296-81a1cb24e431)

This is the code used for domain informed degradation detection and prognostics for HRI dataset. The current version of the code utilized operational data and performs analysis on the turbine's guide bearings. 

Dataset used: a) Vibration data from Turbine Guide Bearings, and b) Power data from Turbine Guide Bearings

The code has three submodules: 
1. Raw Data Preprocessing 
2. Degradation Detection 
3. RUL Estimation 
### RAW DATA PREPROCESSING:

Raw data from the HRI dataset contains anomalous reading and data from non-operational time period of the powerhouse. Therefore, a preprocessing of the raw data is needed to ensure a smooth data profile before proceeding with the analysis. 

The raw data preprocessing is done using RUL_Estimation/preprocessing.py code. It involves changing the timestamps to a python readable time format, and removing the unusually low or high vibration/power readings from the data. The code contains comment and is easily interpretable. 


### DEGRADATION DETECTION 

#### PREPROCESSING 
The preprocessing under the degradation detection submodule is mentioned in Degradation_Detection/preprocessing.py file. It consists of five different functions: 

a. **Converting KW to MW**: This is mentioned in the function convert_kw_to_mw. Some power data has the unit mentioned in KW. Therefore the raw data preprocessing module converts them to MW. 

b. **Preprocess Vibration**: This is mentioned in the function preprocess_Vibration. Not all vibration data has same length. Therefore converting them to pandas dataframe generates NaN value. The preprocess_Vibration converts those NaN to zero values. Later, the zero values are removed from our analysis. We assume the powerplant is not operational during those period.

c. **Finding common powerhouse**: find_common_powerhouses is used for this functionality. The power data and vibration data of turbine is not sorted in the same order. There are many common powerhouses in both the datasets. We utilize only those common power houses. 

d. **Creating clean csv files each for both power and vibration**: This is done by create_clean_csv. It runs the above three functions and create a cleaner csv file one each for power and vibration. 

e. **Matching timestamps of power and vibration data**: This is done by trim_power_file. In our analysis we found the vibration data to have a longer time range than the power data. Hence we trimmed the cleaner power csv file to match the time stamps of the vibration data. 

#### CORRELATION ANALYSIS 

Once the cleaner (and time stamped matched) power and vibration data are available, the next objective is to analyze the correlation between the data. The power data can explain the vibration only if there is a non-zero correlation between them. However, it must be kept in mind that both power and vibration are time-series dataset. Therefore a rolling window based correlation must be done to check the temporal correlations. It must also be noted that these correlation analysis are done per powerhouse. Therefore, it is theoretically possible that for some powerhouses power and vibration are correlated but not for others, depending on the operational HRI data.  

The code for doing correlation analysis is found in Degradation_Detection/CorrelationAnalysis.py. It utilizes a rolling median based pearson correlation analysis to analyze and plot the correlations between power-vibration of the turbine. 

#### DEGRADATION DETECTION (MAIN CODE)

The pearson correlation is a simple mechanism to analyze the inter-dependencies between power and vibration data of a powerhouse. However, the true correlation might be complex enough for pearson correlation to visualize. Therefore, we utilize a neural network based method to analyze the correlation and subsequently flag abnormal vibrations in a turbine. 

This functionality is mentioned in Degradation_Detection/DegraationDetection.py code. It involves training a LSTM that takes input of the past timesteps of both power and vibration to predict the future vibrations. The current version in the code utilizes last 90 days of power and vibration dat to predict the next 30 days of data. The LSTM training parameters are chosen at random and cross-validation is done manually. Users can do their own cross-validation depending on the dataset. 

Once the LSTM is trained, for the test dataset we provide power and vibration timeseries and put a threshold on the residuals (square of L2 norm of prediction - true) of the predicted vibration. High residuals are inferred as abnormal vibrations i.e., observations that could not be explained due to power generated by the powerhouse. 
### RUL ESTIMATION 

Once the degraded turbine bearings are detected using the "Degradation Detection" submodule, the next task is to predict its Remaining Useful Life probabilistically. We utilize a random mixed coefficient model with exponential functional form for bearing vibrations. In addition the random parameters in the model are updated in a Bayesian manner using the most recent vibration data. The code is mentioned in RUL_Estimation/Prognostics/ExponentialModel.py.

The code outputs a Remaining Lifetime Distribution (RLD) and the median of that distribution as the RUL of the bearing. Prediction errors are also plotted to understand the confidence interval of the prediction. The code for plotting those are mentioned in RUL_Estimation/ResultsPlotting.py. 

## Authors
* **Ayush Mohanty** (Wayne State University)
* **Shijia Zhao** (Argonne National Laboratory)
* **Murat Yildirim** (Wayne State University)
* **Feng Qiu** (Argonne National Laboratory)

## Acknowledgments

* This material is based upon work supported by the U.S. Department of Energy, Office of Energy Efficiency and Renewable Energy (EERE), specifically the **Water Power Technology Office (WPTO)**, under hydro power lab call project "A New Generation of Domain-Informed Models for Degradation and Prognostics in Hydro-Components: Co-Developing an Open Source Tool by Harnessing HRI Data, Industry Records and Physical Models". 


## License

```text
Domain-informed Models for Degradation and Prognostics in Hydro Components.
Copyright © 2023-2024, UChicago Argonne, LLC. All Rights Reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted
provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of
   conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of
   conditions and the following disclaimer in the documentation and/or other materials provided
   with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors may be used to
   endorse or promote products derived from this software without specific prior written
   permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR
IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY
AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
```
