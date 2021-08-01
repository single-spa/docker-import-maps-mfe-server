# docker-import-maps-mfe-server

Docker image, utilities, and examples for docker container hosting of frontend assets for microfrontends with import maps.

## When to use this

This repository exists for single-spa users who wish to host their microfrontends and import map in a docker container. Generally, it is simplest and best for performance (since cloud object stores have global edge locations and availability) to host static frontend assets in a cloud object store, such as AWS S3, GCP Storage, Azure Storage, or Digital Ocean Spaces, rather than in a Docker container. However, for some organizations, a docker container is required or preferred, and docker-import-maps-mfe-server exists for that situation.

## Overview

This project hosts all microfrontends in a single docker container, along with an import map used by the browser to discover the microfrontends and the latest version of them. It is intended for use in deployed environments, not for local development. It is intended to integrate with your CI/CD processes to facilitate building and deploying of microfrontends.

It is best for the deployment of a microfrontend to include uploading new static frontend assets (JS, CSS, HTML) to a server, rather than modifying existing files. This is important for performance, as a server can instruct browsers and CDNs to cache versioned files forever, but cannot do the same for files that are modified in place. Additionally, this is important for the consistency of the system during deployments: users who lazy load assets after a deployment has occurred should receive the version of the assets that is guaranteed to work with the assets they already have downloaded. In other words, the versions of frontend assets should be locked at the beginning of a page load, and the user should always receive those versions of files until a page reload occurs.

The one exception to this rule is the import map file itself - that file should be modified in-place on the server, and CDNs and browsers should not cache the file for very long. This is because the import map serves as a manifest file that tracks the currently deployed versions of all microfrontends.

The server must continue hosting old versions of static assets even after a deployment, since users will continue requesting old versions of the files until they reload the page and/or their browser cache busts for the import map file.

With this in mind, a microfrontend server might host files with the following structure:

```sh
https://example.com/app.importmap # Manifest of which version of each MFE is "live"
https://example.com/libs/@org-name/navbar/v1.0.0/navbar.js
https://example.com/libs/@org-name/navbar/v1.1.0/navbar.js
https://example.com/libs/@org-name/navbar/v1.2.0/navbar.js
https://example.com/libs/@org-name/navbar/v1.3.0/navbar.js
https://example.com/libs/@org-name/settings/v1.0.0/settings.js
https://example.com/libs/@org-name/settings/v1.1.0/settings.js
https://example.com/libs/@org-name/settings/v1.2.0/settings.js
```

The `import-maps-mfe-server` image within this project is used as a base image that can be used to host the files shown above for example.com. The `import-maps-mfe` Dockerfile within this project is used to help in deploying new versions of that image.

## Initial Setup

The first step is to create an organization-specific docker image, based on the singlespa/import-maps-mfe-server image.

```sh
export DOCKER_ORG_NAME=example
docker pull singlespa/import-maps-mfe-server
docker tag singlespa/import-maps-mfe-server $DOCKER_ORG_NAME/import-maps-mfe-server
docker push $DOCKER_ORG_NAME/import-maps-mfe-server
```

This will create an image with an `app.importmap` file that is an empty import map.

If you wish to change the file name, you can build your own version of the image. The `.importmap` extension is encouraged, since many web servers only send the correct `Content-Type` HTTP response header for import maps (`application/importmap+json`) for files with the `.importmap` extension.

```sh
curl https://raw.githubusercontent.com/single-spa/docker-import-maps-mfe-server/main/import-maps-mfe/Dockerfile | docker build --build-arg importMapFilename=main.importmap https://github.com/single-spa/docker-import-maps-mfe-server.git#main:import-maps-mfe-server -t $DOCKER_ORG_NAME/import-maps-mfe-server
```

## Testing Locally

To run a container locally to verify that your docker image is working as expected:

```sh
docker run -d -it --name local-mfe-server -p 7400:80 $DOCKER_ORG_NAME/import-maps-mfe-server

# The following commands uses HTTPie, which is an easier-to-use alternative to curl. You can use curl, if you prefer.
# https://httpie.io/

# View the import map
http :7400/app.importmap

# View a specific microfrontend file. You'll need to modify this to match your file names and version numbers
http :7400/libs/navbar/v1.0.0/navbar.js
```

