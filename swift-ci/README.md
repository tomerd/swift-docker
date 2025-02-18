# Continuous Integration Docker images

Swift.org uses Docker based virtual build environment to build & qualify Swift toolchains on Linux.

The Continuous Integration system uses the Dockerfiles in this directory to define the virtual build environment, then runs the build and qualification steps inside a docker container based on the image.

## Directory Structure

The Dockerfiles used for Continuous Integration are layed out under the top level `swift-ci` directory. Under that, we have a directory for each of the target branches, e.g. Continuous Integration for Swift's `main` branch uses the `swift-ci/master` Dockerfiles.

There is also a specific directory (`swift-docc-render`) for the Dockerfile used to build Swift-DocC-Render. Swift-DocC-Render builds separately from the rest of the projects in the Swift toolchain and ships a pre-built copy for use in the toolchain in the Swift-DocC-Render-Artifact repository.

## Continuous Integration

This system is designed to support many distributions. Once a working Dockerfile is added to this repository, we set up Continuous Integration jobs to produce toolchains for the distribution and publish them on Swift.org.

## Local development & testing

First build & tag the Dockerfile:

```bash
docker build -f <dockerfile path> . -t <some tag>
```

Next, run the Swift build using that Docker image:

```bash
docker run \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  -v <swift source location>:/source \
  -v <some docker volume name>:/home/build-user \
  -w /home/build-user/ \
  <docker image tag from above> \
  /bin/bash -lc "cp -r /source/* /home/build-user/; ./swift/utils/build-script --preset buildbot_linux install_destdir=/home/build-user/swift-install installable_package=/home/build-user/swift-DEVELOPMENT-SNAPSHOT-$(date +'%F')-a.tar.gz"
```

## Contributions

Contributions via pull requests are welcome and encouraged :)

Focus on mainstream distributions such as Debian, Ubuntu, Fedora, CentOS, RedHat, etc.

Note that the build must run as non-root user.
