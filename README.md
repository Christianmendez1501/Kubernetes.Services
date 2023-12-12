# Como Desplegar servicios en Kubernetes

![Kubernetes](https://imgs.search.brave.com/nVRNLIUhUys1o8szFLUcty6gbZB34r_BJr-I9gdyybY/rs:fit:560:320:1/g:ce/aHR0cHM6Ly91cGxv/YWQud2lraW1lZGlh/Lm9yZy93aWtpcGVk/aWEvY29tbW9ucy90/aHVtYi82LzY3L0t1/YmVybmV0ZXNfbG9n/by5zdmcvNjQwcHgt/S3ViZXJuZXRlc19s/b2dvLnN2Zy5wbmc)

¡Bienvenido a esta guía educativa sobre cómo desplegar servicios en Kubernetes! En esta guía, aprenderemos a desplegar tres servicios ficticios en Kubernetes utilizando archivos YAML. Vamos a crear un namespace llamado casafeudal y desplegar servicios de "horarios", "trayectos", y "comilonas".
Paso 1: Crear un Namespace

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: casafeudal
```

Este archivo YAML crea un namespace llamado casafeudal que actuará como un entorno aislado para nuestros servicios.
Paso 2: Desplegar "Horarios"
Deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: horarios-deployment
  namespace: casafeudal
spec:
  replicas: 100
  selector:
    matchLabels:
      app: horarios
  template:
    metadata:
      labels:
        app: horarios
    spec:
      containers:
      - name: horarios
        image: casafeudal/horas:1.2.1
        ports:
        - containerPort: 5045
        env:
        - name: IDPERIODISTA
          valueFrom:
            secretKeyRef:
              name: horarios-secrets
              key: idperiodista
        - name: CONTRASENA
          valueFrom:
            secretKeyRef:
              name: horarios-secrets
              key: contrasena
      initContainers:
      - name: download-horarios
        image: curlimages/curl
        command: ["curl", "-O", "https://www.generochico.com/horariosjuergasdelviejo.csv"]
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

Este archivo YAML define el despliegue de "horarios". Se despliegan 100 replicas y se utiliza un initContainer para descargar los horarios.
CronJob

```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: horarios-cronjob
  namespace: casafeudal
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: horarios
            image: casafeudal/horas:1.2.1
            env:
            - name: IDPERIODISTA
              valueFrom:
                secretKeyRef:
                  name: horarios-secrets
                  key: idperiodista
            - name: CONTRASENA
              valueFrom:
                secretKeyRef:
                  name: horarios-secrets
                  key: contrasena
          initContainers:
          - name: download-horarios
            image: curlimages/curl
            command: ["curl", "-O", "https://www.generochico.com/horariosjuergasdelviejo.csv"]
          restartPolicy: OnFailure
```


Este archivo YAML configura un CronJob para ejecutarse cada hora, asegurando actualizaciones regulares.
Secret

```yml
apiVersion: v1
kind: Secret
metadata:
  name: horarios-secrets
  namespace: casafeudal
type: Opaque
data:
  idperiodista: base64_encoded_idperiodista
  contrasena: base64_encoded_contrasena
```


Este archivo YAML crea un Secret en el namespace casafeudal para almacenar información sensible como las credenciales de los periodistas.

### Para desplegar los servicios de nuestro proyecto, utilizaremos el comando 
```bash
kubectl apply -f .
```
en resumen, con estas configuraciones, has aprendido a desplegar servicios en Kubernetes de manera efectiva. ¡Ahora, tu aplicación estará lista para enfrentar desafíos en cualquier parte del mundo! ¡Buena suerte!

**Fuente:**

> https://kubernetes.io/docs/tutorials/stateless-application/expose


