# This action Submits Kubeflow Pipelines to a Kubeflow cluster

The purpose of this action is to allow for automated deployments of [Kubeflow Pipelines](https://github.com/kubeflow/pipelines) on any Kubeflow cluster. The action will collect the pipeline from a python file and compile it before uploading it to Kubeflow. If using GCP, the Kubeflow deployment must be using [IAP](https://www.kubeflow.org/docs/gke/deploy/monitor-iap-setup/).

# Usage

## Example Workflow that uses this action 


To compile a pipeline and upload it to kubeflow: 

```yaml
name: Compile and Deploy Kubeflow pipeline
on: [push]

# Set environmental variables

jobs:
  build:
    runs-on: ubuntu-18.04
    steps:
    - name: checkout files in repo
      uses: actions/checkout@master


    - name: Submit Kubeflow pipeline
      id: kubeflow
      uses: NikeNano/kubeflow-github-action@master
      with:
        KUBEFLOW_URL: ${{ secrets.KUBEFLOW_URL }}  # Optional
        ENCODED_GOOGLE_APPLICATION_CREDENTIALS: ${{ secrets.GKE_KEY }}  # Optional
        GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcloud-sa.json  # Optional
        CLIENT_ID: ${{ secrets.CLIENT_ID }}  # Optional
        PIPELINE_CODE_PATH: "example_pipeline.py"
        PIPELINE_FUNCTION_NAME: "flipcoin_pipeline"
        PIPELINE_PARAMETERS_PATH: "parameters.yaml"
        EXPERIMENT_NAME: "Default"
        RUN_PIPELINE: False
        VERSION_GITHUB_SHA: False

```

If you also would like to run it use the following: 

```yaml
name: Compile, Deploy and Run on Kubeflow
on: [push]

# Set environmental variables

jobs:
  build:
    runs-on: ubuntu-18.04
    steps:
    - name: checkout files in repo
      uses: actions/checkout@master


    - name: Submit Kubeflow pipeline
      id: kubeflow
      uses: NikeNano/kubeflow-github-action@master
      with:
        KUBEFLOW_URL: ${{ secrets.KUBEFLOW_URL }}  # Optional
        ENCODED_GOOGLE_APPLICATION_CREDENTIALS: ${{ secrets.GKE_KEY }}  # Optional
        GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcloud-sa.json  # Optional
        CLIENT_ID: ${{ secrets.CLIENT_ID }}  # Optional
        PIPELINE_CODE_PATH: "example_pipeline.py"
        PIPELINE_FUNCTION_NAME: "flipcoin_pipeline"
        PIPELINE_PARAMETERS_PATH: "parameters.yaml"
        EXPERIMENT_NAME: "Default"
        RUN_PIPELINE: True
        VERSION_GITHUB_SHA: False

```
The repo also contains an example where the containers in the pipeline are versioned with the github hash in order to improve operations and tracking of errors. However this requires that the pipelines function to be wrapped in a function with one argument: 

```python 

  def pipeline(github_sha :str):
      ... 
      
```

the containers is versioned with the hash: 


```python
  pre_image = f"gcr.io/{project}/pre_image:{github_sha}"
  train_forecast_image = f"gcr.io/{project}/train_forecast_image:{github_sha}"

```
      
for example see [here](https://github.com/NikeNano/kubeflow-github-action/blob/master/forecast_peython_wiki/deployment/pipline.py)

## Mandatory inputs

1) KUBEFLOW_URL: The URL to your kubeflow deployment
2) PIPELINE_CODE_PATH: The full path to the python file containing the pipeline
3) PIPELINE_FUNCTION_NAME: The name of the pipeline function the PIPELINE_CODE_PATH file
4) PIPELINE_PARAMETERS_PATH: The pipeline parameters
5) EXPERIMENT_NAME: The name of the kubeflow experiment within which the pipeline should run
6) RUN_PIPELINE: If you like to also run the pipeline set "True"
7) VERSION_GITHUB_SHA: If the pipeline containers are versioned with the github hash

## Mandatory when using GCP

1) GKE_KEY: Service account with access to kubeflow and rights to deploy, see [here](http://amygdala.github.io/kubeflow/ml/2019/08/22/remote-deploy.html) for example, the credentials needs to be bas64 encode:
``` bash
cat path-to-key.json | base64
```
2) GOOGLE_APPLICATION_CREDENTIALS: The path to where you like to store the secrets, which needs to be decoded from GKE_KEY
3) CLIENT_ID: The IAP client secret



# Future work

Add so that pipelines can be scheduled to run as well. Soooon done! 
