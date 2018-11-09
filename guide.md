PyconUk - Kubernetes Workshop
=============================

Home instructions
-----------------

- laptop requirement:
    * ~4 GB of RAM
    * ~15 GB of free disk space
- Download & install virtualbox: https://www.virtualbox.org/wiki/Downloads
   - Download & unzip our disk image: https://tinyurl.com/y9tvcsbj
- Download & install ssh client (mac & linux have this usually installed), for windows:
  https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
- When following after the workshop at Pycon UK, see the guide at:
  https://github.com/alixedi/pyconuk-2018-k8s

Session 1 - 90m
---------------

1. Introduction to speaker

2. After this class we hope you will be able to:
    * Demonstrate Kubernetes to your colleagues.
    * Deploy a web application to Kubernetes.
    * Manage your deployment e.g. scaling up and down, rolling updates and rollbacks. 
    * Deploy multiple services on Kubernetes.
    * Describe the Kubernetes API.
    * Demonstrate using kubectl to interact with the Kubernetes API.

   Time limitations may mean you have to achieve some objectives after class.

3. Installation
    * Laptop requirements:
       * ~4 GB of RAM
       * ~15 GB of free disk space
       * Virtualbox
       * ssh client
    * Download the virtual disk - Ask us for a USB or download from [here](https://tinyurl.com/y9tvcsbj)
    * Unzip the virtual disk
    * Create a virtual machine:
      * Type: Linux, version: Ubuntu 64 bit
      * 2048 RAM
      * "use existing virtual hard disk file" (use the unziped virtaul disk from the previous step)
      * Create 
      * machine -> settings
      * network -> advanced -> port-forwarding -> create new rule: host port: 2222, guest port: 22
    * You VM image has microk8s (https://asciinema.org/a/182634) installed. This includes Docker and Kubernetes.
    * Start the VM
    * Verify that everything is working
      * ssh into your machine: `ssh osboxes@127.0.0.1 -p 2222` (if using putty: there is a field for the port)
      * login with username: osboxes password: osboxes.org
      * Verify `kubectl get pods --all-namespaces`, if this fails, wait for a one minute (can take up to several minutes),
        then try again, expected output, example:
        ```
        NAMESPACE     NAME                                              READY     STATUS    RESTARTS   AGE
        kube-system   heapster-v1.5.2-577898ddbf-kt27r                  4/4       Running   8          4d
        kube-system   kube-dns-864b8bdc77-cwsmw                         3/3       Running   6          4d
        kube-system   kubernetes-dashboard-6948bdb78-pnsq6              1/1       Running   2          4d
        kube-system   monitoring-influxdb-grafana-v4-7ffdc569b8-5wq4s   2/2       Running   4          4d
        ```
      * update the repo to the latest: `cd ~/pyconuk-2018-k8s && git pull`
      * It's usefull to use ipython, as it makes copying wrongly indented code from the guide easier: `pip3 install ipython`
      * pull the latest redis image to save time later in the workshop: `docker pull redis` (sudo password is the same as the login password)
    * We will encourage you to pair with someone

4. Hello World! (code is in step0)
    * The most basic Flask app in the world:
    
    ```python
    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def hello():
        return "Hello World!"
    ```
    
   * go the the example directory for this code: `cd step0`
   * install flask: `pip3 install flask`
   * You can run it like: `FLASK_APP=hello.py flask run`
   * Open a second ssh terminal or use fg/bg if you are familiar with this:
     `curl localhost:5000`, output: `Hello World!`

5. Hello Docker World!
    * A very basic dockerfile for our hello world app can look like:

    ```docker
    FROM python
    RUN pip install flask
    ADD hello.py /
    ENV FLASK_APP=/hello.py
    CMD ["flask", "run", "-h", "0.0.0.0"]
    ```
    
    * Build a docker image: `docker build . -t hello:local`
    * Now you have a docker image: `docker images |grep hello`
    * Run the image: `docker run --net host hello:local`
    * access the service again: `curl localhost:5000`
    <!--- 30 minutes --->

6. A very brief intro to Docker
    * Versus virtualenv, conda and VMs
    * A brief explanation of images, containers etc.
    * image is a lightweight virtual machine image with isolation
    * Docker is like virtualenv but it isolated not just python packages but the filesystem, network interfaces and system libraries, and other.
      Docker also standarizes (a lot of things) on how you run applications.

7. Interactive Console (code is in step1)

    ```python
    import code
    import io
    import contextlib
    
    import flask
    
    
    app = flask.Flask(__name__)
    app.consoles = {}
    
    
    class WebConsole:
    
        def __init__(self):
            self.console = code.InteractiveConsole()
    
        def run(self, code):
            output = io.StringIO()
            with contextlib.redirect_stdout(output):
                with contextlib.redirect_stderr(output):
                    for line in code.splitlines():
                        self.console.push(line)
            return {'output': str(output.getvalue())}
    
    
    @app.route('/api/<uname>/run/', methods=['POST'])
    def run(uname):
        if not uname in app.consoles:
            app.consoles[uname] = WebConsole()
        return flask.jsonify(
            app.consoles[uname].run(
                flask.request.get_json()['input']
            )
        )
    ```

8. Assignment #1: webconsole in docker
    * Display instructions from step 5 + the ones below
    * Run the application directly
    * Create a `Dockerfile` for webconsole
    * Build the image
    * Run the image
    * What happens when you try and use requests POST something to: `/api/<username>/run/`, for example:
       ```python
        import requests
        requests.post('http://localhost:5000/api/ali/run/',
                      json={'input': 'print("Hello World")'}).json()
        ```
    <!--- 1 hour -->

9. Introduction to Kubernetes
    * Challenges of building modern applications
        * Complexity
        * Load characteristics
        * Horizontal scaling
        * CI/CD
    * Microservice - when are they useful and why? 
    * Kubernetes - what and why? - solves most of the microservices problems

10. Kubernetes Web Console (code in step2)
    * Build the image:
      * `cd ../step2`
      * `./build.sh`
    * Run as service deployment: 
    ```bash
    kubectl run webconsole \
        --image pyconuk-2018-k8s:step2 \
        --port 5000 \
        --replicas 2 \
        --expose
    ```
    
   * Explain pods, deployments, and services
   * Access the service:
     * Grab the service ip: 
     `export WEBCONSOLE_IP=$(kubectl get service webconsole -o go-template="{{ .spec.clusterIP  }}")`
     * Use the service:
       ```python
       import requests
       import os
       requests.post(f'http://{os.environ["WEBCONSOLE_IP"]}:5000/api/ali/run/',
                     json={'input': 'print("Hello World")'}).json()
       ```
     
11. Show some kubernetes features  
   * Try and scale the deployment - show the new pods being created
     `kubectl scale deployment webconsole --replicas 5`
   * Look at the new pods: `kubectl get pods`
   * Try and kill a pod - show that it gets recreated
     `kubectl delete <pod-name>`
   * Simulate a service failure:   
       ```python
       import requests
       import os
       requests.get(f'http://{os.environ["WEBCONSOLE_IP"]}:5000/api/crash/')
       ```
   * Show restarts on the pod: `kubectl get pods`

12. Introduction to kubectl and Kubernetes API
    * `$ kubectl --help` has sections for basic commands - beginners and intermediate.

13. Assignment:
    * Build the image:
      * `cd ../step2`
      * `./build.sh`
    * Run as service deployment: 
    ```bash
    kubectl run webconsole \
        --image pyconuk-2018-k8s:step2 \
        --port 5000 \
        --replicas 2 \
        --expose
    ```
    * Lets explore a few kubectl commands.
    ```
        kubectl get pods
        kubectl get deployments
        kubectl get services
        kubectl --help
        kubectl explain
        kubectl describe
    ```
    * Assignment: See if you can figure out a problem with our application

Session 2 - 90m
---------------

1. Introduction to kubernetes manifests:
   * Show:
    ```
    kubectl run webconsole \
        --image pyconuk-2018-k8s:step2 \
        --port 5000 \
        --replicas 2 \
        --expose \
        --dry-run -o yaml 
   ```
   * Walk through of the simplified yaml produced:
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: webconsole
    spec:
      replicas: 2
      template:
        metadata:
          labels:
            app: webconsole
        spec:
          containers:
          - image: pyconuk-2018-k8s:step2
            name: webconsole
            ports:
              - name: api
                containerPort: 5000
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: webconsole
    spec:
      ports:
      - name: webconsole
        port: 5000
        targetPort: api
      selector:
        app: webconsole
    ```
   * Questions?
    
2. Infrastructure as code, why is this nice?
   * Ask the room what the preference is, between cli and yaml and why?
   * You can code review YAML, and essentially it's infrastructure as code.
   * Yaml is declarative, for instance if you want to scale up your application
     you would edit the yaml, and re-apply instead of running specific commands on the cli. 

3. Back to the example, demonstrate the problem:
    * grab the service ip:
    `export WEBCONSOLE_IP=$(kubectl get service webconsole -o go-template="{{ .spec.clusterIP  }}")`
    * demonstrate:
    ```python
    import requests
    import os
    requests.post(f'http://{ os.environ["WEBCONSOLE_IP"] }:5000/api/paul/run/',
                  json={'input': 'a = 1'}).json()
    requests.post(f'http://{ os.environ["WEBCONSOLE_IP"] }:5000/api/paul/run/',
                  json={'input': 'print(a)'}).json()
    ```
    * Ask if people understand why?
    * Explain the issue with load balancing

4. Split the Web Console to solve the problem (code is in step3):
    * Introduce jobs
    * Quick walkthrough of [consolehub.py](https://github.com/alixedi/pyconuk-2018-k8s/blob/master/step3/consolehub/consolehub.py)
    * Walk-through of [job_template.yaml](https://github.com/alixedi/pyconuk-2018-k8s/blob/master/step3/consolehub/job-template.yaml)
    * (see step 5 for instructions: delete, build, apply)
    <!--- 30 minutes --->

5. Assignment, run step 3: 
   * Remove the old service and deployment:
    ```
    kubectl delete service webconsole
    kubectl delete deployment webconsole
    ```
   * Build: `./step3/build.sh`
   * Apply the manifest: `kubectl apply -f step3/consolehub/deployment.yaml`
   * grab the service ip:
   `export CONSOLEHUB_IP=$(kubectl get service consolehub -o go-template="{{ .spec.clusterIP  }}")`
   * Use the application, check if the problem is solved, example:
    ```python
    import requests
    import os
    requests.post(f'http://{ os.environ["CONSOLEHUB_IP"] }/api/paul/start/').json()
    requests.post(f'http://{ os.environ["CONSOLEHUB_IP"] }/api/paul/run/',
                    json={'input': 'a = 1'}).json()
    requests.post(f'http://{ os.environ["CONSOLEHUB_IP"] }/api/paul/run/',
                    json={'input': 'print(a)'}).json()
    ```

6. Problems with the current implementation:
  * Tight coupling of the application code and infrastructure
  * You create the job synchronously
  <!--- 60 minutes --->
  
7. Introduce the final version (code in step 4):
    * Quick walkthrough of [consolehub.py](https://github.com/alixedi/pyconuk-2018-k8s/blob/master/step4/consolehub/consolehub.py)
    * Only the provisioner is coupled to kubernetes [provisioner.py](https://github.com/alixedi/pyconuk-2018-k8s/blob/master/step4/provisioner/provisioner.py)
    
8. Demonstrate rolling update (see assignment)
    ```python
    import requests
    import os
    requests.post(f'http://{ os.environ["CONSOLEHUB_IP"] }/api/paul/run/',
                    json={'input': 'print(a)'}).json()
    ```

    
9. Assignment, do the rolling update:
   * Build: `./step4/build.sh`
   * Apply the manifest: `kubectl apply -f step4/k8s_manifests`
   * use `kubectl get pods` to see pods shutdown and start in a rolling way

10. Next steps:
    * Local running options:
      * docker for mac & windows: https://blog.docker.com/2018/01/docker-mac-kubernetes/
      * minikube (sometimes fiddly) or microk8s (alpha)
        * Connect to a service in a production cluster:
          * proxy: `kubectl proxy` (explain what this does)
          * Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
    * Production clusters:
      * gcp/aws/digital ocean
      * Other options: https://kubernetes.io/docs/setup/     

11. Q&A
