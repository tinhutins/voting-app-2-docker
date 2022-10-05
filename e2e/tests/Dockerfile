FROM node:8.16.0-jessie-slim

RUN apt-get install -qy ca-certificates curl libfontconfig1 --no-install-recommends
RUN yarn global add phantomjs-prebuilt
ADD . /app
WORKDIR /app
CMD ./tests.sh
