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

\subsection{Matching process}
The double auction mechanism is used to incorporate real-world constraints on both the demand and supply sides, whereas $D$ be the set of buy orders and $S$ be the set of sell orders in a P2P market:
\begin{itemize}
  \item Buy orders: $D = \{ (d_1, p^d_1, q^d_1), \ldots, (d_n, d^b_n, d^b_n) \}$ where each tuple represents a buy order with identifier $d_i$, a willing-to-pay price $p^d_i$, and a desired quantity $q^d_i$.
  \item Sell orders: $S = \{ (s_1, p^s_1, q^s_1), \ldots, (s_m, p^s_m, q^s_m) \}$ where each tuple represents a sell order with identifier $s_i$, an asking price $p^s_i$, and an available quantity $q^s_i$.
\end{itemize}
The quantities on each are adjusted at each bus $i$, ${g}_i$ (generation at bus $i$) $\equiv q^s_i$ and adjusted load at bus $i: {\rho}_i \equiv q^d_i$ (or that seller cannot sell more than ${g}_i$ and inversely, the buyer cannot buy more than ${\rho}_i$.
The matching process is described in Algorithm~\ref{alg:Matching_algo}.
\begin{algorithm}
\caption{Market Matching Algorithm}
\label{alg:Matching_algo}
\begin{algorithmic}[1]\small
\State $D \gets \text{set of buy orders } \{(d_1,p^d_1,q^d_1),\ldots,(d_n,p^d_n,q^d_n)\}$
\State $S \gets \text{set of sell orders } \{(s_1,p^s_1,q^s_1),\ldots,(s_m,p^s_m,q^s_m)\}$
\State $T \gets \emptyset$ \Comment{initialize matches}
\State Sort $D$ by increasing $p^d$
\State Sort $S$ by decreasing $p^s$
\For{each buy order $(d_i,p^d_i,q^d_i)$ in $D$}
    \If{$q^d_i=0$}
        \State \textbf{continue}
    \EndIf
    \For{each sell order $(s_j,p^s_j,q^s_j)$ in $S$}
        \If{$q^s_j>0$ and $p^d_i\geq p^s_j$}
            \State $q^m_{ij} \gets \min(q^d_i,q^s_j)$
            \State $t \gets (s_j,d_i,q^m_{ij})$ \Comment{form trade tuple}
            \State Add $t$ to $T$
            \State $q^d_i \gets q^d_i-q^m_{ij}$
            \State $q^s_j \gets q^s_j-q^m_{ij}$
            \If{$q^d_i=0$}
                \State \textbf{break}
            \EndIf
        \EndIf
    \EndFor
\EndFor
\State $Q_{\text{total}} \gets \sum_{t\in T} q^m_{ij}$ \Comment{Calculate total matched quantity}
\end{algorithmic}
\end{algorithm}
    

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
