# post-deploy-script

inspired by the [dokku-post-deploy-script plugin](https://github.com/baikunz/dokku-post-deploy-script) by @baikunz.

---

Dokku plugin to execute scripts on the dokku host (not within a container!) after a deploy. In contrast to the referenced plugin from @baikunz, the post deploy script is expected to be in the deployed image (in the projects root) rather than on the dokku host only.

This can be useful if you have to e.g. build an additional docker image after each deploy (to run afterwards by jupyterhub.).

## requirements

- dokku 0.4.0+

## installation

```shell
# on 0.4.x
dokku plugin:install https://github.com/lebalz/dokku-post-deploy-script.git post-deploy-script
```

## hooks

This plugin provides hooks:

- `post-deploy`: Execute script on dokku host after deploy

## Commands

```sh
post-deploy-script:show    <app>     Shows the content of the POST_DEPLOY_SCRIPT
post-deploy-script:report  <app>     Display the configuration for an app
post-deploy-script:disable <app>     disables the execution of the post deploy script for an app
post-deploy-script:enable  <app>     (re)enables the execution of the post deploy script for an app (if present)
```

## Usage

This plugin allows you to execute a script on your dokku host as root (not inside the docker container).

The file must be in your projects root and must be named `POST_DEPLOY_SCRIPT`.

For images built via Dockerfile, make sure to copy the `POST_DEPLOY_SCRIPT` over to the `WORKDIR` during the build phase.

If your script needs other files to run, make sure to specify these files as `DOKKU_POST_DEPLOY_SCRIPT_DEPENDENCIES`, separated by `;`. E.g.

```sh
dokku config:set --no-restart node-js-app DOKKU_POST_DEPLOY_SCRIPT_DEPENDENCIES="images/Dockerfile;another-file"
```

Will copy the files `images/Dockerfile` to `/home/dokku/node-js-app/post-deploy-scripts/images/Dockerfile` and `another-file` to `/home/dokku/node-js-app/post-deploy-scripts/another-file`.

### Example to build jupyter notebook image for jupyterhub after deploy

(fyi: at [dokku-jupyterhub](https://github.com/lebalz/dokku-jupyterhub) a ready to use repo can be found)

```sh
dokku-jupyterhub
├── app
│   ├── [...]
├── jupyterlab-images
│   └── Dockerfile
├── Dockerfile
├── POST_DEPLOY_SCRIPT
└── README.md
```

```sh
# set the image jupyterhub should start for each user
dokku config:set --no-restart dokku-jupyterhub DOCKER_JUPYTER_IMAGE="custom-build:latest"

# make sure the Dockerfile is copied to the dokku host too
dokku config:set --no-restart dokku-jupyterhub DOKKU_POST_DEPLOY_SCRIPT_DEPENDENCIES="jupyterlab-images/Dockerfile"
```

and in your projects root add a `POST_DEPLOY_SCRIPT`:

```sh
#!/bin/bash
TAG=$(dokku config:get $APP DOCKER_JUPYTER_IMAGE)
# build the image
(cd jupyterlab-images && docker build . -t $TAG)
```
