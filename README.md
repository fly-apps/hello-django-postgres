## HelloDjangoPostgres!

This is an example project demonstrating how to deploy a Django application with Postgres on Fly.io.

### Local Development

The environment variables are stored in the `.env` file.

Duplicate `.env.sample` file and rename it to `.env`. Update the environment variables:
```
SECRET_KEY=<your-generated-django-secure-super-secret-key>
DEBUG=True
DATABASE_URL=postgres://<user>:<password>@<host>:<port>/<db>
```

> The default `DATABASE_URL` is `postgres://postgres:postgres@localhost:5432/hello_django` (check `hello_django/settings.py`).

---
## Getting started

[flyctl](https://fly.io/docs/hands-on/install-flyctl/) is the command-line utility provided by Fly.io.

If you still don't have installed it, you can follow the [instructions here](https://fly.io/docs/hands-on/install-flyctl/) to install it, [sign up](https://fly.io/docs/hands-on/sign-up/) and [log in](https://fly.io/docs/hands-on/sign-in/) to Fly.io.

---
## Create and configure a new app (`fly launch`)

This simple app already contains all the basic configuration for deploying to Fly.io.
- `Dockerfile` contain commands to build the image.
- `.dockerignore` list of files or directories Docker will ignore during the build process.
- `fly.toml` configuration for deployment on Fly.io.

---
#### Dockerfile

`SECRET_KEY` is required when running `collectstatic`. The `SECRET_KEY` is set on `Dockerfile` for building purposes:
```Dockerfile
ENV SECRET_KEY "non-secret-key-for-building-purposes"  # <-- Set SECRET_KEY for building purposes
RUN python manage.py collectstatic --noinput
```

Instead, an alternative is to collect the static files _manually_ after the deployment by doing:
```shell
❯ fly ssh console
# cd code
# python manage.py collectstatic --noinput
```

---
When running `fly launch`, copy the existing configuration to your own app:

```shell
❯ fly launch
An existing fly.toml file was found for app hello-django-postgres
? Would you like to copy its configuration to the new app? Yes
```

### Secrets

During the `fly launch`, the necessary secrets will be set:
- `SECRET_KEY`
- `DATABASE_URL`

## Django `SECRET_KEY`

```shell
❯ fly launch
Creating app in /Projects/flyio/hello-django-postgres
Scanning source code
Detected a Django app
? Choose an app name (leave blank to generate one): hello-django-postgres
...
Hostname: hello-django-postgres.fly.dev
Set secrets on hello-django-postgres: SECRET_KEY  # <-- SECRET_KEY is set automatically
```

## Postgres `DATABASE_URL`

Set up a Postgres database during the `fly launch` step:
```shell
❯ fly launch
Creating app in /Projects/flyio/hello-django-postgres
Scanning source code
Detected a Django app
? Choose an app name (leave blank to generate one): hello-django-postgres
...
? Would you like to set up a Postgresql database now? Yes  # <-- You will be asked HERE
? Select configuration: Development - Single node, 1x shared CPU, 256MB RAM, 1GB disk
Creating postgres cluster in organization fly-io
Creating app...
Setting secrets on app hello-django-postgres-db...
Provisioning 1 of 1 machines with image flyio/postgres:14.6
Waiting for machine to start...
....
Postgres cluster hello-django-postgres-db is now attached to hello-django-postgres
The following secret was added to hello-django-postgres:
  DATABASE_URL=postgres://hello_django_postgres:****@****:5432/hello_django_postgres?sslmode=disable
Postgres cluster hello-django-postgres-db is now attached to hello-django-postgres  # <-- postgres cluster attached to the app
```

> Important: During this step, save your credentials in a secure place - you won't be able to see them again!

Your postgres cluster will be automatically attached to your app.

## `ALLOWED_HOSTS` and `CSRF_TRUSTED_ORIGINS`

Before we can deploy it, make sure to update `ALLOWED_HOSTS` and `CSRF_TRUSTED_ORIGINS` in `hello_django/settings.py` with the chosen/generated app name on Fly.io. 

```python
ALLOWED_HOSTS = ['127.0.0.1', 'localhost', '<your-app-name>.fly.dev']
CSRF_TRUSTED_ORIGINS = ['https://<your-app-name>.fly.dev']
```

Your Django app is now ready to be deployed!

---
## Deploy (`fly deploy`)

Once all the previous steps are done, you can deploy your app:
```shell
❯ fly deploy
==> Verifying app config
--> Verified app config
==> Building image
...
 1 desired, 1 placed, 1 healthy, 0 unhealthy [health checks: 1 total, 1 passing]
--> v0 deployed successfully
```

Your app is now up and running!

```shell
❯ fly open
```