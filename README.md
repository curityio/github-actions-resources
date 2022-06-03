# github-actions-utilities

Utilities used in GitHub Actions workflows that test the Curity repositories. Some files are useful when running GitHub Actions workflows locally.

## Usage

Usually, the repository will be checked-out in a GitHub Actions workflow step:

```yml

- name: Checkout repository
  uses: actions/checkout@v3
  with:
    repository: curityio/github-actions-utilities
    path: utils

```

## Running the GitHub Actions Workflow Locally

The GitHub Actions workflow can be run locally using a tool called [act](https://github.com/nektos/act>). In order to run
the workflow you will need:

- Checkout this repository locally.
- Install the `act` tool.
- Copy the `secrets.template` file into `.secrets` file in the root folder. Usually you will find the template file in `tests/workflows` directory in the repository that you want to test. Set the correct values in the file.
- Build the docker container that is used to run the tests locally. You will find the Dockerfile in this repository in
  the `act` directory. From this directory run:

```bash
    docker build -t act-ubuntu-for-cypress .
```

- Run the workflow using the built container. From the root of the repository that you want to test run:

```bash
    act -P ubuntu-latest=act-ubuntu-for-cypress -b workflow_dispatch
```

The `-b` switch is used to mount the tested repository in the container instead of copying it. Thanks to that, test reports will be available in `tests/cypress/reports`.

### Running Workflows with act When Additional Services Used

Normally, the test workflows are run with the Curity Identity Server started in a docker container. When you run the
workflow locally with `act`, the container with the Curity Identity Server is actually started on your development machine,
not inside the `act` container. This means that if you start a service inside the `act` container, then the Curity 
Identity Server might have problems reaching that service. If this is the case, you might have to implement a workaround
to the workflow. You can start the Curity Identity Server directly in the `act` container, instead of its own docker.
To do that, follow these steps:

- Download the Curity Identity Server from https://developer.curity.io/downloads to and unpack it to a directory in the
  tested repository, e.g. `idsvr`.
- Copy a license file to `idsvr/etc/init/license/`.
- Copy the `tests/idsvr/config.xml` file to `idsvr/etc/init/`.
- If needed, adjust the config file to point to `localhost` instead of any virtual domains.
- Change the step in `.github/workflows/end-to-end-tests.yaml` that runs the Curity Identity Server container to this command:

```yaml
  - run: ./idsvr/bin/idsvr &
```
