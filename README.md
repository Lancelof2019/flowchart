DeepProg overall workflow
├── Input
│   ├── Multi-omics matrices
│   │   ├── RNA-Seq
│   │   ├── miRNA
│   │   └── DNA methylation
│   └── Survival data
│       ├── survival time t
│       └── event indicator e
│
├── Ensemble construction
│   ├── Build multiple submodels
│   └── Each submodel uses 80% random subset of training samples
│
├── For each submodel
│   ├── Module 1: Unsupervised subtype inference
│   │   ├── Normalization
│   │   │   ├── Select top-variance features
│   │   │   ├── Rank-based normalization
│   │   │   ├── Sample-sample Pearson correlation / distance
│   │   │   └── Rank normalization again
│   │   ├── Autoencoder transformation
│   │   │   ├── One autoencoder for each omic type
│   │   │   └── Hidden layer size = 100 (default)
│   │   ├── Survival-related latent feature selection
│   │   │   ├── Univariate Cox-PH for each hidden feature
│   │   │   └── Keep features with p < 0.01
│   │   ├── Merge selected hidden features into matrix Z
│   │   └── Gaussian mixture clustering on Z
│   │       ├── test K = 2,3,4,5
│   │       └── sort clusters by median survival
│   │
│   ├── Module 2: Supervised classifier construction
│   │   ├── Kruskal-Wallis test
│   │   │   ├── for each omic type
│   │   │   └── for each original feature
│   │   ├── Select top 50 discriminative features per omic
│   │   ├── Combine selected features into training matrix M
│   │   ├── Train SVM on M
│   │   └── Hyperparameter selection
│   │       ├── define candidate parameter grid
│   │       ├── perform 5-fold cross-validation
│   │       └── choose parameter set with minimum average test-fold error
│   │
│   └── Submodel filtering
│       ├── remove model if no survival-related hidden feature found
│       └── remove model if cluster labels are not significantly associated with survival
│
├── Prediction for a new sample
│   ├── Match common omics/features with training set
│   ├── Apply the same normalization procedure
│   ├── Use corresponding trained SVM of each valid submodel
│   ├── Obtain subtype probabilities from each submodel
│   └── Average probabilities across retained submodels
│
└── Final output
    ├── final subtype label
    └── worst-survival subtype probability