## Build / Deploy a Microfrontend

To deploy a microfrontend to a production environment, the CI/CD process for that microfrontend should do the following:

### Build steps

1. Build the frontend static assets to the `dist` directory within the CI runner. This often involves `npm install && npm run build`
2. Store the `dist` directory as a CI/CD artifact, to be used later during deployments.

### Deploy steps

1. Retrieve the `dist` artifact from the build step
2. Pull down the latest version of your docker image and modify it to contain the new version of the microfrontend, along with updating the import map to point to that new version:

```sh
# Select version of import-maps-mfe to use when building. "main" refers to the main Git branch, which means you'll automatically use the latest code. It's recommended to lock to a specific version of import-maps-mfe to avoid unintended upgrades. To do that, use "v1.0.0" (replace with the version you want to use)
export DOCKERFILE_VERSION=main

# Push vars. Each deployed environment has its own docker tag
export ENVIRONMENT_NAME=prod
export DOCKER_ORG_NAME=mycompany

# Helper variable for the command below
export DOCKER_ORG_NAME=orgname

# $PROJECT_UNIQUE_HASH_VERSION should be replaced with env vars provided by your CI/CD tool that uniquely identify this build. For example, CI_COMMIT_HASH can be used in many CI/CD tools.

# This command builds a docker image. See docker build args section below for details
curl https://raw.githubusercontent.com/single-spa/docker-import-maps-mfe-server/$DOCKERFILE_VERSION/import-maps-mfe/Dockerfile | docker build -f - . -t $DOCKER_ORG_NAME/import-maps-mfe-server:$ENVIRONMENT_NAME --build-arg libName=@org-name/navbar --build-arg libVersion=$PROJECT_UNIQUE_HASH_VERSION --build-arg baseImage=$DOCKER_ORG_NAME/import-maps-mfe-server
```

3. Push the docker image with the correct tag

```sh
docker push $DOCKER_ORG_NAME/import-maps-mfe-server:$ENVIRONMENT_NAME
```
4. Deploy a container with the newly pushed image. Docker Swarm, Kubernetes, AWS ECS, etc can be used for this.

### Docker build args

The following [Docker build args](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg) are available when using the `import-maps-mfe` Dockerfile

They can be added to the docker build command above.

#### libName

```sh
--build-arg libName=@org-name/navbar
```

Required. This is the name of your microfrontend in the import map, which is translated into directories in the URL for loading the microfrontend's main javascript file.

For example, `libName=@org-name/navbar` means that all files will be uploaded under the directory https://example.com/libs/@org-name/navbar/

#### libVersion

```sh
--build-arg libVersion=v1.0.0
```

Required. This is the version of the microfrontend that's being deployed. It should be unique to this particular Git commit / CI build.

A common way to implement this is to use an environment variable provided by your CI/CD tool. For example, `$CI_COMMIT_HASH` is a common environment variable available in some CI/CD tools, representing the unique Git hash for any particular git commit.

#### libMainFile

```sh
--build-arg libMainFile=navbar.js
```
Required. This is the filename of the main javascript file for this microfrontend. This will be part of the import map entry for the microfrontend.

#### importMapFilename

```sh
--build-arg importMapFilename=app.importmap
```

Optional, defaults to `app.importmap`. This is the name of the import map file that will be accessible at https://example.com/app.importmap. The `.importmap` extension is encouraged, since many web servers only send the correct `Content-Type` HTTP response header for import maps (`application/importmap+json`) for files with the `.importmap` extension.

#### sourceDir

```sh
--build-arg sourceDir=dist
```

Optional, defaults to `dist`. This is the local directory (often the directory within the CI runner) that will be copied into the docker image. This directory should contain all frontend assets for this version of the microfrontend (JS, HTML, CSS, etc)

#### virtualDir

```sh
--build-arg virtualDir=libs
```

Optional, defaults to `libs`. This is a prefix put before all microfrontend static assets, as a directory in the URL.

#### nginxDir

```sh
--build-arg nginxDir=/usr/share/nginx/html
```

Optional, defaults to `/usr/share/nginx/html`. This is the base directory that the files copied from `sourceDir` will be copied to. If not using a debian image, this might need to be altered.