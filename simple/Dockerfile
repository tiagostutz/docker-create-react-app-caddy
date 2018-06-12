FROM abiosoft/caddy:builder as builder 
ARG version="0.11.0" 
ARG plugins="git,filemanager,cors,realip,expires,cache" 
# process wrapper 
RUN go get -v github.com/abiosoft/parent
RUN VERSION=${version} PLUGINS=${plugins} ENABLE_TELEMETRY=false /bin/sh /usr/bin/builder.sh

FROM node:8

RUN npm install -g create-react-app
RUN mkdir -p /usr/src/app

ARG NODE_ENV
ENV NODE_ENV $NODE_ENV
RUN rm -rf build || true

# install caddy 
COPY --from=builder /install/caddy /usr/bin/caddy
# validate install 
RUN /usr/bin/caddy -version
RUN /usr/bin/caddy -plugins
EXPOSE 80 443 2015 

COPY Caddyfile /etc/Caddyfile
COPY --from=builder /go/bin/parent /bin/parent

WORKDIR /usr/src/app
RUN mkdir -p /srv

ONBUILD COPY ./package.json /usr/src/app
ONBUILD RUN npm install

ONBUILD COPY ./public /usr/src/app/public
ONBUILD COPY ./src /usr/src/app/src
ONBUILD RUN npm run build

ONBUILD RUN chmod -R 777 /usr/src/app/build/
ONBUILD RUN cp -r /usr/src/app/build/* /srv/

ENTRYPOINT ["/bin/parent", "caddy"]
CMD ["--conf", "/etc/Caddyfile", "--log", "stdout", "--agree=true"]
