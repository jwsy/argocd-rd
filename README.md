# ArgoCD on Rancher Desktop
With help and inspiration from https://github.com/tarosaiba/argocd-on-rancher-desktop

## Usage
1. Install ArgoCD

    Follow the [getting started instructions on the Argo CD website](https://argo-cd.readthedocs.io/en/stable/getting_started/). The first step creates a new namespace, `argocd`, where Argo CD services and application resources will live.

    ``` bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

2. Deploy the additional configmap and redeploy the app 

    This sets the server to be in insecure mode (so we don't need to config a certificate) as [described in the Argo CD docs](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/). Then, restart the Argo CD server

    ``` bash
    kubectl -n argocd apply -f argocd-cmd-params-cm.yaml
    kubectl -n argocd rollout restart deployment argocd-server
    ```

3. Deploy the ingress

    There are two options to deploy the ingress

    1. Use a normal ingress. I like this because it shows up nicely with the hostname when I run `kubectl get all,ing`

        ``` bash
        kubectl -n argocd apply -f argocd-ingress-server.yaml
        ```

    2. If you're using Rancher Desktop, you can use the Traefik ingress

        ``` bash
        kubectl -n argocd apply -f traefik/argocd-ingress-server-traefik.yaml
        ```

4. Log into and get to know ArgoCD

    Visit the app by logging in with username `admin` and the password in `argocd-initial-admin-secret`. You can get that from the [command provided in the docs](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli) shown below

    ``` bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

## Additional steps to deploy the jade-shooter app

1. Create a new app in Argo CD by clicking the **NEW APP** button in the upper left 
- Application name: set to some app name. Example: `simplest-k8s`
- Select the default project name: `default`
- Leave the checkboxes *empty* for the deletion finalizer, sync options, and prune propagation properties
- SOURCE Repository URL: set to the repository. Example: `https://github.com/jwsy/simplest-k8s`
- SOURCE Revision: set to the branch to sync. Example: `main`
- SOURCE Path: set to the path where manifests files are. Example: `.`
- DESTINATION Cluster URL: set to the cluster. Example: `https://kubernetes.default.svc`
- DESTINATION Namespace: set to the namespace where the app will deploy to: `https://kubernetes.default.set`

2. Sync with a dry run to check your manifests
3. Sync the app and watch the magic!