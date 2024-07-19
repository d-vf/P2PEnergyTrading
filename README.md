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

Notebook: Demand and supply simulation 09_23.ipynb
   * clusters solar, residential and commercial profiles, simulation
     
# Trading pairs
 
 ## Matching process (double auction)
The double auction mechanism is used to incorporate real-world constraints on both the demand and supply sides, whereas $D$ be the set of buy orders and $S$ be the set of sell orders in a P2P market:

* Buy orders: $D = \{ (d_1, p^d_1, q^d_1), \ldots, (d_n, d^b_n, d^b_n) \}$ where each tuple represents a buy order with identifier $d_i$, a willing-to-pay price $p^d_i$, and a desired quantity $q^d_i$.
* Sell orders: $S = \{ (s_1, p^s_1, q^s_1), \ldots, (s_m, p^s_m, q^s_m) \}$ where each tuple represents a sell order with identifier $s_i$, an asking price $p^s_i$, and an available quantity $q^s_i$.

The quantities on each are adjusted at each bus $i$, ${g}_i$ (generation at bus $i$) $\equiv q^s_i$ and adjusted load at bus $i: {\rho}_i \equiv q^d_i$ (or that seller cannot sell more than ${g}_i$ and inversely, the buyer cannot buy more than ${\rho}_i$.

The matching process is described in the Algorithm "Matching_algo".

1.  D ← set of buy orders { (d1, p^d1, q^d1), ..., (d_n, p^d_n, q^d_n) }
2.  S ← set of sell orders { (s_1, p^s_1, q^s_1), ..., (s_m, p^s_m, q^s_m) }
3.  T ← ∅  // initialize matches
4.  Sort D by increasing p^d
5.  Sort S by decreasing p^s
6.  for each buy order (d_i, p^d_i, q^d_i) in D:
7.      if q^d_i = 0:
8.          continue
9.      for each sell order (s_j, p^s_j, q^s_j) in S:
10.         if q^s_j > 0 and p^d_i ≥ p^s_j:
11.             q^m_ij ← min(q^d_i, q^s_j)
12.             t ← (s_j, d_i, q^m_ij)  // form trade tuple
13.             Add t to T
14.             q^d_i ← q^d_i - q^m_ij
15.             q^s_j ← q^s_j - q^m_ij
16.             if q^d_i = 0:
17.                 break
18. Q_total ← Σ_{t ∈ T} q^m_ij  // Calculate total matched quantity


A trade $t \in T$ is represented as $t = (s_t, d_t, q_t)$ where $s_t$ is the seller (source, matching $s_i$ in sell orders), $d_t$ is the buyer (destination, matching $d_i$ in buy orders), and $q_t$ is the quantity of the trade $t$. This ordering scheme simulates an optimal matching: the highest willing buyer is paired with the lowest willing seller. Each user's bid is capped by their respective load for the given time frame, while each ask is constrained by the available local generation. This ensures that the trading process accurately reflects the physical limitations of the energy system. 
     
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
