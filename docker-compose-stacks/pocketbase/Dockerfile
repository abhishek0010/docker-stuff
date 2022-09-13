FROM alpine:3 as downloader

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=0.7.2

ENV BUILDX_ARCH="${TARGETOS:-linux}_${TARGETARCH:-arm64}"

RUN wget https://github.com/pocketbase/pocketbase/releases/download/v${VERSION}/pocketbase_${VERSION}_${BUILDX_ARCH}.zip \
	&& unzip pocketbase_${VERSION}_${BUILDX_ARCH}.zip \
	&& chmod +x /pocketbase

FROM alpine:3

# Dependencies
RUN apk update && apk add ca-certificates && rm -rf /var/cache/apk/*

EXPOSE 8090

COPY --from=downloader /pocketbase /usr/local/bin/pocketbase
CMD ["/usr/local/bin/pocketbase", "serve", "--http=0.0.0.0:8090", "--dir=/pb_data"]

