## Init

branche step1

### bookstore-pg

```bash
skaffold init --skip-build
skaffold run
```

### bookstore-api

```bash
skaffold init
skaffold debug
```

Ajouter fragment

```yaml
build:
  local:
    useBuildkit: true
```

### bookstore-gui

```bash
skaffold init
skaffold dev
```

Poser un breakpoint et vérifier que le mode debug fonctionne

```bash 
kubectl exec deploy/api-deployment -- sh -c "wget -O stdout  http://127.0.0.1:8090/books; cat stdout"
```

## Port Forward 

branche step2

### bookstore-pg 

```yaml 

portForward:
  - resourceType: deployment
    resourceName: pg-deployment
    port: 5432
    localPort: 5432
```

Faire démo client postgresql local

### bookstore-api 

```yaml 
portForward:
  - resourceType: deployment
    resourceName: api-deployment
    port: 8090
    localPort: 8090
  - resourceType: deployment
    resourceName: api-deployment
    port: 5005
    localPort: 5005
```

Test requête curl

```bash
curl localhost:8090/books
```

### bookstore-gui


```yaml 
portForward:
  - resourceType: deployment
    resourceName: gui-deployment
    port: 8080
    localPort: 8080
```

Démo IHM

## Orchestration

branche step3

Créer fichier à la racine

```yaml
apiVersion: skaffold/v2beta15
kind: Config

requires:
  - path: ./bookstore-pg
  - path: ./bookstore-api
  - path: ./bookstore-gui
```

Essayer de montrer erreur de démarrage (lié à la non gestion des dépendances)

## Optimisation FileSync (npm)

branche step4

```yaml
build:
  local:
    useBuildkit: true
  artifacts:
    - image: k3d-registry:46697/gui
      docker:
        dockerfile: Dockerfile
      sync:
        infer:
          - "public/**"
          - "src/**"
          - "package.json"
          - "package-lock.json"
```

Vérifier la synchro d'un fichier modifié

## Optimisation FileSync (mvn)

branche step5

```yaml
build:
  local:
    useBuildkit: true
  artifacts:
    - image: k3d-registry:46697/api
      docker:
        dockerfile: Dockerfile
      sync:
        infer:
          - "src/**"
          - "pom.xml"
```

Vérifier la synchro d'un fichier modifié

https://github.com/GoogleContainerTools/skaffold/issues/2357

## Optimisation JIB

branche step6

### Mvn
Ajouter fragment

```xml
    <profiles>
        <profile>
            <id>jib</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>com.google.cloud.tools</groupId>
                        <artifactId>jib-maven-plugin</artifactId>
                        <version>3.0.0</version>
                        <configuration>
                            <from>
                                <image>gcr.io/google-appengine/openjdk:8</image>
                            </from>
                            <container>
                                <jvmFlags>
                                    <jvmFlag>-Djava.security.egd=file:/dev/./urandom</jvmFlag>
                                </jvmFlags>
                            </container>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

```

### Skaffold


Corriger conf

Rajouter conf job

```xml
build:
  artifacts:
    - image: k3d-registry:46697/api
      jib:
        fromImage: amazoncorretto:16-alpine
      sync:
        auto: true

```

Vérifier la structure de l'image docker

