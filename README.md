# Clone repo and install dvc

```bash
uv venv
. .venv/bin/activate   # activate the venv

git clone https://github.com/miguel-montes-lorenzo/dvc-data-pipeline.git

cd dvc-data-pipeline

# work in a feature branch (safer than pushing directly to main)
git switch -c mm/setup-dvc
```

# Turn the git repo into DVC repo

```bash
dvc init

git status

# only track DVC metadata files
git add .dvc .gitignore

git commit -m "initialized dvc"
```

# Download the data for example code

```bash
mkdir -p data

dvc get https://github.com/iterative/dataset-registry \
          get-started/data.xml -o data/data.xml
```

# Track the data

```bash
dvc add data/data.xml

git add data/data.xml.dvc
git commit -m "Track raw dataset with DVC (data.xml)"
```

# Install Packages from sample code

```bash
pip install -r src/requirements.txt
```

# Pipeline Stages

Using `dvc stage add` to create stages.

Stages allow connecting code to its corresponding data input and output.

```bash
dvc stage add -n prepare \
                -p prepare.seed,prepare.split \
                -d src/prepare.py -d data/data.xml \
                -o data/prepared \
                python src/prepare.py data/data.xml
```

A `dvc.yaml` file is generated. It includes information about the command we
want to run (python src/prepare.py data/data.xml), its dependencies, and outputs.

DVC uses the pipeline definition to automatically track the data used
and produced by any stage, so there's no need to manually run `dvc add`
for `data/prepared`!

Commit the pipeline definition:

```bash
git add dvc.yaml
git commit -m "Add DVC pipeline stage: prepare"
```

Similarly, we add the stage for featurizer and training. We define
outputs of a stage as dependencies of another, we can describe a sequence of
dependent commands which gets to some desired result. This is what we call a
dependency graph which forms a full cohesive pipeline.

## Featurize

```bash
dvc stage add -n featurize \
                -p featurize.max_features,featurize.ngrams \
                -d src/featurization.py -d data/prepared \
                -o data/features \
                python src/featurization.py data/prepared data/features

git add dvc.yaml
git commit -m "Add DVC pipeline stage: featurize"
```

## Train

```bash
dvc stage add -n train \
                -p train.seed,train.n_est,train.min_split \
                -d src/train.py -d data/features \
                -o model.pkl \
                python src/train.py data/features model.pkl

git add dvc.yaml
git commit -m "Add DVC pipeline stage: train"
```

## Running the pipeline

It uses dvc.yaml to easily reproduce the pipeline.

```bash
dvc repro
```

It will create a `dvc.lock` (a "state file") to capture the reproduction's results.

Commit it immediately to record the current state & results:

```bash
git add dvc.lock
git commit -m "first pipeline repro"
```

## Configure a DVC remote

Before pushing data, configure a default remote (example: local folder for testing):

```bash
mkdir -p ../dvc-remote
dvc remote add -d localremote ../dvc-remote

git add .dvc/config
git commit -m "Configure default DVC remote (local folder)"
```

(You can later switch to SSH/S3/GDrive if needed.)

## Push pipeline outputs to remote

```bash
dvc push
```

## Push commits to GitHub

Always sync safely with rebase before pushing:

```bash
git fetch origin
git pull --rebase origin main
git push origin mm/setup-dvc   # or main if pushing directly
```

## Visualizing the pipeline

```bash
dvc dag
```
