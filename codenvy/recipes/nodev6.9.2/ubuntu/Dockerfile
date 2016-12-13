FROM eclipse/ubuntu_jre

MAINTAINER mrtrom <ochoaseguralex@gmail.com>

ENV NODE_VERSION=6.9.2 \
    NVM_DIR=/usr/local/nvm \
    REDIS_VERSION=3.2.0 \
    REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-3.2.0.tar.gz \
    REDIS_DOWNLOAD_SHA1=0c1820931094369c8cc19fc1be62f598bc5961ca

RUN sudo rm /bin/sh && \
    sudo ln -s /bin/bash /bin/sh

# Install packages
RUN sudo apt-get update && \
    sudo apt-get -y install build-essential libssl-dev libkrb5-dev gcc make ruby-full rubygems-integration python && \
    sudo gem install sass compass && \
    sudo apt-get clean && \
    sudo apt-get -y autoremove && \
    sudo apt-get -y clean && \
    sudo rm -rf /var/lib/apt/lists/*

# Install NodeJs and global modules.
RUN sudo mkdir $NVM_DIR && \
    sudo chown -R $(whoami) $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash && \
    source $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    nvm use default && \
    npm install -g npm@latest && \
    npm install --unsafe-perm -g gulp gulp-cli bower grunt grunt-cli bunyan yo generator-angular-fullstack

# Install Mongo.
RUN \
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10 && \
  sudo touch /etc/apt/sources.list.d/mongodb.list && \
  echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee --append /etc/apt/sources.list.d/mongodb.list > /dev/null && \
  sudo apt-get update && \
  sudo apt-get install -y mongodb-org && \
  sudo rm -rf /var/lib/apt/lists/*

# Define mountable directories.
VOLUME ["/data"]
VOLUME ["/data/db"]

# Install Redis.
RUN buildDeps='gcc libc6-dev make' \
    && set -x \
    && sudo apt-get update && sudo apt-get install -y $buildDeps --no-install-recommends \
    && sudo rm -rf /var/lib/apt/lists/* \
    && sudo wget -O redis.tar.gz "$REDIS_DOWNLOAD_URL" \
    && echo "$REDIS_DOWNLOAD_SHA1 *redis.tar.gz" | sha1sum -c - \
    && sudo mkdir -p /usr/src/redis \
    && sudo tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && sudo rm redis.tar.gz \
    && sudo make -C /usr/src/redis \
    && sudo make -C /usr/src/redis install \
    && sudo rm -r /usr/src/redis \
    && sudo apt-get purge -y --auto-remove $buildDeps

COPY docker-entrypoint.sh /usr/local/bin/
RUN sudo chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]

ENV NODE_PATH=$NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH=$NVM_DIR/v$NODE_VERSION/bin:$PATH

# Expose ports.
EXPOSE 3000 5000 9000 27017 28017 6379

# Define working directory.
WORKDIR /projects

CMD tail -f /dev/null