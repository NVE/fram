# Why do we need a core model? 
In FRAM, the energy market models access data through a **core model** - a Python object that holds the information about all [components](why_components.md) in the data. The core model is therefore [an extra layer of data processing](how_fram_works.md) between the database and the model input / output in FRAM.

You might wonder, why do we need a core model at all? Here we will try to answer this question by describing the main advantages of having a core model - smarter data manipulation, possibility to transform data between high-level and low-level components and scalability of the framework.

## Why is data manipulation using a core model smarter? 
The FRAM core model allows to manipulate and aggregate data for specific energy market models in a smart way. For example, one model might require aggregated hydropower, while another model requires detailed data about each hydropower plant. 

-	Data manipulations through the core model will perform the aggregation and calculate all parameters correctly. 
-	Also, model results can be sent back to the core model to perform the “backwards” data manipulation, for example disaggregate model results so that they can be compared with results from other energy market models. 
-	Other examples of data manipulations that the core model can perform are transforming geographical resolution, time resolution and measurement units of the data. 

**But why won’t we just use data tables to manipulate the data directly and send it directly to the energy market models?**

First of all, working with attributes is much smarter when they are defined as fields in a Python object rather than columns in a table. For example, you can have a table with thermal power plants and many columns describing different parameters (attributes) of these power plants:  

_Table with attributes as columns_  

{{ read_csv('docs/tables/core_model_attribute_table.csv') }}

Or you can have the same data as attributes in a Python object:  

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

When you want build a specific energy market model or to run differently configured models or to create different scenarios, it is much easier to stage the data using Python objects than using tables:  

* _Manipulation with tables_  
With tables you will need to add new columns, write code that interprets the columns correctly and performs operations on the table in the way you want. For creating scenarios, you will need to modify the data in table rows or columns and handle different versions of the table. Also, if you want to run the same scenario in different energy market models, you may need to add model-specific columns to specify that attributes must be interpreted differently for different models. Handling tables, columns and rows therefore becomes large and complex, and it is difficult to keep the data model independent.

* _Manipulation with Python objects_  
The same can be done much more efficiently and elegantly using Python objects. Each object would know how to interpret different attributes, because each attribute is already described as a Python class. Each component would know how to populate itself with data, how to aggregate it, and so on. You can also easily create different variants of the same component by modifying the Python object, for example to create different scenarios. There is no need to add more information to the table and make it constantly bigger. This is why we implemented the core model in FRAM – we wanted to work with data in a smarter way, representing data as attributes in Python objects.

## Transformation between high- and low-level components
The second big advantage of using Python objects is the possibility to transform high-level components (e.g. PowerPlant, Demand, Hydropower) into low-level, more abstract components - flow and node. [Read here to learn why this is so important](why_components.md).

## Scalability of the solution
And here is an explanation of the advantages of the core model from the development perspective.

It is necessary to have an abstract layer of data processing in order to be able to connect many different energy market models to FRAM. Different energy market models have different internal data models with their own level of detail, which may support or not support some modelling features. It would be very difficult to build generic applications (for example, workflow running the analysis and dashboard showing the results) on top of these different data models without this extra data abstraction level that the core model represents. 

Without the core model we would have to build an application that understands all possible data models and in addition all possible "hacks" that the analysts sometimes use when the model is missing support for some feature. This would not be scalable or reasonable or cost-efficient.Instead, the core model creates a stable and consistent API that different applications can use without being dependent on the various internal data models. 
