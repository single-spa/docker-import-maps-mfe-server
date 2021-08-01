ARG baseImage=singlespa/import-maps-mfe-server

FROM ${baseImage}

ARG sourceDir=dist
ARG virtualDir=libs
ARG libName=unknown
ARG libVersion=unknown
ARG nginxDir=/usr/share/nginx/html
ARG moduleName=sample-name-do-not-use
ARG moduleUrl=https://example.com/do-not-use.js
ARG importMapFilename=app.importmap

RUN apt-get update && apt install -y jq

COPY ${sourceDir}/ ${nginxDir}/${virtualDir}/${libName}/${libVersion}/

# Update import map with the module URL
RUN moduleName=${moduleName}; moduleUrl=${moduleUrl}; echo $(cat ${nginxDir}/${importMapFilename} | jq --arg moduleName "$moduleName" --arg moduleUrl "$moduleUrl" '.imports[$moduleName] = $moduleUrl') > ${nginxDir}/app.importmap

# Add "package as trailing slash" entry in import map
# https://github.com/WICG/import-maps#packages-via-trailing-slashes
RUN moduleName=${moduleName}/; moduleUrl=$(echo ${moduleUrl} | sed 's/[^/]*$//'); echo $(cat ${nginxDir}/${importMapFilename} | jq --arg moduleName "$moduleName" --arg moduleUrl "$moduleUrl" '.imports[$moduleName] = $moduleUrl') > ${nginxDir}/app.importmap