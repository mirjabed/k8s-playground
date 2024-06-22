## Build the password generator app
```bash
export docker_registry_name=ahsanemon # <- replace this with your own registry name
docker build -t ${docker_registry_name}/password-generator .
docker push ${docker_registry_name}/password-generator
```

## Run the app
Replace the image registry if you have used your own or run the following command
```bash
kubectl apply -f https://raw.githubusercontent.com/ahsanemon/k8s-playground/main/password-generator/k8s-manifests/password-generator.yaml
```