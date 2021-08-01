ARG baseImage=singlespa/import-maps-mfe-server

FROM ${baseImage}

ARG sourceDir=dist
ARG virtualDir=libs
ARG libName=unknown
ARG libVersion=unknown
ARG libMainFile=unknown.js
ARG nginxDir=/usr/share/nginx/html
ARG publicOrigin=https://example.com
ARG importMapFilename=app.importmap

RUN apt-get update && apt install -y jq

COPY ${sourceDir}/ ${nginxDir}/${virtualDir}/${libName}/${libVersion}/

# Update import map with the module URL
RUN moduleName=${libName}; moduleUrl=${publicOrigin}/${virtualDir}/${libName}/${libVersion}/${libMainFile}; echo $(cat ${nginxDir}/${importMapFilename} | jq --arg moduleName "$moduleName" --arg moduleUrl "$moduleUrl" '.imports[$moduleName] = $moduleUrl') > ${nginxDir}/app.importmap

# Add "package as trailing slash" entry in import map
# https://github.com/WICG/import-maps#packages-via-trailing-slashes
RUN moduleName=${libName}/; moduleUrl=${publicOrigin}/${virtualDir}/${libName}/${libVersion}/; echo $(cat ${nginxDir}/${importMapFilename} | jq --arg moduleName "$moduleName" --arg moduleUrl "$moduleUrl" '.imports[$moduleName] = $moduleUrl') > ${nginxDir}/app.importmap