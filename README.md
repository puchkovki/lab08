## Laboratory work VIII [![Build Status](https://travis-ci.com/puchkovki/lab08.svg?branch=master)](https://travis-ci.com/puchkovki/lab08)

Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере **Docker**

```sh
$ open https://docs.docker.com/get-started/
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
- [x] 2. Ознакомиться со ссылками учебного материала
- [x] 3. Выполнить инструкцию учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
```

```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
```

```sh
$ cat > Dockerfile <<EOF # Added OS basic image
FROM ubuntu:18.04
EOF
```

```sh
$ cat >> Dockerfile <<EOF # Updated dependency list, install gcc, c++, cmake

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```

```sh
$ cat >> Dockerfile <<EOF # Where to copy our directories, establish working directory

COPY . print/
WORKDIR print
EOF
```

```sh
$ cat >> Dockerfile <<EOF # Added build commands

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```

```sh
$ cat >> Dockerfile <<EOF # # Added environment variable

ENV LOG_PATH /home/logs/log.txt
EOF
```

```sh
$ cat >> Dockerfile <<EOF # Added directories for mounting

VOLUME /home/logs
EOF
```

```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```

```sh
$ cat >> Dockerfile <<EOF # Added image entry point to create container; you may change ENTRYPOINT to CMD in case of errors

ENTRYPOINT ./demo
EOF
```

```sh
$ docker build -t logger . # Builded configuration file with name "logger" and path "./" for the Dickerfile
Sending build context to Docker daemon  13.79MB
Step 1/12 : FROM ubuntu:18.04
 ---> 4e5021d210f6
Step 2/12 : RUN apt update
 ---> Running in 79c31325eedc
Removing intermediate container 79c31325eedc
 ---> d7c73fa4464b
Step 3/12 : RUN apt install -yy gcc g++ cmake
 ---> Running in 6f8ea36a5dfe
Removing intermediate container 50c40581b439
 ---> 9880a03373c4
Step 4/12 : COPY . print/
 ---> a5029c93f46d
Step 5/12 : WORKDIR print
 ---> Running in e4ccaa4024b1
Removing intermediate container e4ccaa4024b1
 ---> 90cc38ada85e
Step 6/12 : RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
 ---> Running in fd6adf3db6cb
Removing intermediate container fd6adf3db6cb
 ---> 7f571a0c9ff9
Step 7/12 : RUN cmake --build _build
 ---> Running in 18d0195dd245
Scanning dependencies of target print
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
Removing intermediate container 18d0195dd245
 ---> d2ce9e229e6b
Step 8/12 : RUN cmake --build _build --target install
 ---> Running in 59f600d9ae3c
[100%] Built target print
Install the project...
-- Install configuration: "Release"
-- Installing: /print/_install/lib/libprint.a
-- Installing: /print/_install/include
-- Installing: /print/_install/include/print.hpp
-- Installing: /print/_install/cmake/print-config.cmake
-- Installing: /print/_install/cmake/print-config-release.cmake
Removing intermediate container 59f600d9ae3c
 ---> 4bf3d31b5126
Step 9/12 : ENV LOG_PATH /home/logs/log.txt
 ---> Running in 1027d28d94b7
Removing intermediate container 1027d28d94b7
 ---> b46451d757df
Step 10/12 : VOLUME /home/logs
 ---> Running in df90310ce5ba
Removing intermediate container df90310ce5ba
 ---> f4cd2befcb5a
Step 11/12 : WORKDIR _install/bin
 ---> Running in 987ccd083f4d
Removing intermediate container 987ccd083f4d
 ---> 511ef6569aab
Step 12/12 : ENTRYPOINT ./demo
 ---> Running in de00e998d8dc
Removing intermediate container de00e998d8dc
 ---> 8f77694119c2
Successfully built 8f77694119c2
Successfully tagged logger:latest
```

```sh
$ docker images # Showed all OS images
REPOSITORY     TAG           IMAGE ID        CREATED             SIZE
logger        latest       8f77694119c2        9 minutes ago       333MB
ubuntu        18.04        4e5021d210f6        2 weeks ago         64.2MB
```

```sh
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger # run logger image in online mode (-it flag), mounting /home/logs directory from ./logs directory (-v = volume)
text1
text2
text3
<C-D>
```

