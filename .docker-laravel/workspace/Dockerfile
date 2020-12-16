#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/Laradock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG LARADOCK_PHP_VERSION=8.0
FROM laradock/workspace:latest-${LARADOCK_PHP_VERSION}

ARG LARADOCK_PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

###########################################################################
# Laradock non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

ARG UBUNTU_SOURCE
COPY ./sources.sh /tmp/sources.sh

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} laradock && \
    useradd -u ${PUID} -g laradock -m laradock -G docker_env && \
    usermod -p "*" laradock -s /bin/bash && \
    apt-get install -yqq \
      apt-utils \
      #
      #--------------------------------------------------------------------------
      # Mandatory Software's Installation
      #--------------------------------------------------------------------------
      #
      # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
      # installed on the base image 'laradock/workspace' image. If you want
      # to add more Software's or remove existing one, you need to edit the
      # base image (https://github.com/Laradock/workspace).
      #
      # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
      libzip-dev zip unzip \
      # Install the zip extension
      php${LARADOCK_PHP_VERSION}-zip \
      # nasm
      nasm && \
      php -m | grep -q 'zip'

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/laradock/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/laradock/aliases.sh && \
    chown laradock:laradock /home/laradock/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

USER laradock

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

###########################################################################
# Non-root user : PHPUnit path
###########################################################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER laradock

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Crontab
###########################################################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d


###########################################################################
# xDebug:
###########################################################################

USER root

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y php${LARADOCK_PHP_VERSION}-xdebug && \
    sed -i 's/^;//g' /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-xdebug.ini \
;fi

# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini

###########################################################################
# Node / NVM:
###########################################################################

# Check if NVM needs to be installed
ARG NODE_VERSION=node
ENV NODE_VERSION node
ARG INSTALL_NODE=true
ARG INSTALL_NPM_GULP=false
ARG INSTALL_NPM_BOWER=false
ARG INSTALL_NPM_VUE_CLI=false
ARG INSTALL_NPM_ANGULAR_CLI=false
ENV NVM_DIR /home/laradock/.nvm

RUN if [ ${INSTALL_NODE} = true ]; then \
    # Install nvm (A Node Version Manager)
    mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash \
        && . $NVM_DIR/nvm.sh \
        && nvm install ${NODE_VERSION} \
        && nvm use ${NODE_VERSION} \
        && nvm alias ${NODE_VERSION} \
        && if [ ${NPM_REGISTRY} ]; then \
        npm config set registry ${NPM_REGISTRY} \
        ;fi \
        && if [ ${INSTALL_NPM_GULP} = true ]; then \
        npm install -g gulp \
        ;fi \
        && if [ ${INSTALL_NPM_BOWER} = true ]; then \
        npm install -g bower \
        ;fi \
        && if [ ${INSTALL_NPM_VUE_CLI} = true ]; then \
        npm install -g @vue/cli \
        ;fi \
        && if [ ${INSTALL_NPM_ANGULAR_CLI} = true ]; then \
        npm install -g @angular/cli \
        ;fi \
        && ln -s `npm bin --global` /home/laradock/.node-bin \
;fi

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

# Add NVM binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/laradock/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

# Add PATH for node
ENV PATH $PATH:/home/laradock/.node-bin

# Make it so the node modules can be executed with 'docker-compose exec'
# We'll create symbolic links into '/usr/local/bin'.
RUN if [ ${INSTALL_NODE} = true ]; then \
    find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
    NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
    ln -s $NODE_MODS_DIR/bower/bin/bower /usr/local/bin/bower && \
    ln -s $NODE_MODS_DIR/gulp/bin/gulp.js /usr/local/bin/gulp && \
    ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue /usr/local/bin/vue && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue-init /usr/local/bin/vue-init && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue-list /usr/local/bin/vue-list \
;fi

RUN if [ ${NPM_REGISTRY} ]; then \
    . ~/.bashrc && npm config set registry ${NPM_REGISTRY} \
;fi

###########################################################################
# MySQL Client:
###########################################################################

USER root

ARG INSTALL_MYSQL_CLIENT=true

RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
    apt-get update -yqq && \
    apt-get -y install mysql-client \
;fi

###########################################################################
# ping:
###########################################################################

USER root

ARG INSTALL_PING=true

RUN if [ ${INSTALL_PING} = true ]; then \
    apt-get update -yqq && \
    apt-get -y install inetutils-ping \
;fi

###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${LARADOCK_PHP_VERSION}."

###########################################################################
# Oh My ZSH!
###########################################################################

USER root

ARG SHELL_OH_MY_ZSH=true
RUN if [ ${SHELL_OH_MY_ZSH} = true ]; then \
    apt install -y zsh \
;fi

USER laradock
RUN if [ ${SHELL_OH_MY_ZSH} = true ]; then \
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh) --keep-zshrc" && \
    sed -i -r 's/^plugins=\(.*?\)$/plugins=(laravel5)/' /home/laradock/.zshrc && \
    echo '\n\
bindkey "^[OB" down-line-or-search\n\
bindkey "^[OC" forward-char\n\
bindkey "^[OD" backward-char\n\
bindkey "^[OF" end-of-line\n\
bindkey "^[OH" beginning-of-line\n\
bindkey "^[[1~" beginning-of-line\n\
bindkey "^[[3~" delete-char\n\
bindkey "^[[4~" end-of-line\n\
bindkey "^[[5~" up-line-or-history\n\
bindkey "^[[6~" down-line-or-history\n\
bindkey "^?" backward-delete-char\n' >> /home/laradock/.zshrc \
;fi

USER root

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www
