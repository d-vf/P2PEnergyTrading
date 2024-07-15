# P2PEnergyTrading

![Methods](https://github.com/nc2y/P2PEnergyTrading/blob/main/implementation/bids_bus_R.jpg) 

## Explain Simulation

* loads
* matching (double auction)

## Explain actual optimization

## Metrics

# List:

1. single_timestep_LP_ DC_aprox.ipynb
   * single timestep LP DC approximation for testing purposes

2. Demand and supply simulation 09_23.ipynb
   * clusters solar, residential and commercial profiles, simulation
  
3. P2P_24_hour__block_LP_DC_aprox.ipynb (being updated)
   
    def`update_network_for_hour`
       Function to update the network for a specific hour
       this generates the simulation for each hour to do the matching, all nets are a list with network data for each hour
    
    def `match_orders`
        matching for each hour block
    
     def `correct_baseline`
    Correcting baseline function (it is PF of everyone + whatever P2P) Network state without P2P
    
    def `run_optimization`
    Function to run optimization for a specific hour (takes, network state for each hour and matches for that hour from previous def)
    (net, B, adj_matrix, matches_hour)