```sh
$ docker inspect logger# Output info about logger
[
    {
        "Id": "sha256:2690861acf0c3d0ffa8bf546ce2edc7a44acbc689e00e8717898efca34f0fb33",
        "RepoTags": [
            "logger:latest"
        ],
        "RepoDigests": [],
        "Parent": "sha256:135729961fb8e990d8e7b9b34da8b2d1b624de7b6cafbe07f224156b0b4daf34",
        "Comment": "",
        "Created": "2020-04-09T12:14:51.095077375Z",
        "Container": "71ee4a21a8acd016cac617b59ae61ff40117fba3474896330bdfdfca99d8767f",
        "ContainerConfig": {
            "Hostname": "71ee4a21a8ac",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "ENTRYPOINT [\"/bin/sh\" \"-c\" \"./demo\"]"
            ],
            "Image": "sha256:135729961fb8e990d8e7b9b34da8b2d1b624de7b6cafbe07f224156b0b4daf34",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "19.03.6",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": null,
            "Image": "sha256:135729961fb8e990d8e7b9b34da8b2d1b624de7b6cafbe07f224156b0b4daf34",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 338464315,
        "VirtualSize": 338464315,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/76f597a76f568973fe14260e82f42a3af1d8e16ed9a319fb90f5f5f3fd3c2a55/diff:/var/lib/docker/overlay2/6cc565f561e8636165c2cae630105d3ad8fa0efb93e061fe4aa6a64b74f2d5ba/diff:/var/lib/docker/overlay2/42b5dc380f967e94d6b41aaa59512ba9f01b664529bd5c194dda1cf28468d2f3/diff:/var/lib/docker/overlay2/12b05646bde9d2a82d08bc2c3acd5d658475daad7dff59986d7039010baf0aea/diff:/var/lib/docker/overlay2/f323b346a512e4fb9249722acdd542ed9da71e7e6481ba46c48bdf9acfa14f3c/diff:/var/lib/docker/overlay2/ba18af35f8adfff0289b5e809f2dafab3c6ae473233fede5032736ce6b3c6cb5/diff:/var/lib/docker/overlay2/403716a3a13fbc3711bd3a19dda56847ef6eab2f5dcc19e30f63e57b584aa21d/diff:/var/lib/docker/overlay2/5540010d311c8fbc0af4c10d17776adf99022f772a6a09ef712031dda243b1af/diff:/var/lib/docker/overlay2/7e8251bf6db5550f59a9b9d58eb03e6412d273a31420c27ebdecb3f4387728ae/diff",
                "MergedDir": "/var/lib/docker/overlay2/9ef9a83c5a75338fabcc460951e52fd65cf3fbb4e353ec96313e73ab44e7cc83/merged",
                "UpperDir": "/var/lib/docker/overlay2/9ef9a83c5a75338fabcc460951e52fd65cf3fbb4e353ec96313e73ab44e7cc83/diff",
                "WorkDir": "/var/lib/docker/overlay2/9ef9a83c5a75338fabcc460951e52fd65cf3fbb4e353ec96313e73ab44e7cc83/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:c8be1b8f4d60d99c281fc2db75e0f56df42a83ad2f0b091621ce19357e19d853",
                "sha256:977183d4e9995d9cd5ffdfc0f29e911ec9de777bcb0f507895daa1068477f76f",
                "sha256:6597da2e2e52f4d438ad49a14ca79324f130a9ea08745505aa174a8db51cb79d",
                "sha256:16542a8fc3be1bfaff6ed1daa7922e7c3b47b6c3a8d98b7fca58b9517bb99b75",
                "sha256:bf66378e505e31ff48807d162916ec5e0a9a60739ecc377167d18155fcb71d87",
                "sha256:8c3b2dd5fa87423d5acc9484980a760e35f66f785a9a908bb69f92093a8903dd",
                "sha256:212f37638101daaa326faff7a0adaa9c423cf7903c091232d93180cf7a2e4f33",
                "sha256:5edfd1e45174e3162f7eae352b555979baa960c804801c60bf823307037516da",
                "sha256:64e4dc62e15486b08407d502ab80ddeea67981c2340b6c06cf7a68ff1e0aecaf",
                "sha256:60a67e5e2f155e3f08850e210f15b86b30674fc7f2ca866e676e627f2f50ac50"
            ]
        },
        "Metadata": {
            "LastTagTime": "2020-04-09T15:14:51.616923577+03:00"
        }
    }
]

```

```sh
$ cat logs/log.txt
text1
text2
text3
```

```sh
$ gsed -i 's/lab07/lab08/g' README.md # Changed Travis icon
```

```sh
$ vim .travis.yml # Changed travis.yml file
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```

```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
[master 7679a27] adding Dockerfile
 2 files changed, 24 insertions(+), 12 deletions(-)
$ git push origin master
Total 72 (delta 17), reused 0 (delta 0)
remote: Resolving deltas: 100% (17/17), done.
To https://github.com/puchkovki/lab08
 * [new branch]      master -> master

```

```sh
$ travis login --auto --pro
Successfully logged in as puchkovki!
$ travis enable
puchkovki/lab08: enabled :)
```

## Report

```sh
$ popd
$ export LAB_NUMBER=08
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Links

- [Book](https://www.dockerbook.com)
- [Instructions](https://docs.docker.com/engine/reference/builder/)

```
Copyright (c) 2015-2019 The ISC Authors
```
