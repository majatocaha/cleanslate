# Clean Slate

☀️ Track food without judgment.

## What is Clean Slate?

Clean Slate is a free calorie tracker. It is designed for people who struggle with:

- Binging
- Self-compassion
- Logging food consistently
- Dieting itself

It can do stuff like:

- Search and log food
- Quick add calories and protein
- Create custom foods and recipes
- Scan barcodes
- Track exercise

To learn more, visit [our website](https://cleanslate.sh) or [watch our demo video](https://youtu.be/wCoqpIImNdg).

## Do I need to host Clean Slate to use it?

Nope! We maintain a free instance at [cleanslate.sh](https://cleanslate.sh). It offers free accounts with social login (e.g. Login with Google).

## How is Clean Slate licensed?

Clean Slate is licensed under Apache 2.0 and is open source!

## How do I host Clean Slate?

Hosting Clean Slate is easy. You just need a Linux server with Git, Docker, and Docker Compose installed.

1.  Run `git clone https://github.com/successible/cleanslate` on your server. `cd` inside the newly created folder called `cleanslate`.

2.  Create a `.env` file in the `cleanslate` folder. Replace `<>` with your values.

```bash
POSTGRES_PASSWORD=<your-desired-password>
NEXT_PUBLIC_HASURA_DOMAIN=<your-server-domain>
HASURA_GRAPHQL_ADMIN_SECRET=<long-secret-value>
HASURA_GRAPHQL_CORS_DOMAIN=https://<your-server-domain>
# Read this file for an explanation of the value of HASURA_GRAPHQL_JWT_SECRET
# https://github.com/successible/cleanslate/blob/main/src/helpers/getJWT.ts
# Only change this value if you are using Firebase.
# It will look like something like this: '{ "type": "RS256", "audience": "<XXX>", "issuer": "https://securetoken.google.com/<XXX>", "jwk_url": "https://www.googleapis.com/service_accounts/v1/jwk/securetoken@system.gserviceaccount.com" }'
HASURA_GRAPHQL_JWT_SECRET='{ "type": "HS256", "key": "d374e7c8-912c-4871-bac2-7dde6afc2b55" }'

# Change if using Firebase
# You can find your project config in your Firebase project settings
# If it will look like '{"apiKey":"<XXX>","appId":"<XXX>","authDomain":"<XXX>","messagingSenderId":"<XXX>","projectId":"<XXX>","storageBucket":"<XXX>"}'
NEXT_PUBLIC_FIREBASE_CONFIG='{}'
NEXT_PUBLIC_LOGIN_WITH_APPLE='true'
NEXT_PUBLIC_LOGIN_WITH_FACEBOOK='true'
NEXT_PUBLIC_LOGIN_WITH_GITHUB='true'
NEXT_PUBLIC_LOGIN_WITH_GOOGLE='true'
NEXT_PUBLIC_REACT_SENTRY_DSN='' # Change to your DSN if you are using Sentry
NEXT_PUBLIC_USE_FIREBASE='false' # Change to true if using Firebase
```

4.  Run `bash deploy.sh`. This script will build and start the database, client, and server via Docker Compose. The client is what the user will interact with. It runs on `http://localhost:3000`. The server (Hasura) runs on port `8080`. The database (PostgreSQL) runs on `5432`.

> Note: Clean Slate uses the default `postgres` user and `postgres` database. It runs the database, Postgres 15, on port `5432` in a container managed by Docker Compose. If you do not like that behavior, you **must** create a custom `docker-compose.yml` and use that. You can do that by running export COMPOSE_FILE=custom.yml; bash deploy.sh

5.  On your domain, point a reverse proxy, like Caddy or Nginx, to `http://localhost:3000` and `http://localhost:8080` as outlined in this `Caddyfile`.

> Note: Clean Slate must be served over `https` to function. We recommend Caddy [^1] as the reverse proxy, and have tested Clean Slate with it. Caddy is great because it handles `https` automatically and for free via Let's Encrypt [^2].

    XXX/v1* {
        # API (Hasura)
        reverse_proxy localhost:8080
    }

    XXX/v2* {
        # API (Hasura)
        reverse_proxy localhost:8080
    }

    XXX/console* {
        # Admin panel (Hasura).
        reverse_proxy localhost:8080
    }

    XXX/healthz* {
        # Health check (Hasura).
        reverse_proxy localhost:8080
    }

    XXX {
        # Static files (Clean Slate)
        reverse_proxy localhost:3000
    }

6.  Go to the `https://<your-domain>/console`. Log in with your `HASURA_GRAPHQL_ADMIN_SECRET` defined in `.env`. Click `Data`, then `public`, then `profiles`, then `Insert Row`. On this screen, enter no input. Instead, click `Save`. This will create a new Profile. Click to `Browse Rows`. Take note of the `authId` of the row you just made. That is your (very long) credential to log in.

7.  You can now log in to `https://<your-domain>` with that credential.

8.  To deploy the newest version of Clean Slate, run `bash deploy.sh` again. Before you deploy, read `CHANGELOG.md`. We will list any breaking changes that have occurred.

### Handling authentication

Clean Slate was built around delegating authentication to Firebase. Firebase is a very secure authentication service maintained by Google. It is our default recommendation for any instance of Clean Slate with more than a few users. Consult the `Using Firebase` section of the appendix for how to set up Firebase with Clean Slate.

However, Firebase is too complex for the most common hosting scenario. That scenario is a privacy conscious user who wants to host Clean Slate for individual or family use. Hence, our default authentication system, `authId`, works differently.

The `authId` system is incredibly simple. There is no username or password. Clean Slate does not even require a server that can send email. Instead, Clean Slate uses very long tokens (uuid4) stored as plain text in the database. Because each token is very long and generated randomly, they are very secure. And if you ever need to change the value of the `authId`, you can just use the Hasura Console. If you would rather not use the `authId` system, you will need to use Firebase instead.

### Using Firebase

- Create a new Firebase project.
- Enable Firebase authentication.
- Enable the Google provider in Firebase.
- Create your `.firebaserc` locally in the root with the following content. Example:

```json
{
  "projects": {
    "default": "your-firebase-project-name"
  }
}
```

- Create a `firebase-config.json` locally filled with the content of `firebaseConfig`. You can find that on your Project Settings page on Firebase.

```json
{
  "apiKey": "XXX",
  "appId": "XXX",
  "authDomain": "XXX",
  "messagingSenderId": "XXX",
  "projectId": "XXX",
  "storageBucket": "XXX"
}
```

- Login with Firebase locally via `npx firebase login`.

- Run `npx firebase deploy` to deploy the Firebase functions in `/functions`.

You can now run Firebase locally or in production with these last steps.

Locally (Linux or Mac):

- Install [jq](https://stedolan.github.io/jq/download/)
- Mac only: Install and use GNU [sed](https://formulae.brew.sh/formula/gnu-sed)
- Run `export FIREBASE='true'; npm run dev`

Production (Linux):

- Change the values in the `.env` as outlined above.

## How to contribute to Clean Slate

Run Clean Slate locally, make changes, and then submit a pull request on GitHub.

> Note: Clean Slate is written in [React](https://reactjs.org) and [TypeScript](https://www.typescriptlang.org), with [Next.js](https://github.com/vercel/next.js) as the framework. It uses [Hasura](https://hasura.io) as the backend and [PostgreSQL](https://www.postgresql.org) as the database.

Here is how to run Clean Slate locally:

- Install [Git](https://git-scm.com/downloads), [Docker Desktop](https://www.docker.com/products/docker-desktop/), [Hasura CLI](https://hasura.io/docs/latest/hasura-cli/commands/hasura_console/), [Node.js (LTS)](https://nodejs.org/en/), and [Homebrew](https://brew.sh/). Make sure Docker Desktop is running.

- Run `npm run dev` after cloning down the repository. This will spin up these servers:

  - Hasura (API): `http://localhost:8080`.
  - Hasura (Console): `http://localhost:9695`.
  - Next.js: `http://localhost:3000`.
  - PostgreSQL: `http://localhost:1270`

- Navigate to `https://localhost` and login with token `22140ebd-0d06-46cd-8d44-aff5cb7e7101`.

> Note: To test the deployment process locally, install `caddy`. Then, run `bash deploy.sh` after creating the `.env` below in your `cleanslate` folder. Then, run `caddy start -c Caddyfile.dev`. `caddy` will pick up the `Caddyfile.dev` and serve Clean Slate on `https://localhost`.

> Note: If you want to proxy the development version of Clean Slate to test on a mobile device, install `ngrok`. Run `ngrok http --host-header localhost https://localhost:443` in another terminal.

```bash
# .env for testing the hosting process locally. Do not use in an actual production setting!
POSTGRES_PASSWORD=XXX
NEXT_PUBLIC_HASURA_DOMAIN=localhost
HASURA_GRAPHQL_ADMIN_SECRET=XXX
```

[^1]: https://caddyserver.com/docs/getting-started
[^2]: https://letsencrypt.org/
