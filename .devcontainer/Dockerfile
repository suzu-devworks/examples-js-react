ARG VARIANT="20-bullseye"

# For develop-stage.
# > docker build --target develop-stage .
FROM node:${VARIANT} as develop-stage

## Configure apt and install packages
RUN set -x \
    && apt update \
    && apt install -y --no-install-recommends apt-utils dialog 2>&1 \
    && apt install -y procps iproute2 nano \
    ;

## Create a non-root user to use if preferred
ARG USERNAME=vscode
ARG USER_UID=2001
ARG USER_GID=$USER_UID
RUN set -x \
    && apt install -y sudo \
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    ;

# Clean up
RUN set -x \
    && apt autoremove -y \
    && apt clean -y \
    && rm -rf /var/lib/apt/lists/* \
    ;

# For build-stage.
# > docker build --target build-stage --file .devcontainer/Dockerfile .
# ### Please build in the parent folder. ###
FROM develop-stage  AS build-stage
WORKDIR /build
ARG APP=apps/examples-nuxt-quasar
COPY ${APP} /build

ARG BUILD="yarn generate"
RUN set -x \
    && yarn install \
    && ${BUILD}

# For production-stage.
# > docker build --file .devcontainer/Dockerfile .
# ### Please build in the parent folder. ###
FROM nginx:stable-alpine AS production-stage
#COPY .devcontainer/config/nginx/nginx.conf /etc/nginx/nginx.conf
#COPY .devcontainer/config/nginx/mime.conf /etc/nginx/conf.d/mime.conf
#COPY .devcontainer/config/nginx/default.conf /etc/nginx/conf.d/default.conf
#COPY --from=build-stage /build/dist /usr/share/nginx/html
COPY --from=build-stage /build/dist /app

EXPOSE 433
CMD [ "nginx", "-g", "daemon off;" ]
