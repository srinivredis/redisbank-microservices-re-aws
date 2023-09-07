## Redisbank test data generation app

## Building and publishing

1. Build the app locally using `./mvnw clean package`
1. Build a Docker image using `./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=<yourdockerhubid>/redisbank-dg`
1. Push Docker image using `docker push <yourdockerhubid>/redisbank-dg:latest`

# Deploying on EKS

1. Get the manifest files from k8s/apps folder.
1. Edit the `dg-manifest.yaml` file with the required env vars, or better yet, use k8s secrets. Also add your docker hub id.
1. Deploy the services using `kubectl apply -f dg-manifest.yaml`