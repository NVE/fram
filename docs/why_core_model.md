---
title: "Why We Need a Core Model"
description: "Learn why FRAM uses a shared core model to unify data, ensure consistency, and simplify the development and comparison of power‑market models."
---

# Why do we need a core model? 
The most important part of FRAM is the [**core model**]({{ framlinks.core }}){:target="_blank"}. This is a Python object that holds the information about all [components](why_components.md) in the modelled energy system. The energy market models access input data through the core model. Different energy models have different internal data models with their own level of detail, which support different modelling features and require different levels of input data detail and data formats. The core model is therefore an [extra layer of data processing](how_fram_works.md) between the database and the model input / output in FRAM. A more detailed technical description of the core model can be found [here]({{ framlinks.core }}){:target="_blank"}. 

You might wonder, why do we need a core model at all? Why won't we just send the data directly from the database to the energy market model? If you are a user of the model framework and only work in a user interface, you might not care because you do not interact with the core model directly. But for developers and users who want to understand the technical side of the system this question is very important. 

The main reasons for having a core model is scalability of the solution and smart data manipulation. 

## Scalability of the solution
From a development perspective, the scalability and possibility to add more energy market models using the same data is an important advantage of the core model. 

Different energy market models have different internal data models with their own level of detail, which may support or not support some modelling features. It would be very difficult to build generic applications (for example, workflow running the analysis and dashboard showing the results) on top of these different data models without this extra data abstraction layer that the core model represents. 

Without the core model we would have to build an application that understands all possible data models and in addition all possible "hacks" that the analysts sometimes use when the model is missing support for some feature. This would not be scalable or reasonable or cost-efficient. Instead, the **core model creates a stable and consistent API that different applications can use without being dependent on the various internal data models**. 

## Smart data manipulation 

Core model is a **descriptive model that uses Python objects** to manipulate data. Therefore it takes advantage of all the strenghts of the object-oriented programming. And if you wonder why not just use Python dataframes to perform all data operations, here comes an explanation. 

Python dataframe is a table where data is stored in columns and rows - a way of representing the data that most of us are familiar with. For example, here is the data for a thermal power plant as a Python dataframe:

_Dataframe with attributes as columns_

{{ read_csv('docs/tables/core_model_attribute_table.csv') }}
 
And here is the same data represented as attributes in Python objects:  

_Attributes as “fields” in a core model component Thermal plant_

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

Unlike dataframes, Python objects store both values *and* the logic for how those values should be interpreted and used, i.e. how each component should behave. Each object knows how to interpret its attributes because each attribute is described as a Python class. For example both **Efficiency()** and **StartUpCost()** in FRAM are Python classes with their own properites. Thermal plant component will therefore know exactly how to calculate efficiency, start-up cost and other attributes correctly when we for example want to aggregate thermal plants. 

We can also easily create different scenarios by creating different instances of an object, for example modifying data about minimum capacity of a Thermal plant. Or we can add new "fields" to support new functionality of a new energy market model that we want to connect to FRAM. 

Results from a model run can also be sent back to the Python object. For example we can store results about the production of the thermal plant in its own "field" in the Thermal plant component, so that it holds data on both input and output at the same time. This is very beneficial when we want to visualise data or compare results from different energy market models with each other.

All the above functionality would be very difficult to achieve with python dataframes. We would have to add new columns and write extensive code that interprets the columns correctly and performs operations on the table in the way you want. For creating scenarios, we would need to handle different versions of the table with different rows or columns. We might also need to add model-specific columns to specify that attributes must be interpreted differently for different models. Handling tables, columns and rows would become complex and not scalable, and it would be harder to keep the data model independent. 

Another important advantage of using Python objects is the **possibility to transform high-level components into low-level components**. This is a simple operation with Python objects and almost impossible to achieve with dataframes. Read here [why high- and low-level components are so important](why_components.md).
