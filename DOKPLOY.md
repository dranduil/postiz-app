# Dokploy deployment

This repository's `docker-compose.yaml` is prepared for a Dokploy Compose application. It runs the Postiz image and Temporal stack, while the application uses the shared PostgreSQL and Redis containers already managed by Dokploy.

## Dokploy application settings

1. Create a Compose application from this repository and use `docker-compose.yaml`.
2. Make sure the shared PostgreSQL and Redis containers are attached to the external Docker network named `dokploy-network`.
3. Add the environment variables below in Dokploy. Do not commit a `.env` file or credentials.
4. Configure the Dokploy domain to target service `postiz` on container port `5000`. Do not publish a host port.
5. Deploy and wait for the `postiz` health check to become healthy before testing the domain.

If the shared network has another name, set `DOKPLOY_NETWORK_NAME` to that exact Docker network name.

For repeatable releases, set `POSTIZ_IMAGE` to a reviewed image tag or digest instead of using the default `latest` tag.

## Required environment variables

```dotenv
MAIN_URL=https://marketing.example.com
FRONTEND_URL=https://marketing.example.com
NEXT_PUBLIC_BACKEND_URL=https://marketing.example.com/api
JWT_SECRET=<long-random-secret>
DATABASE_URL=postgresql://<user>:<password>@<shared-postgres-container>:5432/<database>
REDIS_URL=redis://<shared-redis-container>:6379
```

Use the shared container's Docker DNS name, not `localhost`, for `DATABASE_URL` and `REDIS_URL`. URL-encode any special characters in the PostgreSQL username or password. The PostgreSQL database must already exist; the application runs its existing Prisma startup step when the container starts.

`TEMPORAL_CORS_ORIGINS` is optional. Set it to the public frontend URL only if you expose the internal Temporal UI; otherwise Compose uses a local-only fallback.

The default values in Compose use local uploads and enable registration. Set the corresponding environment variables in Dokploy for production storage, registration policy, social providers, email, OAuth, AI, billing, or object storage.

## Verification

After deployment, verify all of the following in Dokploy and from the public HTTPS domain:

- `postiz` is healthy and listening on port `5000`.
- The public domain loads without a 502/504 response.
- The application logs show successful PostgreSQL, Redis, and Temporal connectivity.
- Login and one authenticated API request work through `/api`.
- Uploaded media survives a container recreation when persistent storage is enabled.

The Temporal PostgreSQL and Elasticsearch services in this Compose file are internal to the Temporal stack; they are separate from the shared application PostgreSQL and Redis services.
