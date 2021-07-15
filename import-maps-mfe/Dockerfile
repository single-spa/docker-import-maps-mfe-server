ARG baseImage=singlespa/import-maps-mfe-server

FROM ${baseImage}

ARG sourceDir=dist
ARG virtualDir=libs
ARG libName=unknown
ARG libVersion=unknown
ARG nginxDir=/usr/share/nginx/html

COPY ${sourceDir}/ ${nginxDir}/${virtualDir}/${libName}/${libVersion}/
