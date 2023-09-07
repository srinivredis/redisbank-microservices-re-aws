## Redisbank pfm app

## Building and publishing

1. Build the app locally using `./mvnw clean package`
1. Build a Docker image using `./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=<yourdockerhubid>/redisbank-pfm`
1. Push Docker image using `docker push <yourdockerhubid>/redisbank-pfm:latest`

# Deploying on EKS

1. Navigate to the [k8s](k8s) folder.
1. Edit the `pfm-manifest.yaml` file with the required env vars, or better yet, use k8s secrets. Also add your docker hub id.
1. Deploy the services using `kubectl apply -f pfm-manifest.yaml`