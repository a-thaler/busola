FROM eu.gcr.io/kyma-project/test-infra/buildpack-node-chromium:v20201023-6c4eabe1 as builder
ARG app_name=core
WORKDIR /app

# Set env variables
ENV PRODUCTION true
ENV CI true

COPY . /app

RUN cd /app/${app_name} && \
  make resolve && \
  make verify && \
  make pull-licenses && \
  make build


### Main image ###
FROM eu.gcr.io/kyma-project/incubator/console-nginx:f4976fad

ARG app_name=core

COPY ./${app_name}/nginx/nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /app/${app_name}/src /usr/share/nginx/html/
COPY --from=builder /app/${app_name}/src/assets/luigi-core /usr/share/nginx/html/assets/luigi-core
COPY --from=builder /app/${app_name}/licenses/ /app/licenses/
COPY --from=builder /app/${app_name}/nginx/conf.d /etc/nginx/conf.d/
COPY --from=builder /app/${app_name}/nginx/nginx.conf /etc/nginx/
COPY --from=builder /app/${app_name}/nginx/mime.types /etc/nginx/

RUN touch /var/run/nginx.pid && \
  chown -R nginx:nginx /var/run/nginx.pid

EXPOSE 8080

ENTRYPOINT ["nginx", "-g", "daemon off;"]
