FROM docker.io/library/alpine:edge AS git
ARG TAG
ENV TAG=${TAG:-main}
RUN apk update && apk add --no-cache \
	git
WORKDIR /
RUN git -c advice.detachedHead=false clone --branch $TAG --depth=1 --recurse-submodules https://github.com/pfm-powerforme/certimate.git source
WORKDIR /source/
RUN echo "VITE_APP_VERSION=$TAG" > ./ui/.env


FROM docker.io/library/node:lts-alpine AS frontend
RUN apk update && apk add --no-cache \
	gcc musl-dev g++ make py3-pip
RUN corepack enable
WORKDIR /
COPY --from=git /source/ui/ /frontend/
WORKDIR /frontend/
RUN npm install && npm run build


FROM docker.io/library/golang:alpine AS backend
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux
WORKDIR /
COPY --from=git /source/ backend/
RUN rm -rf /backend/ui/dist
COPY --from=frontend /frontend/dist/ /backend/ui/dist
WORKDIR /backend/
RUN go build -ldflags="-w -s" -o certimate


FROM docker.io/library/alpine:latest AS runtime
WORKDIR /app/
COPY --from=backend /backend/certimate /app/certimate
RUN mkdir -p /app/pb_data
VOLUME ["/app/pb_data"]
EXPOSE 8090
ENTRYPOINT ["/app/certimate", "serve", "--http", "0.0.0.0:8090"]
