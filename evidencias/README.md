# Laboratorio 3 - Despliegue CI/CD en Kubernetes

## Alumno

Pablo Anavalon

## Descripción

Este laboratorio implementa un flujo CI/CD completo para una aplicación desarrollada con NestJS.

El flujo permite:

- Instalar dependencias.
- Ejecutar pruebas automáticas.
- Construir una imagen Docker.
- Publicar la imagen en Docker Hub.
- Desplegar la aplicación en Kubernetes.
- Automatizar todo el proceso mediante Jenkins.

## Tecnologías utilizadas

- Node.js
- NestJS
- pnpm
- Docker
- Docker Hub
- Kubernetes
- Jenkins
- Jenkins Kubernetes Plugin

## Arquitectura general

```text
Código fuente
   ↓
GitHub
   ↓
Jenkins
   ↓
Agente Kubernetes
   ↓
install
   ↓
test
   ↓
build
   ↓
push
   ↓
deploy
   ↓
Kubernetes
```

Jenkins utiliza un agente Kubernetes definido en `agent.yaml`.

El agente contiene contenedores para:

```text
node
docker
kubectl
```

## Repositorio Git

```text
https://github.com/panavalong/lab03.git
```

Rama principal:

```text
main
```

## Imagen Docker

```text
panavalong/lab3:pablo-anavalon
```

## Recursos Kubernetes

```text
Namespace:   ns-pablo-anavalon
Deployment:  deployment-pablo-anavalon
Service:     svc-pablo-anavalon
ConfigMap:   config-pablo-anavalon
Secret:      secret-pablo-anavalon
```

El Deployment utiliza 2 réplicas.

## Configuración de la aplicación

La aplicación utiliza:

```text
AMBIENTE
API_KEY
```

`AMBIENTE` se obtiene desde un ConfigMap.

`API_KEY` se obtiene desde un Secret de Kubernetes.

# Ejecución manual

## 1. Instalar dependencias

```bash
pnpm install
```

## 2. Ejecutar pruebas

```bash
pnpm test
```

Resultado esperado:

```text
Test Suites: 2 passed
Tests: 5 passed
```

## 3. Construir la imagen Docker

```bash
docker build -t panavalong/lab3:pablo-anavalon .
```

## 4. Publicar la imagen en Docker Hub

```bash
docker login
docker push panavalong/lab3:pablo-anavalon
```

## 5. Desplegar en Kubernetes

```bash
kubectl apply -f entrega.yaml
```

## 6. Verificar el Deployment

```bash
kubectl get deployment -n ns-pablo-anavalon
```

## 7. Verificar los Pods

```bash
kubectl get pods -n ns-pablo-anavalon
```

Resultado esperado:

```text
2 Pods en estado Running
```

## 8. Verificar el Service

```bash
kubectl get svc -n ns-pablo-anavalon
```

## 9. Verificar las variables de entorno

```bash
kubectl exec deployment/deployment-pablo-anavalon \
  -n ns-pablo-anavalon -- \
  printenv | grep -E 'AMBIENTE|API_KEY'
```

Resultado esperado:

```text
AMBIENTE=produccion
API_KEY=api-key-pablo-anavalon
```

## 10. Verificar logs

```bash
kubectl logs deployment/deployment-pablo-anavalon \
  -n ns-pablo-anavalon
```

# Prueba de la aplicación

En una terminal:

```bash
kubectl port-forward \
  svc/svc-pablo-anavalon \
  8080:80 \
  -n ns-pablo-anavalon
```

En otra:

```bash
curl http://localhost:8080/lab
```

Resultado esperado:

```json
{
  "AMBIENTE": "produccion",
  "API_KEY": "api-key-pablo-anavalon"
}
```

# Pipeline Jenkins

El pipeline está definido en `Jenkinsfile` y utiliza un agente Kubernetes definido en `agent.yaml`.

Stages implementados:

```text
install
test
build
push
deploy
```

## Stage install

```bash
pnpm install --frozen-lockfile
```

## Stage test

```bash
pnpm test
```

## Stage build

```bash
docker build -t panavalong/lab3:pablo-anavalon .
```

## Stage push

Publica la imagen en Docker Hub.

Las credenciales no están escritas directamente en el Jenkinsfile.

Se utiliza la credencial Jenkins:

```text
dockerhub-credentials
```

## Stage deploy

```bash
kubectl apply -f entrega.yaml
```

Luego:

```bash
kubectl rollout status \
  deployment/deployment-pablo-anavalon \
  -n ns-pablo-anavalon \
  --timeout=120s
```

# Permisos de Jenkins

Jenkins utiliza el ServiceAccount:

```text
jenkins
```

Los permisos RBAC para desplegar en `ns-pablo-anavalon` se configuran mediante:

```text
jenkins-rbac.yaml
```

# Resultado final

```text
Pipeline ejecutado correctamente
Finished: SUCCESS
```

Flujo final:

```text
Código
  ↓
GitHub
  ↓
Jenkins
  ↓
Docker
  ↓
Docker Hub
  ↓
Kubernetes
```

# Evidencias

La carpeta `evidencias/` contiene las salidas y capturas del laboratorio, incluyendo:

```text
kubectl cluster-info
kubectl get nodes
kubectl get pods
kubectl get deployment
kubectl get svc
kubectl logs
kubectl exec
kubectl get configmap
kubectl get secret
curl http://localhost:8080/lab
Pipeline Jenkins exitoso
```

# Estructura principal del proyecto

```text
lab03/
├── .dockerignore
├── Dockerfile
├── Jenkinsfile
├── README.md
├── agent.yaml
├── entrega.yaml
├── jenkins-rbac.yaml
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── src/
├── test/
└── evidencias/
```
