### Dockerfile

```dockerfile
FROM  python:3
LABEL "vendor"="klv"
LABEL version="v1.1"
LABEL description="Aplicação de cálculo de idade com JavaScript e servidor python"
LABEL maintainer="rfabriciors@gmail.com"

RUN apt-get update -y && \
   apt-get install -y python python3-pip python-dev

RUN pip install Flask

WORKDIR /app

COPY . .

EXPOSE 5000

CMD [ "python", "./app.py"]
```

### deployment-appidadeklv.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: appidadeklv
  name: appidadeklv
spec:
  replicas: 2
  selector:
    matchLabels:
      app: appidadeklv
  template:
    metadata:
      labels:
        app: appidadeklv
    spec:
      containers:
      - image: rfabricio/appidadeklv:latest
        name: appidadeklv
        ports:
        - containerPort: 5000
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 3
          periodSeconds: 2
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 2
        readinessProbe:
          httpGet:
            path: /ready
            port: 5000
          initialDelaySeconds: 3
          periodSeconds: 2
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 2          
        resources: {}
status: {}
```

*o arquivo heath.txt serve como referência para o recurso livenessProbe*

### service-appidadeklv.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: appidadeklv
  name: appidadeklv
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 5000
    nodePort: 30000
  selector:
    app: appidadeklv
  type: NodePort
status:
  loadBalancer: {}
  ```

## Disparando o deployment

```bash
kubectl apply -f deployment-appidadeklv.yaml
```

## Expondo o serviço

```bash
kubectl apply -f deployment-appidadeklv.yaml
```

O serviço é verificado através do recurso *livenessProbe*

```yaml
        livenessProbe:
          httpGet:
            path: /health.txt
            port: 80
```

O *livenessProbe* pode ser verificado através da descrição do pod:

```bash
kubectl describe pod <pod_name>
.
.
.
Liveness:       http-get http://:80/health.txt delay=0s timeout=1s period=10s #success=1 #failure=3
```