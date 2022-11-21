# redbuild

super easy containerized builds

## overview

redbuild is a super simple drop-in script enabling software to be built in pre-defined containers. with `podman` installed, any supported project can be built with a single command. projects provide a `build.docker` and `build.sh` for defining the build environment and build steps.

`redbuild.sh`: just add water!

## try the example!

```sh
cd example
../redbuild.sh
```

## detailed usage

### creating the build environment and build script

1. create a `build.docker` file in the project root. this file should contain a `FROM` directive for the base image to use for the build environment. the build environment should contain all the tools necessary to build the project.

    `build.docker`:

    ```dockerfile
    FROM debian:buster-slim

    # install dependencies
    RUN apt-get update && apt-get install -y \
    bash \
    curl wget xz-utils \
    gcc make libc6-dev libcurl4 \
    git libxml2 \
    && rm -rf /var/lib/apt/lists/* && apt autoremove -y && apt clean


    # install dlang
    RUN curl -fsS https://dlang.org/install.sh | bash -s install ldc-1.30.0 \
    && echo "source ~/dlang/ldc-1.30.0/activate" >> ~/.bashrc

    # set up main to run bash
    CMD ["/bin/bash", "-l"]
    ```

2. create a `build.sh` file in the project root. this file should contain the steps necessary to build the project. the build script should be written to be run in the build environment.

    `build.sh`:

    ```sh
    #!/bin/bash
    dub build --compiler ldc2 -B release
    ```

that's it! now you can build the project with `redbuild.sh`, copy it into the project root, and run it.

### passing custom arguments to `redbuild.sh`

`redbuild.sh` supports passing custom arguments to the build script. there are three types of arguments:

1. any arguments passed to `redbuild.sh` will be passed to the build script. for example, `redbuild.sh --foo` will run `build.sh --foo`.
2. any arguments in `CBUILD_ARGS` will be passed to the container build command. for example, `CBUILD_ARGS="--pull"` will run `podman build --pull`.
3. any arguments in `CRUN_ARGS` will be passed to the container run command. for example, `CRUN_ARGS="-v /tmp:/tmp"` will run `podman run -v /tmp:/tmp`.