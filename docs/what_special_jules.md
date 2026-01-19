# What is special about JulES?
JulES is a fundamental energy market simulation model for operational planning, that uses TuLiPa as building blocks, developed by NVE and applied in the FRAM demo.

The model is specifically **designed to handle an extremely large number of storage units, which is essential for hydropower modelling**. This requires an algorithm that applies simplifications without compromising the quality of the results.  

Unlike approaches such as SDDP or SDP, JulES does not attempt to compute an optional strategy for the entire planning horizon in advance. Instead, it **computes strategies dynamically as the simulation progresses, dividing the problem into smaller subproblems and using parallelization**. We believe this offers better scalability.   

The flow in the algorithm is illustrated in the figure below. From each starting point, the simulation proceeds in three main steps:  

1.	**Price prognosis:** JulES generates price forecasts for each weather scenario (often 30 years) at multiple time scales. The price projections have varying lengths and granularities. They use the same time horizon, but the time resolution is most detailed in the short-term and becomes coarser the further ahead we look on the horizon (less detail for long-term). It is solved efficiently by aggregating over time and space, and solving the weather scenarios in parallel. Along with coarser time resolution long-term, an important aggregation is that the price forecasts use aggregated hydropower, while the next step uses more detail. The time scales are:  
    * Long-term: up to five years ahead  
    * Medium-term: one year ahead  
    * Short-term: one week 

2.	**Storage values:** For all storage systems that must anticipate future conditions to make bids – such as reservoirs and other hydro assets, JulES solves stochastic programming problems using Bender’s decomposition. The resulting storage values for all storages/reservoirs, are then used as inputs to the market clearing step.

3.	**Market clearing:** Finally, the market is cleared based on the updated state. At this point, all relevant information is available: short-term conditions and updated storage values. The full market is cleared in a single joint step, with all units and constraints included.  

When the three steps have been performed, the market clearing feeds into a new starting state, for which the process is continuously repeated. By dividing the problem into separate steps and applying parallelization, JulES achieves good scalability.  

In addition, JulES computes inflow models based on the current starting state. As the figure shows, these inflow estimates feed into the price prognosis and storage‑value calculations in steps 1–2.  This forms part of JulES’ data‑driven forecasting framework, which continuously updates continuously updates future scenarios based on the initial state. 

![Illustration JulES algoritm](img/jules_algorithm.svg)

Currently, the model includes simple demand with reservation prices, and batteries to represent short-term storage and demand shifting. In addition, JulES can handle sector couplings (fire wood, substitution effects).

For documentation and code for the JulES model, see [JulES on Github]({{ juleslinks.jules }}){:target="_blank"}.

For more information about the JulES API connecting FRAM to JulES, see [FRAM JulES]({{ framlinks.julesAPI }}){:target="_blank"}.
