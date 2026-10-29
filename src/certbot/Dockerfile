FROM certbot/certbot:arm32v6-latest

RUN apk add --no-cache docker-cli \
    && pip install --no-cache-dir certbot-dns-route53
