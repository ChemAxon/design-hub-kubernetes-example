This is an example how to set up Design Hub and its dependencies in Kubernetes. It contains the minimum required to run and in a real setup it probably needs further refinement.

#### Details ####
There are two parts of the example. Base part is stateless in the sense of not containing configurations, credentials and ChemAxon licenses.
The other part contains an example configuration prepared for MiniKube where you only need to add your ChemAxon license and 'hub.chemaxon.com' credentials to make it work.
In a production environment the base part probably need more fine-tuning, and the configuration part should be obviously tailored to your requirements.

#### Try out the example with MiniKube ####

1. ##### Copy ChemAxon licenses #####
    * put your ChemAxon license file under `./license/license.cxl`
   
2. ##### Install and run Kubernetes tools #####
    * install kubectl command line tool to manage Kubernetes cluster. https://kubernetes.io/docs/tasks/tools/install-kubectl/
    * install MiniKube to be able to locally try out Kubernetes. https://minikube.sigs.k8s.io/docs/start/
    ```console
    minikube start
    ```
3. ##### Mount folder to MiniKube #####
    As MiniKube runs in a container you need to mount it first there to be able to use it within the Design Hub deployment.
    ```console
    minikube mount "/<path-to-this-repo-folder>/minikube-example/:/data/"
    ```
4. ##### Configure Docker credentials #####
    Please note that when using Docker on Windows without WSL you have to modify `%USERPROFILE%\.docker\config.json` - remove part with `"credsStore": "desktop"`
    
    Login to ChemAxon Hub Docker repository. Follow steps described here: https://chemaxon.com/products/design-hub/download
    Once successfully logged in, then create a Kubernetes secret from it:
    ```console
    kubectl create secret generic docker-hub-chemaxon --from-file=.dockerconfigjson=<path/to/.docker/config.json> --type=kubernetes.io/dockerconfigjson
    ```   
5. ##### Deploy volumes #####
    ```console
    cd minikube-example/
    kubectl apply -f config-volume.yaml -f license-volume.yaml -f services-volume.yaml
    ```
6. ##### Deploy Postgre #####
    Design Hub requires at start-up to access Postgre, so we should start with that.
    ```console
	cd ..
    cd base/
    kubectl apply -f postgre.yaml
    ```

7. ##### Deploy applications #####
    ```console
    kubectl apply -f config-volume-claim.yaml -f license-volume-claim.yaml -f services-volume-claim.yaml -f jms.yaml -f marvinjs.yaml -f design-hub.yaml
    ```

8. ##### Open Design Hub #####
    ```console
    minikube service design-hub-service
    ```
   
#### Plugins ####
   To add a plugin, do the following steps:
   1. copy plugin files into the 'services' folder
   2. Install the plugin
   ```console
    npm init
    npm install
   ```
   3. Restart Design hub
   ```console
    kubectl rollout restart deployment design-hub-deployment
   ```
