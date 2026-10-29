# unlucky-strike

A Django tech blog.

## Overview

This project shares stories of developers solving problems and learning.

Goals:
- Test new technologies
- Grow continuously
- Share knowledge

## Features

- Blog: Write and manage posts
- Gallery: Upload and display photos
- Home: Static pages (About, Contact)
- Projects: Showcase tech projects
- Backup: Automated database backup
- Deployment: Docker containers
- Server: nginx and uWSGI
- Security: SSL with Let's Encrypt

## Tech Stack

- Django
- SQLite
- Nginx
- Docker
- Bootstrap

## Architecture

Django app containerized with Docker.

## Let's Encrypt with AWS Route53

The production Compose file uses Certbot's Route53 DNS challenge. DNS-01
validation does not require port 80 to serve an ACME challenge, and it also
supports issuing a certificate for both the apex domain and `www`.

1. Create an AWS IAM user (or use an instance/task role) with only the
   permissions needed by the Route53 plugin:
   `route53:ListHostedZones`, `route53:GetChange`, and
   `route53:ChangeResourceRecordSets` for the hosted zone.
2. Copy `src/unluckystrike/.env.route53.example` to
   `src/unluckystrike/.env.route53` and fill in the credentials. Never commit
   this file.
3. From `src/`, build the Certbot image:

   ```sh
   docker compose build certbot
   ```

4. Issue the first certificate before starting Nginx:

   ```sh
   docker compose run --rm --no-deps certbot certonly \
     --dns-route53 \
     --email admin@example.com \
     --agree-tos \
     --no-eff-email \
     --non-interactive \
     -d unluckystrike.com \
     -d www.unluckystrike.com
   ```

   Replace the email address and domain names with the production values.
   The names must match `src/nginx/nginx.conf`.
5. Start the stack:

   ```sh
   docker compose up -d
   ```

The Certbot container checks for renewals every 12 hours. When a certificate
is renewed, it sends `SIGHUP` to the Nginx container through the Docker socket
so Nginx loads the new certificate without downtime. Because access to the
Docker socket grants control over Docker on the host, use this setup only on a
trusted host and protect the host and the Route53 credentials.

Renewal can be tested without changing the certificate using:

```sh
docker compose run --rm --no-deps certbot renew --dry-run
```