<img src=logo-dp-xgboost.svg width=500/> 
===========

[![XGBoost-CI](https://github.com/sarus-tech/dp-xgboost/workflows/XGBoost-CI/badge.svg?branch=master)](https://github.com/sarus-tech/dp-xgboost/actions)
<!-- [![Documentation Status](https://readthedocs.org/projects/xgboost/badge/?version=latest)](https://xgboost.readthedocs.org) -->
![PyPI](https://img.shields.io/pypi/v/dp-xgboost)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/dp-xgboost)
![Twitter Follow](https://img.shields.io/twitter/follow/Sarus_tech?style=social)

# What is Sarus DP-XGBoost?

This is a fork of [XGBoost](https://github.com/dmlc/xgboost/tree/master/) that aims at adding differential-privacy to gradient boosted trees.

A detailed explanation of the theory and methods used can be found in:
[Grislain, Nicolas and Joan Gonzalvez. “DP-XGBoost: Private Machine Learning at Scale.” (2021).](https://arxiv.org/abs/2110.12770).

# Quick Start



# Installing Sarus XGBoost


# Usage

Python examples which build a DP model are given in `sarus/python/`. 

The main parameters involved in DP learning are: 
- `tree_method` which must be set to `approxDP` to use Sarus XGBoost DP tree learning. 
- `dp_epsilon_per_tree`: the privacy budget of a single tree.
- `min_child_weight`: the minimum weight needed to construct a leaf, this influences the DP noise.
- `subsample`: the fraction of the dataset randomly sampled to each tree, subsampling improve the privacy.
- `num_boost_rounds`: the number of trees built. 

The privacy queries used during training are stored in the model and accessible via 
`booster.save_model()`. 

# Privacy consumption 

Note that the total privacy consumption of the boosted trees is given by:

<img src="https://render.githubusercontent.com/render/math?math=n \log{ \left( 1 %2B \gamma(e^{\epsilon} - 1) \right) }">

<!-- $ n \log{ \left( 1 + \gamma(e^{\epsilon} - 1) \right) } $ -->

Where <img src="https://render.githubusercontent.com/render/math?math=n"> is the number of trees, <img src="https://render.githubusercontent.com/render/math?math=\gamma"> the subsample fraction (between 0 and 1), and <img src="https://render.githubusercontent.com/render/math?math=\epsilon"> 
is the budget per tree. You can refer to our explaining article in `doc/sarus` for more details on privacy consumption. 

# Differential Privacy in the C++ library

DP is added at three levels in the XGBoost C++ shared library (under the `src` repo): to construct sketches (with a histogram query), for split selection (with an exponential mech), and for leaf values (with a Laplace mechanism). The mechanisms are located in
`include/xgboost/mechanisms.h`. 

Relevant classes are in the `src/tree/updater_histmaker.cc` file and especially the `DPHistMaker` class which is the DP tree updater called when setting `approxDP` as `tree_method` param in XGBoost. 

# Building for the JVM 

To use with Spark, please follow https://xgboost.readthedocs.io/en/latest/jvm/xgboost4j_spark_tutorial.html. 

- Needed: Java JDK 1.8, Spark 2.12, Maven 3
- Set the JAVA_HOME env variable first:
`export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home/` 
- In the `jvm-packages` folder run `mvn package install -DskipTests -Dmaven.test.skip=true`

This should build the jars `xgboost4j` and `xgboost4j-spark` which will then be passed to
`spark-submit`. The `sarus/spark` folder contains an example of Spark project in Scala with a POM file that should compile and launch Sarus XGBoost with 2 workers.

# Developper guide

Don't forget to `git submodule init` and `git submodule update` after cloning this repo to initialize the XGBoost submodules (e.g. dmlc). 

Build tools needed: `cmake`, `g++` and `libomp` which can be installed with `brew` or your favourite package manager. The following commands will setup the build config:

```bash
mkdir build
cd build
cmake ..
```
