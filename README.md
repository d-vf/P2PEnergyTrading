# P2P Energy Trading

 ![Methods](https://github.com/d-vf/P2PEnergyTrading/blob/main/bids_bus_R.jpg)

This repository contains the code and data files for the article "Peer-to-Peer (P2P) Electricity Markets for Low Voltage Networks" by Diana Vieira Fernandes, Nicolas Christin, and Soummya Kar.

Paper submitted and accepted to the 15th IEEE International Conference on Smart Grid Communications ([SmartGridComm 2024](https://sgc2024.ieee-smartgridcomm.org/about))

  
## Simulation

## Data

Build a baseline to compare trading and not trading

Baseline = loads are fulfilled by retailer, so this is demand

###  Loads (NREL)

typical load profiles (households, commercial)

NREL: https://www.nrel.gov/buildings/end-use-load-profiles.html
Demand: (End Use Load Profiles for the U.S. Building Stock) https://data.openei.org/submissions/4520 (PA, ISO, PJM aggregate TS)
National Renewable Energy Laboratory (NREL). (2021). End-Use Load Profiles for the U.S. Building Stock [data set]. Retrieved from https://dx.doi.org/10.25984/1876417.

https://data.openei.org/s3_viewer?bucket=oedi-data-lake&prefix=nrel-pds-building-stock%2Fend-use-load-profiles-for-us-building-stock%2F2023%2Fcomstock_amy2018_release_1%2Ftimeseries_aggregates%2Fby_iso_rto_region%2Fupgrade%3D0%2Fiso_rto_region%3DPJM%2F

oedi-data-lakenrel-pds-building-stockend-use-load-profiles-for-us-building-stock2023comstock_amy2018_release_1timeseries_aggregatesby_iso_rto_regionupgrade%3D0iso_rto_region%3DPJM

  ![residencial cluster](https://github.com/d-vf/P2PEnergyTrading/blob/main/residential_cluster.png) 

### Generation (Solar Atlas)
Supply (do 10, 30 and 50 of buses)

Solar https://globalsolaratlas.info/map (assuming 3kwp)

  ![solar cluster](https://github.com/d-vf/P2PEnergyTrading/blob/main/solar_cluster.png) 

Other:

https://www.nrel.gov/grid/solar-resource/renewable-resource-data.html

https://nrel.github.io/PyDSS/Extended%20controls%20library.html

### network

* CIGRE low voltage radial distribution network (44 bus system)
* 
  <img src="https://raw.githubusercontent.com/d-vf/P2PEnergyTrading/main/44_network.png" alt="network_44" width="50%">

* Synthetic Voltage Control LV Networks ``Village'' (80 bus system)
* 
  <img src="https://raw.githubusercontent.com/d-vf/P2PEnergyTrading/main/80_network.png" alt="network_80" width="50%">


Trading pairs
 
* matching (double auction)
* 24 time block
    

Notebook: Demand and supply simulation 09_23.ipynb
   * clusters solar, residential and commercial profiles, simulation
     
## optimization

Notebook:

A. single_timestep_LP_ DC_aprox.ipynb
   * single timestep LP DC approximation for testing purposes

B. P2P_24_hour__block_LP_DC_aprox.ipynb

### Functions
   
    def`update_network_for_hour`
       Function to update the network for a specific hour
       this generates the simulation for each hour to do the matching, all nets are a list with network data for each hour
    
    def `match_orders`
        matching for each hour block
    
     def `correct_baseline`
    Correcting baseline function (PF all nodes (Grid) +  P2P) -> Network state without P2P effect
    
    def `run_optimization`
    Function to run optimization for a specific hour (takes, network state for each hour and matches for that hour from previous def)
    (net, B, adj_matrix, matches_hour)

### Other 
4. R plots: https://github.com/nc2y/P2PEnergyTrading/tree/main/data
