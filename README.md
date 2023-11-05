
# Actions Runner Controller (ARC)
Actions Runner Controller (ARC) solution used to run self hosted environments on Kubernetes(K8s) cluster.
For more detailed information on ARC, please refer to the official documentation [here](https://github.com/actions-runner-controller/actions-runner-controller#readme) and [here](https://github.com/actions-runner-controller/actions-runner-controller/blob/master/docs/Actions-Runner-Controller-Overview.md).


# ARC components that we use for OnePlatform runners:

## Authorisation
  The authorisation of the runners is completed using [Github App](https://github.com/actions/actions-runner-controller/blob/master/docs/using-arc-across-organizations.md). This provides [increased API quota](https://docs.github.com/en/developers/apps/building-github-apps/rate-limits-for-github-apps) for ARC to authenticate with GitHub API.

## Deployment
  [RunnerDeployments](https://github.com/actions-runner-controller/actions-runner-controller/blob/master/docs/detailed-docs.md#runnerdeployments) is the type of ARC usage options we have chosen for this setup.  
  The GitHub Runners have a container image configured from a customized Docker image: [Dockerfile](https://github.vodafone.com/VFGroup-MyVodafone-OnePlatform/DevOps-Resources/blob/master/Docker-Resources/ARC-Android-Runner/Dockerfile)


## Scaling
  The runners are dynamically scaled using [WebHooks Driven Scaling](https://github.com/actions/actions-runner-controller/blob/master/docs/automatically-scaling-runners.md#webhook-driven-scaling).

### Getting Started with Webhook Driven runners

1. Create a new Github app using [the following guide](https://github.com/actions/actions-runner-controller/blob/master/docs/authenticating-to-the-github-api.md),
in the webhook URL field add the Ingress URL of the target GitHub application.

2. Create the secret as mentioned in the guide:
    ```bash
    export APP_ID=1075
    export INSTALLATION_ID=1390
    ```
    ```bash
     kubectl create secret generic controller-manager-webhook \
        -n actions-runner-system-webhook \
        --from-literal=github_app_id=${APP_ID} \
        --from-literal=github_app_installation_id=${INSTALLATION_ID} \
        --from-file=github_app_private_key=${PRIVATE_KEY_FILE_PATH}
    ```
3. Apply your helm chart with the new values mentioned in the values.yml file:
    ```bash
    helm repo add actions-runner-controller-webhook https://actions-runner-controller.github.io/actions-runner-controller
    helm upgrade --install --namespace actions-runner-system-webhook \
                 -f values.yml \
                 --wait actions-runner-controller-webhook actions-runner-controller-webhook/actions-runner-controller
    ```
4. Apply your deployment:
    ```bash
    kubectl apply -f runnerdeployment-generic.yaml
    ```
- Not necessary, works with just adding the github token in the controller-manager-webhook secret with value github_token as a key and its value is the GHE token generated 
5. Generate a random complex string with `pwgen cli`. Combine complex strings together to form a single one. Then configure GitHub app > General > webhook secret > Update value of that secret. As well the `github_webhook_secret_token` to be the same value:
    ```bash
    brew install pwgen
    pwgen
    ```

-----
#### OnePlatform ARC Configuration Diagram
![OnePlatform_ARC_Diagram](OnePlatform_ARC_Config_Diagram.png)
