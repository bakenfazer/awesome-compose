# syntax=docker/dockerfile:1.4

FROM node:lts-buster-slim AS development

# harden file owner perms
RUN chown node:node /bin/mount /bin/umount /sbin/unix_chkpwd \
    /usr/bin/passwd /usr/bin/newgrp /usr/bin/gpasswd \
    /usr/bin/expiry /usr/bin/chfn /usr/bin/chage \
    /bin/su /usr/bin/chsh

RUN rm -R /var/local /var/mail

USER node

# Create app directory
WORKDIR /usr/src/app

COPY package.json /usr/src/app/package.json
COPY package-lock.json /usr/src/app/package-lock.json

RUN npm ci  

COPY config/config.json /usr/src/app/config.json
USER root
RUN rm /usr/local/lib/node_modules/npm/node_modules/tar/package.json 
COPY tar-package.json /usr/local/lib/node_modules/npm/node_modules/tar/package.json 
USER node

COPY . /usr/src/app

EXPOSE 3000

CMD [ "npm", "run", "dev" ]

FROM development as dev-envs
RUN <<EOF
apt-get update
apt-get install -y --no-install-recommends git
EOF

RUN <<EOF
useradd -s /bin/bash -m vscode
groupadd docker
usermod -aG docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /

HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
  
  # set user to node so as to not as non-root
USER node

CMD [ "npm", "run", "dev" ]