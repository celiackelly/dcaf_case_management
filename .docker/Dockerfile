FROM ruby:3.1.1-slim-buster
MAINTAINER Colin Fleming <c3flemin@gmail.com>

# configure environment variable
# note: move this to three ARG commands when CircleCI updates their docker
ENV DARIA_DIR=/usr/src/app \
    BUILD_DEPENDENCIES="build-essential libxml2-dev gnupg2 libxslt-dev fontconfig postgresql libpq-dev" \
    APP_DEPENDENCIES="git sudo sassc" \
    AHAB_DEPENDENCIES="ca-certificates curl" \
    FONTCONFIG_PATH=/etc/fonts \
    NODE_ENV=development \
    DOCKER=true

# get our gem house in order
RUN mkdir -p ${DARIA_DIR} && cd ${DCAF_DIR}
WORKDIR ${DARIA_DIR}
COPY Gemfile ${DARIA_DIR}/Gemfile
COPY Gemfile.lock ${DARIA_DIR}/Gemfile.lock
COPY package.json ${DARIA_DIR}/package.json
COPY yarn.lock ${DARIA_DIR}/yarn.lock

# install packages
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y \
    ${BUILD_DEPENDENCIES} \
    ${APP_DEPENDENCIES} \
    ${AHAB_DEPENDENCIES} && \
    gem install bundler --no-document

# get version 14 of nodejs
RUN curl -sL https://deb.nodesource.com/setup_14.x | sudo bash - && \
    apt-get install nodejs -y

RUN npm install -g yarn
RUN yarn set version berry
RUN yarn set version 2.x

# install gemfile and package
RUN bundle install
RUN yarn install

# remove unnecessary build dependencies
RUN apt-get purge -y ${BUILD_DEPENDENCIES} && apt-get autoremove -y 

# Install firefox for system tests
RUN apt-get install -y firefox-esr

# Check docker base image for vulnerable packages, ignore non zero exit code (just informative)
RUN mkdir /tmp/ahab && \
    cd /tmp/ahab && \
    curl -o ahab -O -L https://github.com/sonatype-nexus-community/ahab/releases/download/v0.2.3/ahab-linux.amd64-v0.2.3 && \ 
    chmod +x ahab && \
    update-ca-certificates && \
    dpkg-query --show --showformat='${Package} ${Version}\n' | ./ahab chase || true && \
    rm ahab && \
    cd -

# remove unnecessary ahab dependencies except ca-certificates, which is needed for nodejs
RUN ahab_dep=${AHAB_DEPENDENCIES} && \
    ahab_updated=${ahab_dep##ca-certificates} && \
    apt-get purge -y ${ahab_updated} && apt-get autoremove -y

# Move the rest of the app over
COPY . ${DARIA_DIR}

EXPOSE 3000
