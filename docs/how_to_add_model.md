# How to add a new power market model?

*We are still working on this instruction; more detailed information will come soon. In the meantime you can contact [fram@nve.no](mailto:fram@nve.no)*

To integrate a new model into FRAM you should develop a FRAM API for the model. Use JulES solver from our [simple demo]({{ framlinks.demo }}){:target="_blank"} as an example. 

Use interfaces defined in FRAM core to define the necessary classes: Solver, Populator, etc. 

There are two ways of integration with the core model: 

- Use high-level components (thermal, demand etc) or reuse existing components. For example, if your model needs a smart house component, try to define it using existing components demand and solar power. You can also define your own components - in this case you have to update the populator so that it knows about the new components. </br>
Using high-level components may be easier in the start, but it may lead to a lot of duplicate code if components have many similar properties.

- **Recommended:** Use low-level components: flow and node. These are the most basic components in the FRAM core model, and if you spend some time in the start understanding how they work, you can write "clean" and reusable python code and avoid a lot of duplicate code. See our examples with JulES model to learn how to use flow and node.


