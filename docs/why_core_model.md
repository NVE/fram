# Why do we need a core model? 
The most important part of FRAM is the **core model**. This is a Python object that holds the information about all [components](why_components.md) in the modelled energy system. The energy market models access input data through the core model. Different energy models have different internal data models with their own level of detail, which support different modelling features and require different levels of input data detail and data formats. The core model is therefore an [extra layer of data processing](how_fram_works.md) between the database and the model input / output in FRAM. 

You might wonder, why do we need a core model at all? Here we will try to answer this question by describing the main advantages of having a core model: 
-	the scalability of the framework
-	smart data manipulation 
-	the possibility to transform data between [high-level and low-level components](why_components.md)

The following information is most relevant for developers and others interested in why core was developed as the central part of FRAM. FRAM users are not expected to interact with the core model directly.

## Scalability of the solution
From a development perspective, the scalability and possibility to add more energy market models using the same data is an important advantage of the core model. 

Because the core model acts as a shared abstraction layer, it allows FRAM to support many different energy market models without each application needing to understand their internal data structures. As mentioned, different energy market models have different data models. It would be very difficult to build generic applications (for example, workflows running the analysis and dashboard showing the results) on top of these different data models without the extra data abstraction layer that the core model represents.

Without the core model we would have to build an application that understands all possible data models and in addition all possible "hacks" that the analysts sometimes use when the model is missing support for some feature. This would not be scalable, reasonable or cost-efficient. Instead, the core model creates a stable and consistent API that different applications can use without being dependent on the various internal data models.

## Smart data manipulation 
The FRAM core model allows manipulation and aggregation of data for different energy market models in a flexible and robust way. As mentioned, energy market models have different data needs. For example, one model might require aggregated hydropower, while another model requires detailed data about each hydropower plant. Based on the same input data, data manipulations through the core model can perform the aggregation and calculate all parameters correctly for both models. 

Model results from a solved model can be sent back to the core model to perform “backwards” data manipulation, for example disaggregating model results so that they can be compared with results from other energy market models. See also [Architecture of FRAM](how_fram_works.md/#architecture-of-fram) for more about "backwards" data manipulation.

Examples of data manipulations that the core model can perform are:
* aggregation and disaggregation,
* transformation of geographical resolution, 
* time resolution, and 
* measurement units. 

FRAM starts with tabular input data, which the core model translates into Python objects. Below we explain why this object‑based approach is more suitable for complex data manipulation than using tables directly.

### Data manipulation using core model compared to data tables
Working with attributes as object fields in a Python object is smarter than than columns in a table. When you want build a specific energy market model, to run differently configured models or to create different scenarios, it is easier to stage the data using Python objects rather than by using tables.

__Manipulation with tables__  
Tables are a familiar way to store data. In the table below, each row represents a component (for example, a thermal power plant) and each column represents an attribute such as efficiency or startup cost. 

_Table with attributes as columns_

{{ read_csv('docs/tables/core_model_attribute_table.csv') }}
 
Tables work well as long as the data is static. But when you start building multiple model configurations, adding new assumptions, or creating many scenarios, the table must constantly be expanded. New concepts often require new columns, and each scenario may require its own modified version of the table. Over time the table, and the code that manipulates it, becomes wider, harder to read, and tightly linked to the specific models that use it.

__Manipulation with Python objects__  
The information in the table above can also be represented as attributes in a Python object:  

_Attributes as “fields” in a core model component (Python object)_  
```text
{plant 1:  
    efficiency: 0.5,  
    startup_cost: 100,  
    min_capacity: 40,  
    …  
    },  
{plant 2:  
    efficiency: 0.4,  
    startup_cost: 150,  
    min_capacity: 50,
    …  
    }  
```

At first glance the two approaches look similar. The important difference is that a table only stores values, while a Python object stores both values *and* the logic for how those values should be interpreted and used.

When storing data in Python objects, these objects also need to be expanded when new or modified data is added. The difference is that a Python object can describe not only what a component is, but also how it should behave. Each object can know how to interpret its own different attributes. The object can 
* validate its own data,  
* compute derived values, for example aggregation of the data, and  
* apply scenario adjustments,  

without needing extra columns or creating duplicates. When you introduce a new modelling concept, you can extend the object’s logic, rather than adding more fields. 

This means you can create different scenarios simply by modifying or extending the object, instead of creating new versions of the table. The same component definition can be reused across many models without adding model‑specific columns or duplicating data. 

__Why FRAM uses objects in the core model__  
The use of Python objects in FRAM core keep the data clean, reduce the need for multiple table versions, and allow behaviour to be defined in one place rather than scattered across many columns. This makes it easier to connect to multiple energy market models and to manage large numbers of scenarios without letting the data structure grow out of control.

## Transformation between high-level and low-level components
The last big advantage of using Python objects is the possibility to transform high-level components (e.g., PowerPlant, Demand, Hydropower) into low-level, more abstract components - flow and node. [Read here to learn why this is so important](why_components.md).
