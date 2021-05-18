## K3d

### Cleanup

```bash
k3d cluster delete k3s-default
k3d registry delete k3d-registry
```

### Run with registry 
```bash 
k3d registry create registry --port 0.0.0.0:46697
k3d cluster create --registry-use k3d-registry:46697
```

### Config 

```
k3d kubeconfig get k3s-default > ~/.kube/config
```