+++
date = '2025-07-12T12:44:15+02:00'
draft = false
title = 'A Project Report on MIP Parameter Tuning for Gurobi'
summary= 'This is a report on my project MIP Parameter Tuning for Gurobi'
math=true
showToc= true

tags = ["Tuning", "Gurobi", "Mixed Integer Programming"]
categories = ["Optimization", "ML", "Research"]
+++

This post documents a small personal project I recently completed, where I explored the idea of using machine learning to assist Gurobi parameter tuning. It covers the motivation behind the idea, how I set up the experiment, what worked (and didn’t), and what I learned along the way.

## 1. Introduction: Why Should We Use Machine Learning to Tune Gurobi?

Gurobi is widely regarded as one of the most powerful commercial solvers for mixed-integer programming (MIP). It is fast, robust, and highly configurable. But this configurability comes at a cost: Gurobi exposes over **100 parameters** that users can adjust, ranging from cutting plane aggressiveness (`Cuts`), heuristics intensity (`Heuristics`), presolve behavior (`Presolve`), to branching strategies (`VarBranch`) and solving methods (`Method`, `Threads`).

While the **default settings** work reasonably well for many problems, in practice, the solver’s performance—both in terms of runtime and solution quality—can vary **dramatically** depending on these parameters. A configuration that works well for one problem instance might perform terribly on another. This makes **parameter tuning** both a critical and notoriously difficult task, especially in industrial applications where solving large-scale MIP problems quickly and reliably is essential.

