In most of our examples, we use a test application called Whoami - a simple echo application that displays header from the containers. Here is a basic deployment as well as a service to explose the application within your Kubernetes cluster. 


```yaml
cat << EOF | kubectl apply -f -
kind: Deployment
apiVersion: apps/v1
metadata:
  name: whoamiv1
  labels:
    name: whoamiv1
  namespace: default  
spec:
  replicas: 1
  selector:
    matchLabels:
      task: whoamiv1
  template:
    metadata:
      labels:
        task: whoamiv1
    spec:
      containers:
        - name: whoamiv1
          image: traefik/traefikee-webapp-demo:v2
          args:
            - -ascii
            - -name=WHOAMI
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /ping
              port: 80
            failureThreshold: 1
            initialDelaySeconds: 2
            periodSeconds: 3
            successThreshold: 1
            timeoutSeconds: 2
EOF
```

```yaml
cat << EOF| kubectl apply -f - 
apiVersion: v1
kind: Service
metadata:
  name: whoamiv1
  namespace: default
spec:
  ports:
    - name: http
      port: 80
  selector:
    task: whoamiv1
EOF    
```

Once the Whoami application is deployed we have to create IngressRoute object to expose that application. 

```yaml
cat << EOF | kubectl apply -f -
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-crd-https
  namespace: default
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`whoami.demo.traefiklabs.tech`)
      services:
        - kind: Service
          name: whoamiv1
          port: 80
  tls:
    certResolver: le # we assume that you have added a certificate resolver, if no just use the empty tls: {} object
EOF    
```

The application should be now accessible by reaching the following example URL:

```sh
open https://whoami.demo.traefiklabs.tech
```

As you can see everyone is able to visit the application and to see the headers displayed by the Whoami application. The next step is to protect that application by deploying [OpenID Connect Middleware](https://doc.traefik.io/traefik-enterprise/middlewares/oidc/)
