# Tips for Testing

## Using a development branch

To use qhub from a development branch such as `main` set the
environment variable `QHUB_GH_BRANCH` before running qhub commands:

```
export QHUB_GH_BRANCH=main
```

Then `qhub init` will create a qhub-config.yaml containing, for
example, `quansight/qhub-jupyterlab:main` which is the Docker image
built based on the Dockerfiles specified in the main branch of the
qhub repo (see below for more info on how these are specified).  There
is a GitHub workflow that will build these images and push to Docker
Hub whenever a change is made to the relevant files on GitHub.

In addition, `qhub deploy` can use QHUB_GH_BRANCH to create
GitHub/GitLab workflows which install the development branch of qhub
for their own deploy steps.

If you want to use the development version of qhub for your init and
deploy but want your resulting deployment to be based on a full
release version, do not set the QHUB_GH_BRANCH environment
variable. In that case, Docker tags and workflow `pip install qhub`
commands will be based on the qhub version specified in the
qhub/version.py file, but these tags and releases may not yet exist,
perhaps if the version has been updated to include a beta/dev
component which has not been released.  So you may need to manually
modify the qhub-config.yaml to 'downgrade' the tags to a full release
version.

## Modifying Docker Images

All QHub docker images are located in [`qhub/templates/{{
cookiecutter.repo_directory
}}/image/`](https://github.com/Quansight/qhub-cloud/tree/main/qhub/template/%7B%7B%20cookiecutter.repo_directory%20%7D%7D/image). You
can build any image locally. Additionally, on Pull Requests each
Docker-build will be tested.

```shell
docker build -f Dockerfile.<filename> .
```

### Testing the JupyterLab Docker Image

Often times you would like to modify the jupyterlab default docker
image and run the resulting configuration.

```shell
docker run -p 8888:8888 -it <image-sha> jupyter lab --ip=0.0.0.0
```

Then open the localhost (127.0.0.1) link that is in the terminal

```
[I 2021-04-05 17:37:17.345 ServerApp] Jupyter Server 1.5.1 is running at:
...
[I 2021-04-05 17:37:17.346 ServerApp]  or http://127.0.0.1:8888/lab?token=8dbb7ff1dcabc5fab860996b6622ac24dc71d1efc34fcbed
...
[I 2021-04-05 17:37:17.346 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

## Debug Kubernetes clusters

 To debug Kubernetes clusters, we advise you to use [K9s](https://k9scli.io/), a terminal-based UI that aims to
 simplify navigation, observation, and management of applications in Kubernetes.
 K9s continuously monitors Kubernetes clusters for changes and provides
 shortcut commands to interact with the observed resources becoming a
 fast way to review and resolve day-to-day issues in deployed clusters.

Installation can be done on a macOS, in Windows, and Linux and instructions
 can be found [here](https://github.com/derailed/k9s). For more details on usage,
check out the [Troubleshooting documentation](../02_get_started/06_troubleshooting.md#debug-your-kubernetes-cluster)


## Cypress Tests

Cypress automates testing within a web browser environment. It is integrated into the GitHub Actions tests.yaml workflows in this repo, and 
you can also run it locally. To do so:

```
cd tests_e2e
npm install

export CYPRESS_BASE_URL=http://127.0.0.1:8000/
export QHUB_CONFIG_PATH=/Users/dan/qhub/data-mk/qhub-config.yaml
export CYPRESS_EXAMPLE_USER_PASSWORD=<password>

npm run cypress:open
```

The Base URL can point anywhere that should be accessible - it can be the URL of a QHub cloud deployment.
The QHub Config Path should point to the associated yaml file for that site. Most importantly, the tests will inspect the yaml file to understand 
what tests are relevant. To start with, it checks security.authentication.type to determine what should be available on the login page, and 
how to test it. If the login type is 'password' then it uses the value in CYPRESS_EXAMPLE_USER_PASSWORD as the password (default username is 
`example-user` but this can be changed by setting CYPRESS_EXAMPLE_USER_NAME).

The final command above should open the Cypress UI where you can run the tests manually and see the actions in the browser.

Note that tests are heavily state dependent, so any changes or use of the deployed QHub could affect the results.

# Cloud Testing

Cloud testing on aws, gcp, azure, and digital ocean can be significantly more complicated and time consuming. But it is the only way to truly test the cloud deployments, including infrastructure, of course. To test on cloud Kubernetes, just deploy qhub in the normal way on those clouds, but using the [linked pip install](./index.md) of the qhub package.

Even with the dev install of the qhub package, you may find that the deployed cluster doesn't actually reflect any development changes, e.g. to the Docker images for JupyterHub or JupyterLab. That will be because your qhub-config.yaml references fully released versions. See [Using a development branch](#using-a-development-branch) above for how to encourage the Docker images to be specified based on the latest development code.

You should always prefer the local testing when possible as it will be easier to debug, may be quicker to deploy, and is likely to be less expensive.