# grails-react-k8s

This project is to demonstrate how to deploy an **React** application built with **Grails** to the **EnSaaS 4.0**.



# Deploying to the EnSaaS 4.0

## 1. Getting Your Own Credentials

Go to your [workspace](https://portal-mp-master.es.wise-paas.cn/cluster-info/workspaces) and download the credentials required to manipulate your resources.

![get-config](./img/get-config.png)

You will get prompted to download a config file which contains important credentials. Please make sure you secure this file and **DO NOT LET ANYONE ELSE ACCESS YOUR CONFIG FILE**. You may rename it to whatever you want, say ``wisepass-config``, and then save it to ``$HOME/.kube`` if you are using MacOS.

<br>

## 2. Setting Up KUBECONFIG

If you already have **minikube** installed on your local machine, you will have a file called ``config`` in the directory ``$HOME/.kube``, which is the default file used by Kubernetes. After you start the **minikube**, you can type ``kubectl config view`` to see the information inside this config file.

You may overwrite the ``KUBECONFIG`` environment variable to direct Kubernetes to use the new one that you just downloaded, type ``export KUBECONFIG=$HOME/.kube/wisepaas-config``. Again, you can name it accroding to personal preference. Just make sure that the envrironment variable matches your file name.

<br>

## 3. Clone the Sample Code from WISE-PaaS GitHub

```bash
git clone git@github.com:WISE-PaaS/grails-react-k8s.git
cd grails-react-k8s/
```

<br>

## 4. Building & Pushing Docker Image

In this project, we use **Grails** to create an **React** application. While building the image, we copy over the ``build`` folder to an Nginx container to serve the application.

**Dockerfile:**

```dockerfile
FROM tubbynl/grails as builder
WORKDIR /app
RUN grails create-app myreact -profile react
WORKDIR /app/myreact
RUN ./gradlew client:build
# CMD ["./gradlew", "client:start"]

FROM nginx
EXPOSE 3000
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/myreact/client/build /usr/share/nginx/html
```

<br>

First, build the image while specifying a tag.

```bash
docker build -t <DockerHubAccount>/<ImageName>:<Tag> .
```

For example: ``wise-paas/grails-react:v1.0.0``

Then, push the new image to your own Docker Repository.

```bash
docker push <DockerHubAccount>/<ImageName>:<Tag>
```

<br>

## 5. Do a Little Bit Tuning

It requires some minimun effort to take this code and deploy it to the EnSaaS 4.0.

### I. Setting Up the Host

Modify the ``host`` attribute in ``./k8s/ingress-service.yaml``.

![ingress-host](./img/ingress-host.png)

<br>

**Format:**

``ApplicationName``.``Workspace``.``Cluster``.**internal**

In this example, the ``ApplicationName`` is ``grails``. Note that ``internal`` keyword is **required** at the end of the string.

<br>

### II. Changing the Docker Image Name

Change the ``image`` attribute inside ``./k8s/client-deployment.yaml`` file according to the image you just built and pushed.

```
<DockerHubAccount>/<ImageName>:<Tag>
```

Here is the example in this code:

![docker-image](./img/docker-image.png)

<br>

### III. Setting Up the Resource Usage

Adjust the ``resources`` attribute inside ``./k8s/client-deployment.yaml`` file.

![resources](./img/resources.png)

<br>

## 5. Applying the Configuration

**Note**: Always remember to specify the ``namespace`` that you are working on while applying the Kubernetes configurations.

The following screenshot shows that the namespace is *playgroud*. Yours might be different.

![namespace](./img/namespace.png)

<br>

Apply the Kubernetes configuration.

```bash
kubectl apply -f k8s/ --namespace <NameSpace>
```

<br>

Check all resources (also with namespace):

```bash
kubectl get all --namespace playground

NAME                                   READY   STATUS    RESTARTS   AGE
pod/client-deployment-9d9f994b-4q6zl   1/1     Running   0          40m

NAME                                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/client-cluster-ip-service   ClusterIP   10.0.91.167   <none>        3000/TCP   44m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/client-deployment   1/1     1            1           40m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/client-deployment-9d9f994b   1         1         1       40m
```

<br>

## 6. Verifying the Application

In the sample code, we set the host for the Ingress service.

```yaml
host: grails.se.slave04.internal
```

So, to see the page, we need to go to the following URL:

http://grails-se-slave04.es.wise-paas.cn/

<br>

Mapping rule -- host vs. URL

- **host**

  ```
  <AppName>.<Workspace>.<Cluster>.internal
  ```

- **External URL**

  ```
  <AppName>-<Workspace>-<Cluster>.es.wise-paas.cn
  ```

  <br>

The website should look like:

![ensaas-4.0](./img/result.png)



