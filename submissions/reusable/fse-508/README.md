# Replication package for "Search-Based Adversarial Testing and Improvement of Constrained Credit Scoring Systems", accepted at FSE 2020

This `README` file explains the structure of the package and gives basic guidelines on how to re-execute the experiments. For more detailed instructions on how to install and execute the scripts, please, refer to `INSTALL.md`.

For more information, contact the first author by e-mail: Salah Ghamizi \<salah.ghamizi@uni.lu\>

---
## About
We present CoEva2, a multi-objective search technique to generate adversarial attacks against real-life systems. It uses domain specific constraints and domain specific objectives to craft the attacks. The paper tackles an industrial system and dataset under NDA that we cannot disclose (related to overdraft and credit scoring). In accordance with the Artifact Chair, we are providing a replication study on a public dataset called *Lending Club Loan data*. It shows both how to implement our approach on available datasets and that our results are valid for other contexts. 
See `INSTALL.md` for installation instructions.
## Folder structure:
* ./data: where the study dataset is located.
* ./out: where the experiments results are located, including the random forest model.
* ./experiments: indivdual scripts that cover the research questions of the paper and their visualizations.
* ./configurations: the configuration files (*json*) to customize the experiments without coding.
* ./src/coeva2: the actual implementation of our approach.
## Setup the dataset
Our experiments involve the Lending Club Loan Dataset. You can download the processed version [here](). You can have mon information on the dataset [here](https://www.kaggle.com/wendykan/lending-club-loan-data)
For the FSE review, the dataset is already provided in the folder *./data*
## Setup the model
For the FSE review, we provide a trained Random Forest model in the folder *./out/target_model* You can use the train python script to train your own model.
```shell
python ./train.py
```  
## Experiments
You can either run all the attack experiments or only the analysis.
The available experiments are:
* random: Run a random search (RQ1)
* papernot: Use an iterative papernot attack (RQ1)
* f1f2f3f4: Run a multiobjective search with the 4 objectives f1, f2, f3, f4 (RQ2)
* f1f2f4: Run a multiobjective search with the 3 objectives f1, f2, f3, f4 (RQ2)
* f1f3f4: Run a multiobjective search with the 3 objectives f1, f2, f3, f4 (RQ2)
* retrain: Adversarial training with the f1f2f3f4 adversarial (RQ3)

```shell
# Running the analysis with pre-run experiments
python ./experiments_results.py


# Running the whole experiments and display their results 
# The optional parameter -x allow you to specify only one experiment 
python ./experiments.py [-x random|f1f2f3f4|f1f3f4|f1f2f4|papernot|retrain]
```  

## Customization

### Custom configuration
You can extend or customize the search using the custom.py file 

```shell
# The required parameter -c allows you to specify the json configuration file to use 
# The optional parameter -i allow you to specify a unique ID for the experiment run. 
# The output files will be located in a folder associaed with this ID. 
# If not provided, the current timestamp will be used 
python ./custom.py -c CONFIG_FILE [-i RUN_ID]
``` 

Configuration files are located in the folder **./configurations**:

* *experiment_path*: Will be the folder where the model to attack is located (can be generated by ./train.py) and where the output of the experiments will be stored
* *dataset_path*: The relative path (to custom.py) to the studied dataset
* *dataset_features*: The list of the features of the model, tells which features are mutable, and includes the minimum and maximum values allowed for each feature.
* *model_file* and *scaler_file*: The name of the model nd the scaler inside experiment_path
* *model_threshold*: The model classification threshold. For Lending Club, the threshold of 0.24 allowed the best performances (Recall, Precision, MCC) when training/testing the model
* *initial_states*: The set of initial states that will be attacked. Generated for instance from the script *train.py*
* *training_parameters*: The training parameters is using the script *train.py*
* *ga_parameters*: The Genetic Algorithm parameters like number of generations and population size
 + *objectives_weight*: The weight of the different objectives. F1, F2, F3, R and F4. F1-F3 refer respectively to the misclassification objective, the minimal perturbation objective and the maximal overdraft amount objective. R refers to a random objective (if set to 1 and the weights F1-F3 are set to 0 the search will be random). F4 refers to the constraint validation. We can restrict to a 2 objective search if the weight of F3 = 0 for instance.
 + *gene_types* defines the size of the vector for each individual of the Genetic Algorithm and the type of mutations allowed (integer of real).
* *papernot_parameters*: Used for the iterative papernot attack, defines how many tree of the attack should we iterate over and how many random iterations each.

```json
{
    
    "experiment_path":"./out/target_model",
    "dataset_path":"./data/lcld/lcld_venus.csv",
    "dataset_features":"./data/lcld/lcld_venus_dtypes.csv",
    "model_file":"model.joblib",
    "model_threshold":0.24,
    "scaler_file":"scaler.pickle",
    "initial_states":"attack_candidates.npy",
    "training_parameters":{
        "n_estimators": 125,
        "min_samples_split": 6,
        "min_samples_leaf": 2,
        "max_depth": 10,
        "bootstrap": true
    },
    
    "ga_parameters":{
        "n_generation": 10000,
        "n_offsprings": 50,
        "pop_size": 100,
        "objectives_weight":[1,1,1,0,1],
        "gene_types": [
            "real",
            "int",
            "real",
            "real",
            "real",
            "real",
            "real",
            "real",
            "real",
            "real",
            "real",
            "int",
            "real",
            "real",
            "int"
        ]
    },
    
    "papernot_parameters":{
        "nb_estimators":10, 
        "nb_iterations":10
    }

}
``` 

### Custom constraints

In addition to search parameters, you can define your own domain specific constraints:

```shell
python ./custom.py -c CONFIG_FILE -n NB_CONSTRAINTS -p PATH_CONSTRAINTS_FILE [-i RUN_ID]
``` 

You need to provide a path to the constraint file (plain text) and the number of constraints of the problem.
An example of custom constraint file is provided in *./data/custom_constraints.txt*. Your constraints script has access to the local property **x_ml** which is a numpy array of the current population (population size * state size  - nb features fed in the model). You constraint script should define a **constraint** variable, a list of all the constraints of your problem. You can use any method from the Numpy API within your constraint script. 