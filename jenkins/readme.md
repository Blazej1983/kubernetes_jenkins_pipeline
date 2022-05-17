# moje wypociny na vagrant 
	1. kubectl create ns jenkins
	2. Klonujemy repo gitowe i zgrywamy folder jenkins na kubernetesa mastera
	3. I dpalamy jamle z folderu 
	4.  kubectl -n jenkins apply -f /home/vagrant/jenkins
	5. kubectl -n jenkins get pods	Na namespace jenkins bedą sie budować kontenery
	jenkins$ kubectl -n jenkins get svc	
	 kubectl -n jenkins get svc
	NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                     AGE
	jenkins   ClusterIP   10.106.201.24   <none>        8080/TCP,50000/TCP,80/TCP   135m
	vagrant@kubemaster:~/jenkins$ kubectl -n jenkins get pods
	NAME                      READY   STATUS    RESTARTS   AGE
	jenkins-b4d464798-rwjkb   1/1     Running   0          135m
	
	Patrzymy w loga aby pobrać klucz do jenkinsa
	9e4793f6bfcb46aba744fc6f90bbf5ce
	I robimy forward  svc na duzy port 
	
	 kubectl -n jenkins expose pod jenkins-b4d464798-rwjkb --name jenkins --type NodePort --port 8080 --target-port 8080
	kubectl get all patrzymy na port svc  ten duzy i wpbijamy po adresie ip do nginx
	http://192.168.56.2:31045/
	
Instalujemy sugerowane pluginny i pluggin kubernetes This plugin integrates Jenkins with Kubernetes
![image](https://user-images.githubusercontent.com/25319617/168824358-22829257-0819-49f1-874c-0603be6b5635.png)




# Jenkins on Amazon Kubernetes

For running Jenkins on AMAZON, start [here](./amazon-eks/readme.md)

# Jenkins on Local (Docker Windows \ Minikube \ etc)

For running Jenkins on Local Docker for Windows or Minikube <br/>
Watch the [video](https://youtu.be/eRWIJGF3Y2g)

# Setting up Jenkins Agent

After installing `kubernetes-plugin` for Jenkins
* Go to Manage Jenkins | Bottom of Page | Cloud | Kubernetes (Add kubenretes cloud)
* Fill out plugin values
    * Name: kubernetes
    * Kubernetes URL: https://kubernetes.default:443
    * Kubernetes Namespace: jenkins
    * Credentials | Add | Jenkins (Choose Kubernetes service account option & Global + Save)
    * Test Connection | Should be successful! If not, check RBAC permissions and fix it!
    * Jenkins URL: http://jenkins
    * Tunnel : jenkins:50000
    * Apply cap only on alive pods : yes!
    * Add Kubernetes Pod Template
        * Name: jenkins-slave
        * Namespace: jenkins
        * Service Account: jenkins
        * Labels: jenkins-slave (you will need to use this label on all jobs)
        * Containers | Add Template
            * Name: jnlp
            * Docker Image: aimvector/jenkins-slave
            * Command to run : <Make this blank>
            * Arguments to pass to the command: <Make this blank>
            * Allocate pseudo-TTY: yes
            * Add Volume
                * HostPath type
                * HostPath: /var/run/docker.sock
                * Mount Path: /var/run/docker.sock
        * Timeout in seconds for Jenkins connection: 300
* Save

# Test a build

To run docker commands inside a jenkins agent you will need a custom jenkins agent with docker-in-docker working.
Take a look and build the docker file in `./dockerfiles/jenkins-agent`
Push it to a registry and use it instead of above configured `* Docker Image: jenkins/jnlp-slave`
If you do not use the custom image, the below pipeline will not work because default `* Docker Image: jenkins/jnlp-slave` public image does not have docker ability.

* Add a Jenkins Pipeline

```
node('jenkins-slave') {
    
     stage('unit-tests') {
        sh(script: """
            docker run --rm alpine /bin/sh -c "echo hello world"
        """)
    }
}
```
