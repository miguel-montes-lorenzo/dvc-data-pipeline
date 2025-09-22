 Turn the git repo into DVC repo

```
dvc init

git status

git add .

git commit -m "initialized dvc"
```
**Download the data for example code**

```
dvc get https://github.com/iterative/dataset-registry \
          get-started/data.xml -o data/data.xml
```

**Track the data**

```
dvc add data/data.xml
```

**Install Packages from sample code**

```
pip install -r src/requirements.txt
```


## Pipeline Stages

Using `dvc stage add` to create stages.

Stages allow connecting code to its corresponding data input and output.

```
dvc stage add -n prepare \
                -p prepare.seed,prepare.split \
                -d src/prepare.py -d data/data.xml \
                -o data/prepared \
                python src/prepare.py data/data.xml
```

A dvc.yaml file is generated. It includes information about the command we \
want to run (python src/prepare.py data/data.xml), its dependencies, and outputs.

DVC uses the pipeline definition to automatically track the data used \
and produced by any stage, so there's no need to manually run dvc add \
for data/prepared!

Similarly, we add the stage for featurizer and training. We define \
outputs of a stage as dependencies of another, we can describe a sequence of \
dependent commands which gets to some desired result. This is what we call a \
dependency graph which forms a full cohesive pipeline.

### Featurize

```
dvc stage add -n featurize \
                -p featurize.max_features,featurize.ngrams \
                -d src/featurization.py -d data/prepared \
                -o data/features \
                python src/featurization.py data/prepared data/features

```

### Train

```
dvc stage add -n train \
                -p train.seed,train.n_est,train.min_split \
                -d src/train.py -d data/features \
                -o model.pkl \
                python src/train.py data/features model.pkl
```

### Running the pipeline

It uses dvc.yaml to easily reproduce the pipeline.

```
dvc repro
```

It will create a `dvc.lock` (a "state file") was created to capture the reproduction's results.

Next, we immediately commit dvc.lock to Git after its creation or modification, to record the current state & results: 

```
git add dvc.lock && git commit -m "first pipeline repro"

git push origin main
```

Visualizing the pipeline

```
dvc dag
```
