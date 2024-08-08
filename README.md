# P2P Energy Trading

 ![Methods](https://github.com/d-vf/P2PEnergyTrading/blob/main/Assets/bids_bus_R.jpg)

This repository contains the code and data files for the article "Peer-to-Peer (P2P) Electricity Markets for Low Voltage Networks" by Diana Vieira Fernandes, Nicolas Christin, and Soummya Kar.

Paper submitted and accepted to the 15th IEEE International Conference on Smart Grid Communications ([SmartGridComm 2024](https://sgc2024.ieee-smartgridcomm.org/about)). 

Pre-print: [arXiv](https://arxiv.org/abs/2407.21403)

## Libraries

#### K-means clustering (loads)

```
pip install sklearn
```

Scikit-learn documentation: "2.3.2. K-means" - https://scikit-learn.org/stable/modules/clustering.html#k-means

https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html#

#### pandapower

```
pip install pandapower
```
L. Thurner, A. Scheidler, F. Schäfer et al, pandapower - an Open Source Python Tool for Convenient Modeling, Analysis and Optimization of Electric Power Systems, in IEEE Transactions on Power Systems, vol. 33, no. 6, pp. 6510-6521, Nov. 2018.

#### gurobipy (Gurobi Python API*)

```
pip install gurobipy
```

Gurobi Optimization Inc., Gurobi Optimizer, 2024 (https://www.gurobi.com)

* https://pypi.org/project/gurobipy/


[requirements.txt](implementation/requirements.txt) 


## Simulation

### Data

####  Loads (NREL)

Typical load profiles (households, commercial)

NREL: https://www.nrel.gov/buildings/end-use-load-profiles.html

Demand: (End Use Load Profiles for the U.S. Building Stock) https://data.openei.org/submissions/4520 (PA, ISO, PJM aggregate TS)
National Renewable Energy Laboratory (NREL). (2021). End-Use Load Profiles for the U.S. Building Stock [data set]. Retrieved from https://dx.doi.org/10.25984/1876417.


[Residential | ResStock](https://data.openei.org/s3_viewer?bucket=oedi-data-lake&prefix=nrel-pds-building-stock%2Fend-use-load-profiles-for-us-building-stock%2F2023%2Fcomstock_amy2018_release_1%2Ftimeseries_aggregates%2Fby_iso_rto_region%2Fupgrade%3D0%2Fiso_rto_region%3DPJM%2F)

[Commercial | ComStock](oedi-data-lakenrel-pds-building-stockend-use-load-profiles-for-us-building-stock2023comstock_amy2018_release_1timeseries_aggregatesby_iso_rto_regionupgrade%3D0iso_rto_region%3DPJM)


  ![Residential cluster](https://github.com/d-vf/P2PEnergyTrading/blob/main/Assets/residential_cluster.png) 

#### Generation (Solar Atlas)

Supply | Generation

Solar https://globalsolaratlas.info/map (assuming 3kWp)

  ![solar cluster](https://github.com/d-vf/P2PEnergyTrading/blob/main/Assets/solar_cluster.png) 

Other:

NREL [Solar Integration Data and Tools](https://www.nrel.gov/grid/solar-resource/renewable-resource-data.html)


#### Network

* CIGRE low voltage radial distribution network (44 bus system)
  
  <img src="https://raw.githubusercontent.com/d-vf/P2PEnergyTrading/main/Assets/44_network.png" alt="network_44" width="50%">


* Synthetic Voltage Control LV Networks ''Village'' (80 bus system)


  <img src="https://raw.githubusercontent.com/d-vf/P2PEnergyTrading/main/Assets/80_network.png" alt="network_80" width="50%">
     
 
### Matching process (double auction) | Trade generation

The double auction mechanism is used to incorporate real-world constraints on both the demand and supply sides, whereas $D$ be the set of buy orders and $S$ be the set of sell orders in a P2P market:

* Buy orders: $D = \{ (d_1, p^d_1, q^d_1), \ldots, (d_n, d^b_n, d^b_n) \}$ where each tuple represents a buy order with identifier $d_i$, a willing-to-pay price $p^d_i$, and a desired quantity $q^d_i$.
* Sell orders: $S = \{ (s_1, p^s_1, q^s_1), \ldots, (s_m, p^s_m, q^s_m) \}$ where each tuple represents a sell order with identifier $s_i$, an asking price $p^s_i$, and an available quantity $q^s_i$.

The quantities on each are adjusted at each bus $i$, ${g}_i$ (generation at bus $i$) $\equiv q^s_i$ and adjusted load at bus $i: {\rho}_i \equiv q^d_i$ (or that seller cannot sell more than ${g}_i$ and inversely, the buyer cannot buy more than ${\rho}_i$.

The matching process is described in the Algorithm "Matching_algo".

1. D = set of buy orders $(d_1, p^d_1, q^d_1), ..., (d_n, p^d_n, q^d_n)$
2. S = set of sell orders $(s_1, p^s_1, q^s_1), ..., (s_m, p^s_m, q^s_m)$
3. T = empty set (to store matches)

4. Sort D by increasing $p^d$ (price offered by buyers)
5. Sort S by decreasing $p^s$ (price offered by sellers)

**Matching Loop:**

6. For each buy order $(d_i, p^d_i, q^d_i) \in D$:
    * If $q^d_i$ (quantity demanded by buyer) is already 0, skip to the next buy order.
    * For each sell order $(s_j, p^s_j, q^s_j) \in S$:
        * Check if $q^s_j$ (quantity offered by seller) is positive and $p^d_i$ (buyer's price) is greater than or equal to $p^s_j$ (seller's price).
            * If conditions met:
                * Calculate traded quantity $q^m_ij$ as the minimum of $q^d_i and q^s_j$.
                * Create a trade tuple $t = (s_j, d_i, q^m_{ij})$. 
                * Add $t$ to the set of matches $T$.
                * Update remaining quantities:
                    * $q^d_i$ is reduced by $q^m_{ij}$.
                    * $q^s_j$ is reduced by $q^m_{ij}$.
                    * If $q^d_i$ becomes 0 after the trade, the buyer is satisfied, so move to the next buy order.

7. Calculate the total matched quantity $Q_{Total}$ by summing $q^m_{ij}$ for all trades in $T$.


A trade $t \in T$ is represented as $t = (s_t, d_t, q_t)$ where $s_t$ is the seller (source, matching $s_i$ in sell orders), $d_t$ is the buyer (destination, matching $d_i$ in buy orders), and $q_t$ is the quantity of the trade $t$. This ordering scheme simulates an optimal matching: the highest willing buyer is paired with the lowest willing seller. Each user's bid is capped by their respective load for the given time frame, while each ask is constrained by the available local generation. This ensures that the trading process accurately reflects the physical limitations of the energy system. 

#### `match_orders`

```python
def match_orders(net):
    """
    Function to match buy and sell orders for each hour block.

    Parameters:
    net (dict): Network state containing load and generation data.

    Returns:
    tuple: Two lists containing matches for buyers and sellers.
    """

```
     
#### `update_network_for_hour`

```python
def update_network_for_hour(hour, nets):
    """
    Function to update the network for a specific hour.

    Parameters:
    hour (int): The hour for which the network needs to be updated.
    nets (list): A list containing network data for each hour.

    Returns:
    net (dict): Updated network state for the specified hour.
    """
```

#### `correct_baseline`

```python
def correct_baseline(network):
    """
    Function to correct the baseline (Power Flow for all nodes in the grid and P2P) to get the network state without P2P effect.

    Parameters:
    network (dict): The current network state.

    Returns:
    corrected_network (dict): Network state after correcting for P2P effects.
    """
```
  
#### `run_optimization`
    
```python
def run_optimization(net, B, adj_matrix, matches_hour):
    """
    Function to run optimization for a specific hour.
    
    Parameters:
    net (object): Network state for the specific hour.
    B (matrix): Susceptance matrix.
    adj_matrix (matrix): Adjacency matrix of the network.
    matches_hour (list): Matches for the hour from the previous matching function.

    Returns:
    results: The optimization results for the given hour.
    """
```


  
  <img src="https://raw.githubusercontent.com/d-vf/P2PEnergyTrading/main/Assets/newplot.png" alt="obg_gap" width="50%">

## Output

  <img src="https://raw.githubusercontent.com/d-vf/P2PEnergyTrading/main/Assets/p2pimpact_44_Times_New_Roman.png" alt="p2p_44" width="100%">

  

## [Notebooks](implementation/)

A. Demand and supply simulation 09_23.ipynb [here](implementation/Demand_and_supply_simulation_09_23.ipynb)

 * Description:  Simulates demand and supply patterns (using K-means clustering) for solar, residential, and commercial profiles.


B. Single_timestep_LP_DC_aprox_210724.ipynb [here](implementation/Single_timestep_LP_DC_aprox_210724.ipynb)

   * Description: Contains a single timestep Linear Programming (LP) DC approximation for testing purposes.


C. P2P_24_hour_block_LP_DC_aprox_21072024.ipynb [here](implementation/P2P_24_hour_block_LP_DC_aprox_21072024.ipynb)

   * Description: Handles the optimization process for a 24-hour block using the LP DC approximation method.
     
### Other

4. R (ggplot2) plots: [here](implementation/README.md)
