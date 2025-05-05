FROM node:12
LABEL maintainer="Ahmed Agiza"

WORKDIR /usr/src/app

COPY client/package*.json ./
RUN yarn install

COPY client ./

RUN git init && \
    git config user.email "docker@example.com" && \
    git config user.name "Docker Build" && \
    git add . && \
    git commit -m "Docker build commit"

ENV NODE_ENV production

RUN yarn run build

CMD [ "yarn", "start" ]
