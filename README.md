# 🧠 Network Analysis of HCP Dataset

This repository contains a network analysis of the Human Connectome Project (HCP) dataset. This facilitates the investigation of relationships between variables of interest.

---

## ⚙️ Setup

Install required R packages:
```r
    library(tidyverse)
    library(ggcorrplot)
    library(Hmisc)
    library(reshape2)
    library(qgraph)
    library(igraph)
    library(networktools)
    library(bootnet)
    library(bnlearn)
    library(Rgraphviz)
```
---

## 📂 Data

Merge the two input files of HCP:
```r
    unrestr <- read_csv('Unrestricted.csv', show_col_types = FALSE)
    restr <- read_csv('Restricted_new.csv', show_col_types = FALSE) 
    data_raw <- inner_join(restr, unrestr, by = 'Subject')
```
---

## 🧹 Preprocessing

- Remove monozygotic twins  
- Select relevant variables 
- Encode gender (F = 1, M = 0)  
- Drop missing values  

---

## 📊 Analysis

### Correlation

- Compute correlation matrix 
- Visualize with `ggcorrplot`
```r
    corr <- rcorr(as.matrix(data))
```
---

### Network (Graphical LASSO)
```r
    network <- qgraph::EBICglasso(cor_auto(data), n = nrow(data))
    qgraph(network, layout = "spring")
```
---

### Bootstrapping

- Edge confidence intervals  
- Edge difference tests  
```r
    network <- bootnet::estimateNetwork(data, default = "EBICglasso", corMethod = "cor_auto")
    network_bootstrapped <- bootnet(network, nBoots = 10000, nCores = 10)
```
---

### Centrality

- Strength, closeness, betweenness
- centrality stability
```r
centralityPlot(network,
               scale = "relative")
glassoBoot <- bootnet(network, nBoots = 2000, type = "person", nCores = 6,
                       statistics = c("strength", "closeness", "betweenness"))  
```
---

## 🔗 Bayesian Network

- Structure learning using hill-climbing (`hc`)
- Bootstrapped network (1000 samples)
- Averaged network with thresholding
```r
    fitBN <- hc(data)
    bootnet <- boot.strength(data, R = 1000, algorithm = "hc")
    avgnet <- averaged.network(bootnet, threshold = 0.85)
```
---

## 📁 Outputs

- Correlation plots  
- Network graphs (LASSO)  
- Bootstrapped edge/centrality results
- Bayesian causal network

---

## 📚 References
- Epskamp et al. (2018) — Network psychometrics
- Sachs et al. (2005) — Bayesian networks in biology
- Scurati & Nagarajan (2013) — Threshold selection in BN 
