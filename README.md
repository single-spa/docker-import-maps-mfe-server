This repo is a work in progress. Documentation will be created once it's fully working.

# Initial setup

```sh
docker pull singlespa/import-maps-mfe-server
docker tag singlespa/import-maps-mfe-server $DOCKER_ORG_NAME/import-maps-mfe-server
docker push $DOCKER_ORG_NAME/import-maps-mfe-server
```

# Updating the image

```sh
export DOCKER_ORG_NAME=mycompany
export NPM_ORG_NAME=mycompany
export PROJECT_NAME=navbar
export PROJECT_UNIQUE_HASH_VERSION=$GIT_COMMIT_HASH

docker build -f ../docker-import-maps-mfe-server/import-maps-mfe/Dockerfile . -t $DOCKER_ORG_NAME/import-maps-mfe-server --build-arg libName=@$NPM_ORG_NAME/$PROJECT_NAME --build-arg libVersion=$PROJECT_UNIQUE_HASH_VERSION --build-arg baseImage=$DOCKER_ORG_NAME/import-maps-mfe-server
```