## Init

branche tilt-step1

### bookstore-pg


```python
k8s_yaml(['bookstore-pg/kubernetes/deployment.yml', 'bookstore-pg/kubernetes/service.yml'])
```

### bookstore-api

```python
docker_build('k3d-registry:46697/api', 'bookstore-api')
k8s_yaml(['bookstore-api/kubernetes/deployment.yml', 'bookstore-api/kubernetes/service.yml'])
```


### bookstore-gui

```python
docker_build('k3d-registry:46697/gui', 'bookstore-gui')
k8s_yaml(['bookstore-gui/kubernetes/deployment.yml', 'bookstore-gui/kubernetes/service.yml'])
```

## Port Forward

branche tilt-step2

### bookstore-pg 

```python
k8s_resource('pg-deployment', port_forwards=5432)
```

Montrer port exposé dans IHM
Faire démo client postgresql local

### bookstore-api 

```python
k8s_resource('api-deployment', port_forwards=8090, resource_deps=['pg-deployment'])
```

Test requête curl

```bash
curl localhost:8090/books
```

### bookstore-gui

```python
k8s_resource('gui-deployment', port_forwards=8080, resource_deps=['api-deployment'])
```

Démo IHM

## Orchestration

branche tilt-step3

### bookstore-api 

```python
k8s_resource('api-deployment', port_forwards=8090, resource_deps=['pg-deployment'])
```

Test requête curl

```bash
curl localhost:8090/books
```

### bookstore-gui

```python
k8s_resource('gui-deployment', port_forwards=8080, resource_deps=['api-deployment'])
```

Démo IHM

## Optimisation FileSync (npm)

branche tilt-step4

```python
docker_build('k3d-registry:46697/gui', 'bookstore-gui',
    entrypoint='npm run serve',
    live_update=[
        sync('bookstore-gui/src', '/home/app/src'),
        sync('bookstore-gui/public', '/home/app/public'),
        sync('bookstore-gui/package.json', '/home/app/package.json'),
        sync('bookstore-gui/package-lock.json', '/home/app/package-lock.json')
])
```

Montrer message de synchro d'1 fichier

## Optimisation JIB + FileSync (mvn)

branche tilt-step5

```python 
custom_build('k3d-registry:46697/api',
  'cd bookstore-api; mvn spring-boot:build-image -D image=$EXPECTED_REF',
  ['bookstore-api/pom.xml', 'bookstore-api/target/classes'],
  live_update = [
    sync('bookstore-api/target/classes', '/workspace/BOOT-INF/classes')
  ]
)
k8s_yaml(['bookstore-api/kubernetes/deployment.yml', 'bookstore-api/kubernetes/service.yml'])
k8s_resource('api-deployment', resource_deps=['pg-deployment'])
```

```xml
<properties>
    <image>k3d-registry:46697/api</image>
</properties>
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <excludeDevtools>false</excludeDevtools>
        <image>
            <name>${image}</name>
        </image>
    </configuration>
</plugin>
```
