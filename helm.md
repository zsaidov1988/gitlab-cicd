https://github.com/helm/examples/tree/main
```
helm package --version 1.1.0 .
curl --request POST --form 'chart=@microservice_v2-1.1.0.tgz' --user helm-user:token https://gitlab.zafarsaidov.uz/api/v4/projects/3/packages/helm/api/stable/charts

helm repo add --username helm-user --password token microservice https://gitlab.zafarsaidov.uz/api/v4/projects/3/packages/helm/stable

helm push microservice-1.1.1.tgz microservice_v2
```
