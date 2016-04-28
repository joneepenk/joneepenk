#Overlapping Experiment Infrastructure: More, Better, Faster Experimentation

**Google, Inc.**

**[diane,agarwal,deirdre,mmm]@google.com**

 traffic. Valid but bad experiments (e.g., buggy or unintentionally producing really poor results) should be caught quickly and disabled. Standardized metrics should be easily available for all experiments so that experiment comparisons are fair: two experimenters should use the same filters to remove robot traffic [7] when calculating a metric such as CTR.




![Figure1](Overlapping-Figure-1.png) 

**Figure 1: A sample flow of a query through multiple binaries. Information (and time) flows from left to right.**













While this nesting may seem complex, it affords several advantages. First, having a non-overlapping domain allows us to run experiments that really need to change a wide swath of parameters that might not normally be used together. Next, it allows us to have different partitionings of parameters; one could imagine three domains, one non-overlapping, one overlapping with one partitioning of the parameters, and a third overlapping domain with a different parameter partitioning. Finally, the nesting allows us to more efficiently use space, depending on which partitionings are most commonly used, and which cross-layer parameter experiments are most commonly needed. Note that it is easy to move currently unused parameters from one layer to another layer, as long as one checks to make sure that the parameters can safely overlap with the parameters in the original layer assignment [^1] . Also note that to ensure that the experiments in different layers are independently diverted, for cookie-mod based experiments, instead of mod = f(cookie) % 1000, we use mod = f(cookie, layer) % 1000. While this nesting complexity does increase flexibility, there is a cost to changing the configuration, especially of domains: changing how traffic is allocated to domains changes what traffic is available to experiments. For example, if we change the non-overlapping domain from 10% of cookie mods to 15%, the additional 5% of cookie mods comes from the overlapping domain and cookies that were seeing experiments from each layer in the the overlapping domain are now seeing experiments from the non-overlapping domain.

* Launch layers are a separate partitioning of the parameters, i.e., a parameter can be in at most one launch layer and at most one “normal” layer (within a domain) simultaneously.
* In order to make this overlap of parameters between launch and normal layers work, experiments within launch layers have slightly different semantics. 

Specifically, experiments in launch layers provide an alternative default value for parameters. In other words, if no experiments in the normal experiment layers override a parameter, then in the launch layer experiment, the alternate default value specified is used and the launch layer experiment behaves just like a normal experiment. However, if an experiment in the normal experiment layer does override this parameter, then that experiment overrides the parameter’s default value, regardless of whether that value is specified as the system default value or in the launch layer experiment.
Recall that both experiments and domains operate on a segment of traffic (we call this traffic the “diverted” traffic). Diversion types and conditions are two concepts that we use in order to determine what that diverted segment of traffic is.






* Create a canary experiment (pushed via a data push) to ensure that the feature is working properly. If not, then more code may need to be written.
* Create an experiment or set of experiments (pushed via a data push) to evaluate the feature. Note that configuring experiments involve specifying the diversion type and associated diversion parameters (e.g., cookie mods), conditions, and the affected system parameters.
* Evaluate the metrics from the experiment. Depending on the results, additional iteration may be required, either by modifying or creating new experiments, or even potentially by adding new code to change the feature more fundamentally.
* If the feature is deemed launchable, go through the launch process: create a new launch layer and launch layer experiment, gradually ramp up the launch layer experiment, and then finally delete the launch layer and change the default values of the relevant parameters to the values set in the launch layer experiment.

![Figure3](Overlapping-Figure-3.png) 

**Figure 3: Logic flow for determining which domains, layers, and experiments a query request is in.**



N = (1/queries<sub>control</sub> + 1/queries<sub>experiment</sub>)^(−1)
 
 



![Figure4](Overlapping-Figure-4.png) 

**Figure 4: Slope for calculating s for coverage by diversion type.**







* Support for slicing: aggregate numbers can often be misleading, as a change may not be due to the metric actually changing (e.g., CTR changing), but may rather be due to a mix shift (e.g., more commercial queries). These Simpson’s paradoxes are important to spot and understand, as Kohavi mentions [4].

* Extensibility: it must be easy to add custom metrics and slicings. Especially for new features, the existing suite of metrics and slicings may be insufficient.

![Figure5-2](Overlapping-Figure-5-2.png) 
Figure 5: Graphs showing the trend over time for the number of experiments, people running experiments, and launches.



[^1]: Sociologically, we have observed that if layers have semantically meaningful names, e.g., the “Ad Results Layer” and the “Search Results Layer”, engineers tend to be reluctant to move flags that violate that semantic meaning. Meaningful names can help with robustness by making it more obvious when an experiment config- uration is incorrect, but it can also limit the flexibility that engineers will take advantage of.