Gurobi does provide tools like the [**Tuning Tool**](https://docs.gurobi.com/projects/optimizer/en/current/features/tuning.html), which can help users search for good parameter combinations on a given instance. However, this tool still requires time-consuming trial-and-error on a per-instance basis. For users dealing with **hundreds or thousands of MIP instances**, this is far from ideal.

This raises an important question:

> **Can we use machine learning to *automatically* learn good parameter configurations based on the features of a MIP problem?**

In this project, I explore that idea. By combining Gurobi, benchmark MIP datasets, and learning models like **MLP regression** and tree models like **Random Forest** and **XGBoost**, I attempt to build a prototype system that can predict which parameter settings are likely to perform well for a given instance. The result is an early step toward what I call a *"data-driven parameter assistant"*—a tool that learns from previous optimization runs and helps users get better performance *without manual tuning*.

In the rest of this post, I’ll walk through the core ideas, modeling setup, challenges I faced, and some promising results and directions for future work.

## 2. Project Goal and Concept

The primary goal of this project is to build a prototype system that can **automatically predict which Gurobi parameter configurations are likely to perform well** on a given MIP instance. Rather than manually testing different parameter settings through trial and error, the system attempts to learn a mapping from problem features to solver performance, using data from previously solved instances.

To make the problem tractable, I formulate it as a **regression task**. Each instance is characterized by a set of features (e.g., number of variables, constraints, sparsity ratios), and the model learns to predict **solver performance metrics**—such as runtime or optimality gap—for different parameter configurations. This setup allows the model to generalize and estimate the quality of a configuration even for unseen problems.

Ultimately, the vision is to create a lightweight, intelligent “**parameter assistant**” that could be integrated into a solver workflow. Given a new problem, the assistant would recommend one or more promising configurations, saving time and potentially improving solution quality—especially in large-scale or time-constrained industrial settings.

## 3. Dataset Design and Modeling Choices

### 3.1 Why MIPLIB?

To train a model that can generalize to real-world MIP problems, I needed a reliable and diverse dataset. For this purpose, I chose [**MIPLIB**](https://miplib.zib.de/), a widely used benchmark collection in both academic and industrial research. MIPLIB contains a curated set of **real-world and synthetic instances** that span a broad range of sizes, complexities, and domains (e.g., scheduling, network design, facility location).

In addition to providing high-quality instance files in standard formats (`.mps`, `.lp`), MIPLIB also includes reference solutions via `solu.txt`, which allows us to compute the **optimality gap**—a key performance metric—when evaluating solver outputs. This makes MIPLIB not only a solid foundation for empirical testing but also a natural fit for supervised learning.

### 3.2 Parameter Configurations

To simplify the tuning landscape, I defined a **fixed set of 27 parameter configurations** and a **baseline parameter configuration**, each corresponding to a meaningful variation of key Gurobi parameters. These include:

- `Cuts`: controls cut generation level. Use value 0 to shut off cuts, 1 for moderate cut generation, 2 for aggressive cut generation, and 3 for very aggressive cut generation (ignored in the project). The default -1 value chooses automatically. 
- `Presolve`: toggles presolving strategies. A value of -1 corresponds to an automatic setting. Other options are off (0), conservative (1), or aggressive (2) (ignored in the project). More aggressive application of presolve takes more time, but can sometimes lead to a significantly tighter model.  
- `MIPFocus`: allows you to modify your high-level solution strategy, depending on your goals. By default, the Gurobi MIP solver strikes a balance between finding new feasible solutions and proving that the current solution is optimal. If you are more interested in good quality feasible solutions, you can select `MIPFocus=1`. If you believe the solver is having no trouble finding the optimal solution, and wish to focus more attention on proving optimality, select `MIPFocus=2`. It also has a level of `MIPFocus=3` to choose when the best objective bound is moving very slowly in the extremal case. For the simificity of this project I ignored this case.

Notice that Gurobi offers a large number of parameters that can be tuned to influence solver behavior. You can find the full list on the [reference page](https://docs.gurobi.com/projects/optimizer/en/current/reference/parameters.html). For this project, I selected a subset of parameters as tuning targets based on suggestions from the [guidelines page](https://docs.gurobi.com/projects/optimizer/en/current/concepts/parameters/guidelines.html).

Ideally, I would include all of the key parameters mentioned in the guidelines to build a more complete tuning space. However, doing so would lead to an exponential explosion in the number of parameter combinations. To keep the experiment tractable, I limited the tuning space to just three parameters: `MIPFocus`, `Presolve`, and `Cuts`.

While this simplification made the data generation process more manageable, I also acknowledge that it might be one of the reasons the models struggled to learn meaningful patterns during training.

Each MIPLIB instance was solved using **all 27 configurations** and also on **baseline parameter configuration**, and the following solver outputs were recorded for each run:

- **Runtime** (in seconds)
- **Optimality gap** (if applicable)
- **Solver status** (e.g., optimal, time limit, infeasible)

This structured approach enabled me to create a consistent dataset of instance–configuration–performance triples, which serve as the training data for the machine learning models.

### 3.3 Why Regression Instead of Classification?

At first glance, parameter tuning might seem like a classification problem: just pick the "best" configuration from a finite list. However, in practice, **multiple configurations often yield similar performance**, and “best” can be ambiguous depending on which metric you prioritize (runtime vs. gap vs. robustness).

Instead, I chose to frame this as a **regression task**, where the model learns to **predict the performance metrics** (e.g., runtime, gap) for a given instance–configuration pair. This has several advantages:

- It enables **fine-grained performance estimation**, not just binary winner-takes-all classification.
- It supports **flexible ranking and trade-off decisions**—for example, preferring slightly slower but more reliable settings.
- It allows us to **score all 27 configurations** for a new instance and choose the top-N candidates, rather than committing to one.

Clearly, if I were to frame this project as a classification task—where the goal is to predict the best parameter configuration out of 27 options—then MIPLIB alone would not be sufficient. With only around 230 instances, the dataset is simply too small to train a reliable multi-class classifier.

However, finding a much larger and well-structured collection of MIP instances is quite challenging. Moreover, solving each instance with all 27 parameter configurations is computationally expensive and time-consuming. These practical limitations are the main reasons why I chose a regression-based approach instead.

Overall, this regression-based formulation aligns better with the real-world nature of tuning and gives us more control in post-processing the model's predictions.

### 3.4 Speeding Up Dataset Generation with Parallelism

Although MIPLIB is a well-established benchmark, it contains around **230 instances**, and solving each instance across all **27 parameter configurations + 1 baseline configuration** is extremely time-consuming. To collect performance data efficiently, I implemented a **parallel dataset generation pipeline** in `generate_dataset.py`. Solving hundreds of MIP instances across **27 parameter configurations + 1 baseline configuration** can otherwise take days on a single CPU. To address this, I used Python’s `multiprocessing` module to **distribute the work across multiple cores**:

```python
with Pool() as pool:
    jobs = []
    for inst in instances:
        for config in configs:
            jobs.append(pool.apply_async(run_gurobi, args=(inst, config)))
    results = [job.get() for job in jobs]
```

**Key Benefits**:

- Full CPU utilization: all available cores are used simultaneously

- Parallel over instance–config pairs: ideal granularity for independent solver runs

- Resilient execution: failed runs are logged and skipped without interrupting the pipeline

This parallel execution framework reduced dataset generation time from potentially days to just a few hours, making experimentation much more scalable.

For future scalability, the pipeline could be extended to support cluster or cloud-based parallelism using tools like `joblib`, `Ray`, or `Dask`.


## 4. ML Models and Initial Results

### 4.1 Models Used

To explore how different machine learning methods perform on this parameter tuning task, I implemented and evaluated three types of regression models:

- **Multi-layer Perceptron (MLP)**  
  A simple feedforward neural network, used as a baseline. MLPs are capable of learning non-linear mappings between input features and performance metrics, though they can be sensitive to data size and normalization.

- **Random Forest Regressor**  
  An ensemble method that combines multiple decision trees to reduce overfitting and improve robustness. It’s well-suited for tabular data and can capture non-linear feature interactions effectively.

- **XGBoost Regressor**  
  A state-of-the-art gradient boosting framework known for its high performance and ability to handle sparse features, missing values, and complex feature dependencies.

All three models were trained using the same instance–parameter–performance dataset, with cross-validation to assess generalization.

---

### 4.2 Evaluation Metrics

To evaluate model performance, I used the following standard regression metrics:

- **Mean Squared Error (MSE)**: measures average squared difference between predicted and actual runtime/gap  
- **R² Score**: indicates how much variance in the target variable is explained by the model  
- **Residual Analysis**: to understand the distribution and consistency of prediction errors across configurations

These metrics were computed across all parameter configurations for the held-out test instances.

---

### 4.3 Initial Findings

Here are the main observations from the experiments:

- All three models—**MLP**, **Random Forest**, and **XGBoost**—achieved **comparable performance** in terms of both MSE and R². There was no clear winner.
- **Tree-based models** (Random Forest and XGBoost) demonstrated slightly better **stability across different runs** and were less sensitive to data scaling.
- **XGBoost** offered additional interpretability via feature importance scores, which revealed that a few structural features (like number of variables or constraint sparsity) played a major role in runtime prediction.
- Some parameter configurations consistently performed better across many instances, but **no single configuration dominated universally**—further justifying the need for instance-dependent tuning.

Overall, these results suggest that while the choice of regression model matters to some extent, **the current limitations in dataset size and feature richness** are likely the main bottlenecks. More informative features and a larger, more diverse dataset may have a greater impact on predictive performance than model complexity alone.


## 5. Issues Faced and Observations

While the initial results showed that machine learning models can reasonably predict solver performance, several challenges emerged during the project. These challenges are mostly related to the scale, structure, and nature of the dataset.

### 5.1 Data Sparsity

Although MIPLIB is a well-established benchmark, it contains only around **230 instances**, and solving each instance across all **27 parameter configurations + 1 baseline configuration** is extremely time-consuming. As a result, the dataset remains relatively small (about 6500 records), especially when framed as a regression task over hundreds of input–configuration pairs.

This limited data volume constrains the model’s ability to generalize and increases the risk of overfitting, particularly for models like MLP that require more data to learn stable patterns.

### 5.2 Runtime Skew and Outliers

Solver runtime varies widely across different parameter settings. Some instance–configuration pairs solve in under a second, while others hit time limits or produce very poor optimality gaps. This leads to **highly skewed runtime distributions**, with many long-tail outliers that can dominate loss functions like MSE.

In many cases, it was unclear whether a slow runtime was due to the parameter setting being truly suboptimal, or just because of solver randomness or system noise—making some data points difficult to interpret or learn from.

### 5.3 Feature Limitations

In the current version of the project, all instance features are directly taken from the benchmark set’s provided `.csv` file—specifically from the [Benchmark Set CSV](https://github.com/willweiao/MIP-Parameter-Tuning-with-Gurobi/blob/main/data/reference/The_Benchmark_Set.csv) that accompanies MIPLIB. These features include basic **structural metrics** such as:

- Status (easy, hard)
- Number of variables
- Number of binaray variables
- Number of integer variables
- Number of continuous variables
- Number of nonzero coefficients
- Objective

### 5.4 Model Stability and Underfitting

During training, the **MLP model consistently showed clear signs of underfitting**. Both training and validation loss remained high, and the model failed to capture meaningful trends in the target variables (runtime or gap). Even after increasing the number of epochs and tuning hyperparameters (e.g., hidden layers, activation functions, dropout), the loss quickly plateaued—indicating that the model had already reached its capacity, likely due to limited input feature expressiveness.

This underperformance is not surprising, given that the current feature set—drawn directly from MIPLIB’s benchmark `.csv`—consists only of basic structural properties such as variable counts and constraint sparsity. These features are informative to some extent but lack the complexity and signal richness that neural networks typically require to generalize well.

Interestingly, **XGBoost showed very similar performance**. Although slightly more stable in terms of training behavior, it did not produce significantly better predictive accuracy. Both models exhibit underfitting, which strongly suggests that the bottleneck lies not in the model architecture, but in the **limited size and expressiveness of the dataset**. It achieved lower mean squared error and similar R² scores on test sets, still with signs of underfitting.

---

These findings indicate that **neither model has reached its full potential**, and model choice is not the dominant factor at this stage. The key limiting factor is the **quality and diversity of the input data**. With richer, semantically meaningful, or solver-derived features, and a larger, more representative set of MIP instances, both MLP and tree-based models like XGBoost are likely to perform significantly better.

The good news is that the training process—especially for MLP—shows healthy convergence behavior. This means that the pipeline I implemented (training loop, data generation, evaluation) is structurally sound and ready to scale. With a more robust and well-structured dataset, it would be much more promising to train a model that can truly learn to recommend parameter configurations effectively.


## 6. Future Work

This project demonstrates that while it is feasible to apply machine learning to predict solver performance for different Gurobi parameter configurations, there are clear limitations stemming from the current dataset and feature design. The following directions represent promising paths for future improvements:

### 6.1 Expand and Diversify the Dataset

One of the most critical bottlenecks is the **small size and narrow scope of MIPLIB**. With only ~230 instances, the model has limited exposure to the diversity of real-world problems. To improve generalization:

- Incorporate additional instances from **NEOS**, **real-world industrial cases**, or **synthetic generators**
- Include **instances from different domains** (e.g., scheduling, logistics, routing, energy systems)
- Generate more training samples by varying solver time limits or adding noisy variants

Larger and more varied data will allow the model to distinguish which parameters work better in which structural or semantic contexts.

### 6.2 Engineer Richer Features

Currently, features are limited to structural statistics from the benchmark `.csv` file. To improve predictive power, future versions could integrate:

- **Dynamic solver features**: LP relaxation gap, presolve reductions, node counts, number of cuts applied
- **Solver log parsing**: extract meaningful intermediate statistics from `.log` files
- **Semantic tags**: even coarse-grained instance type labels (e.g., "knapsack", "set cover") could enhance model understanding
- **Graph-based features**: modeling the constraint matrix as a bipartite or hypergraph

These enhancements would help models move beyond size-based heuristics toward truly learning solver behavior patterns.

### 6.3 Improve Label Quality and Target Definitions

Currently, the labels are based on **runtime and optimality gap**, which are noisy and highly skewed across configurations. Future work may explore:

- **Ranking-style learning**: train the model to compare pairs of configurations per instance rather than regressing absolute runtime
- **Multi-objective scoring**: account for both speed and gap simultaneously
- **Classification with confidence**: predict a shortlist of promising configurations with uncertainty estimates

This may help reduce variance and yield more stable recommendations.

### 6.4 Deployment as a Tool

The final vision is to provide an automated, intelligent assistant that recommends parameter settings based on a problem’s features. Next steps toward deployment could include:

- Building a **CLI tool** or **Python package** that takes `.lp`/`.mps` input and outputs ranked parameter sets
- Integrating the system into a solver workflow (e.g., Gurobi callback or tuning interface)
- Providing optional fallback to Gurobi's native tuning if model confidence is low

Such a system could significantly accelerate practical optimization workflows in both academic and industrial environments.

---

Overall, this project sets up a solid foundation for learning-based solver tuning. While current results are limited by data, the methodology is extensible—and with more informative features and better labels, there is strong potential to build a robust, generalizable model for Gurobi parameter recommendation.


## 7. Conclusion

This project set out to explore whether machine learning can assist in tuning Gurobi parameters by predicting solver performance based on problem features. Through constructing a structured dataset of MIP instances and evaluating multiple ML models—including MLP, Random Forest, and XGBoost—I built a prototype pipeline capable of learning performance patterns across parameter configurations.

Although the predictive performance was limited by data quality and feature richness, the training behavior of both neural and tree-based models suggests that the methodology is sound and extensible. Notably:

- The MLP model underfit due to insufficient feature expressiveness, not optimization issues  
- Tree-based models like XGBoost performed slightly better, but still suffered from the same data limitations  
- The current pipeline (data generation, training, evaluation) is modular, parallelized, and ready for scaling

One of the most valuable takeaways is that **data is the main bottleneck, not model capacity**. This insight shifts the focus toward collecting more diverse MIP instances and engineering better features—including solver-derived or semantic-level inputs.

Ultimately, this work demonstrates a viable direction for data-driven solver configuration. With richer datasets and continued iteration, it's entirely plausible to train a model that can recommend high-quality Gurobi parameter settings automatically—reducing manual effort and improving solver efficiency in real-world workflows.

---

If you're working on similar problems or have suggestions for datasets, features, or solver tuning strategies, feel free to reach out or open an issue on [GitHub](https://github.com/willweiao/MIP-Parameter-Tuning-with-Gurobi).

Thanks for reading my report. This article was written with the help of AI (ChatGPT) for content refinement and structure suggestions. All experiments and code are original, which you can check under the Github link above. My deepest impression after finishing all this? I would say there are two things I would keep in mind. First, this project taught me a lot about optimization—and even more about waiting for solvers to finish when generating such a dataset, but that’s life in OR. Second, If this project were an objective function, I'd say it's not even locally optimal—not say globally. Optimization is such a long way and you must count on every possibillities. Stay tuned.