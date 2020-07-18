# Swagger Environment for API Design and Testing - Workflow

## Introduction

This document addresses the topic of creating and using an openapi-based API development and testing environment which uses complimentary Swagger Editor and Swagger Codegen Docker Containers. 

Commands shown in this document pull the required Docker Images from the docker hub and create containers that use them.

Once the environment is up and running, API development and testing can be undertaken as required, and containers stopped and remove once the task is over.

> Docker Images, which `docker run` command use  can be pulled from docker hub. Information on how these images were created and how the containers created using them can be found at:
>
> 1. [Swagger Editor](https://github.com/mwczapski/Swagger_Editor_3_Docker_Container)
> 2. [Swagger Codegen](https://github.com/mwczapski/Swagger_Codegen_3_Docker_Container)

## Swagger-based API Development and Test Environment

### Shared `openapi.yaml `and sharing between containers

#### Create Shared `/api` directory

``` shell
HOST_DIR=/mnt/d/github_materials
SHARED_API_HOST_DIR=${HOST_DIR}/shared_api
mkdir -pv ${SHARED_API_HOST_DIR}

```

#### Create starter __openapi.yaml__ file

``` shell
HOST_DIR=/mnt/d/github_materials
SHARED_API_HOST_DIR=${HOST_DIR}/shared_api

cat <<-'EOF' > ${SHARED_API_HOST_DIR}/openapi.yaml
openapi: "3.0.1"
info:
  title: Weather API
  description: |
    This API is a __test__ API for validation of local swagger editor
    and swagger ui deployment and configuration
  version: 1.0.0
servers:
  - url: 'http://localhost:3003/'
tags:
  - name: Weather
    description: Weather, and so on
paths:
  /weather:
    get:
      tags:
        - Weather
      description: |
        It is __Good__ to be a _King_
        And a *Queen*
        And a _Prince_
        And a __Princess__
        And all the Grandchildren
        And their Children
      operationId: getWeather
      responses:
        '200':
          description: 'All is _well_, but not quite'
          content: {}
        '500':
          description: Unexpected Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/response_500'
components:
  schemas:
    response_500:
      type: object
      properties:
        message:
          type: string

EOF
```

### Swagger Editor with access to shared openapi.yaml

#### Create Start Swagger Editor Container with access to shared `/api` directory  script

``` shell
HOST_DIR=/mnt/d/github_materials
SHARED_API_HOST_DIR=${HOST_DIR}/shared_api
cd ${HOST_DIR}/swagger_editor

SHARED_API_HOST_DIR_DOSISH=d:/github_materials/shared_api

HOST_LISTEN_PORT=3001
IMAGE_VERSION="1.0.0"
IMAGE_NAME="mwczapski/swagger_editor"
CONTAINER_NAME="swagger_editor"
CONTAINER_HOSTNAME="swagger_editor"
CONTAINER_VOLUME_MAPPING=" -v ${SHARED_API_HOST_DIR_DOSISH}:/api"
CONTAINER_MAPPED_PORTS=" -p 127.0.0.1:${HOST_LISTEN_PORT}:3001/tcp "

cat <<-EOF > start_swagger_editor_container_shared_dir.sh

docker.exe run \
    --name ${CONTAINER_NAME} \
    --hostname ${CONTAINER_HOSTNAME} \
    ${CONTAINER_VOLUME_MAPPING} \
    ${CONTAINER_MAPPED_PORTS} \
    --detach \
    --interactive \
    --tty\
        ${IMAGE_NAME}:${IMAGE_VERSION}

EOF

chmod u+x start_swagger_editor_container_shared_dir.sh

```

#### Start the Swagger Editor  container

``` shell
HOST_DIR=/mnt/d/github_materials
cd ${HOST_DIR}/swagger_editor
./start_swagger_editor_container_shared_dir.sh

```

#### View openapi.yaml in Swagger Editor on Host

[http://localhost:3001](http://localhost:3001)

OR:

Use `chrome`, which refreshes when asked with Sift^F5. `firefox` does not refresh when asked with that key sequence.

``` shell
HOST_DIR=/mnt/d/github_materials
cd ${HOST_DIR}/swagger_editor
cat <<-'EOF' >./swagger_editor_in_chrome.sh
#!/bin/bash
/mnt/c/Program\ Files\ \(x86\)/Google/Chrome/Application/chrome.exe -new-window http://localhost:3001 2>/dev/null &
EOF
chmod u+x ./swagger_editor_in_chrome.sh

./swagger_editor_in_chrome.sh

```

#### Edit shared __openapi.yaml__ with __VSCode__ and view with Swagger Editor

```shell
HOST_DIR=/mnt/d/github_materials
SHARED_API_HOST_DIR=${HOST_DIR}/shared_api
cd ${SHARED_API_HOST_DIR}
cat <<-'EOF' >./code_here.sh
#!/bin/bash
/mnt/c/Program\ Files/Microsoft\ VS\ Code/Code.exe . 2>&1 1>/dev/null & 
EOF
chamod u+x ./code_here.sh

./code_here.sh

```

### Swagger Codegen with access to shared openapi.yaml and with nodejs Stub Server

#### Create Start Swagger Codegen Container with access to shared `/api` directory  script

> Pleas note that this script mounts two shared directories, known to the container as `/api` and `/stubs_nodejs`.
>
> Any changes made to files in the corresponding Host directories will be visible to the container.

``` shell
HOST_DIR=/mnt/d/github_materials
HOST_DIR_DOSISH=d:/github_materials
SHARED_API_HOST_DIR=${HOST_DIR}/shared_api
cd ${HOST_DIR}/swagger_codegen

SHARED_API_HOST_DIR_DOSISH=${HOST_DIR_DOSISH}/shared_api
SOURCE_HOST_DIR=${HOST_DIR_DOSISH}/swagger_codegen

HOST_LISTEN_PORT=3003
IMAGE_VERSION="1.0.0"
IMAGE_NAME="mwczapski/swagger_codegen"
CONTAINER_NAME="swagger_codegen"
CONTAINER_HOSTNAME="swagger_codegen"
CONTAINER_VOLUME_MAPPING=" -v ${SHARED_API_HOST_DIR_DOSISH}:/api  -v ${SOURCE_HOST_DIR}/stubs_nodejs:/stubs_nodejs "
CONTAINER_MAPPED_PORTS=" -p 127.0.0.1:${HOST_LISTEN_PORT}:3003/tcp "

cat <<-EOF > start_swagger_codegen_container_shared_dir.sh

docker.exe run \
    --name ${CONTAINER_NAME} \
    --hostname ${CONTAINER_HOSTNAME} \
    ${CONTAINER_VOLUME_MAPPING} \
    ${CONTAINER_MAPPED_PORTS} \
    --detach \
    --interactive \
    --tty\
        ${IMAGE_NAME}:${IMAGE_VERSION}

EOF

chmod u+x start_swagger_codegen_container_shared_dir.sh

```

#### Start the Swagger Codegen container

``` shell
HOST_DIR=/mnt/d/github_materials
cd ${HOST_DIR}/swagger_codegen
./start_swagger_codegen_container_shared_dir.sh

```

#### Test Codegen

Edit the shared `openapi.yaml` file and watch stub server code generation

``` shell
docker container exec -it swagger_codegen tail -f /nohup.out
```

As soon as the `openapi.yaml` file changes the server stub will be re-generated and the stub server will be re-started ready for testing.

## Setup for Ordinary API Development Workflow

In the ordinary course of events a developer who uses the Swagger Editor and the Swagger Codegen containers mentioned in this document would employ the following workflow:

1. Create required Host directories
   1. Shared Host directory which containers will share and see as `/api`
   2. Shared Host directory which the Swagger Codegen will ass as `/stubs_nodejs` and which will contain the generated stub server nodejs code
2. Set up Swagger Editor environment
   1. Create a Host directory for Swagger Editor artefacts - startup scripts and suchlike
   2. Create startup scripts
   3. Start the Swagger Editor container
3. Set up Swagger Codegen environment
   1. Create a Host directory for Swagger Codegen artefacts - startup scripts and suchlike
   2. Create startup scripts
   3. Start the Swagger Codegen container
4. Test environment
   1. In a terminal run the docker command to watch the Swagger Codegen and Stub Server activity  
	`docker container exec -it swagger_codegen tail -f /nohup.out`
   2. On Host, use VSCode or a Text Editor to make a change to the `openapi.yaml` file, which the Host and both containers share, which will be visible in Swagger Editor
   3. Note activities in the terminal window (4.1)
   4. Start the Host Web Browser with the URL pointing to the Swagger Codegen /docs directory  
   In this setup `http://localhost:3003/docs`
   5. Test the API GET method using the Swagger Codegen Docs UI
   6. Force-refresh the windows to make sure that the latest `openapi.yaml` is used
   7. Start the Host Web Browser with the URL pointing to the Swagger Editor `/#` directory  In this setup `http://localhost:3001/#`
   8. Force-refresh the windows to make sure that the latest `openapi.yaml` is used
   9. Test the API GET method using the Swagger Editor Docs UI
   10. Make a visible change to the `openapi.yaml` in the Swagger Editor browser window
   11. Save the `openapi.yaml` file through the Swagger Editor Browser window to the Host directory shared between Host and both containers
   12. Note activities in the terminal window (4.1)
   13. Verify in Swagger Codegen Docs window that the change made in the Swagger Editor is visible in the Swagger Codegen Docs window

## Ordinary API Development Workflow

With  the environment up and running, tested and working, you can develop and test API specification using any tools that can change files in the shared Host directory  either form the Host or from the Container. 

I typically use VSCode to edit `openapi.yaml` form Windows, refresh Swagger Editor browser to see if I introduced any errors and test from there. Stub generation and execution is transparent. Each time the specification changes nodejs stub code is regenerated  and the stub server is restarted.

The Stub server code is basic. How basic depends on the sophistication of your `openapi.yaml` specification. For example, if the `openapi.yaml` specification includes a GET method that returns a JSON response,and the response stanza includes an example, JSON defined in the example will be returned. It will always be the same and the call will always succeed (if the stub was re-generated correctly and the stub server was re-started). 

To have the stub server return non-default responses stub code must be modified. Since the stub code is visible to the Host it is editable form the host and can be changes as much as desired. This topic may be covered in another article.

## Licensing

The MIT License (MIT)

Copyright  2020 Michael Czapski

Rights to Docker (and related), Git (and related), Debian, its packages and libraries, and 3rd party packages and libraries, belong to their respective owners.

2020/07 MCz