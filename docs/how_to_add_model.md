# How to add a new power market model?
To connect a new energy market model to FRAM you need to create an API for the model - a Python package that will connect the core model to the optimisation algorithm in the energy market model. Use JulES API package (fram-jules) from our [simple demo]({{ framlinks.demo }}){:target="_blank"} as an example. 

![Connecting new model to FRAM](../assets/model_api_example.svg "Connecting new model to FRAM")

Energy market model is normaly a third-party software that contains the algorithm for solving the market optimisation problem, as well as the model input data and output data in some format. The algorithm for solving the optimisation problem may be implemneted in different language, e.g. GAMS, Julia, Matlab etc. Many models have their own interfaces for handling the input and output data. 

To connect the model to FRAM you have to create an API package that understands the core model, is able to get data from it, send it to the 3-rd party software and start the optimisation. It will also have to read the results of the optimisation and send them back to the core model. The two most important methods you have to implement are **get_config** and **solve** methods. Interfaces for them can be found in [FRAM core]({{ framlinks.core }}){:target="_blank"}:
- framcore.solvers.Solver.py
- framcore.solvers.SolverConfig.py

What is behind these methods may vary widely from model to model, depending on how the optimisation model was set up by its developers. For most of optimisation models, the Solver-method will have to create all necessary input files in some format (e.g. csv, h5 ot txt) and call upon the optimisation software (e.g. GAMS solve or juliacall). It will also have to read output files in some format (e.g. txt) and put data back into the Model() object.  

Note that in many cases it may be easier to pass-by the original "user-friendly" interface of the energy market model in order to avoid redundant steps. It may for example be easier and faster to create input files and call upon the optimisation software directly instead of calling upon some functions in the user interface that will anyway perform the same operations. 

## Do I need to add something to the core model?

If the energy market model you want to connect has data that is not yet supported in the core model, than you have to create new components in the core model (og reuse the existing components if possible). If you create new components, than you also have to send new data from the database to the core mode, i.e. extend the Populator and add new data validation steps. 

The core model is constantly under development, and we are constantluy adding support for new components. If you have added new components that may be useful for others, [tell us about it](mailto:fram@nve.no) so we can extend the core model with your components. 

## How to send data from the core model to the Solver?
Some models operate with "concrete" data models where input data is based on "real-world" definitions (thermal, demand, transmission etc). These models can use existing high-level components in the core model directly. For example, if your model needs a smart house component, try to define it using existing components, e.g. demand, battery and solar plant. You can also define your own components - in this case you have to update the Populator so that it knows about the new components. 

Other models have more "abstract" data models. For example nodes and flows. FRAM core model supports these models as well because all high-level components can be decomponsed into low-level, more abstract components **flow** and **node**. These are the most basic components in the core model, and if you spend some time in the start understanding how they work, you can write "clean" and reusable Python code and avoid a lot of duplicate code. See our examples with JulES model to learn how to use flow and node.




 
 
 



