---
title: "Options for hosting your private pip packages"
excerptOther: "Sometimes you don't want to publish a pip package for the world to see. What are the options for private hosting?"
postDate: "2021-07-28"
date: "2021-07-28"
tags:
  - pip
  - python
  - packages
---

In this post we will look at 3 options for hosting your private pip packages.

- [pip VCS support](#pip-vcs-support)
- [A private registry](#a-private-registry)
- [jfrog](#jfrog)

We will also look at how to use these options with docker and in your build pipelines. This post assumes you have python setup and have a basic understanding of [pip](https://pip.pypa.io/en/stable/).

# pip VCS support

Here we will use pip's build in [VSC support](https://pip.pypa.io/en/stable/topics/vcs-support/).
For demonstration purposes we are going to work with the popular [requests package](https://github.com/psf/requests)
but this will work the same for your own private git repository. You must setup [connecting to GitHub with SSH](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh) to follow along.

## [Optional] setup a pipenv environment

```bash
mkdir pip-vcs-practice
cd pip-vcs-practice
pipenv --python 3.8
```

## Install requests from github

Install requests from github with pip:

```bash
pip install git+ssh://git@github.com/psf/requests.git@master
```

Or to run in pipenv:

```bash
pipenv run pip install git+ssh://git@github.com/psf/requests.git@master
```

You should see output similar to this:

```
Collecting git+ssh://****@github.com/psf/requests.git@master
  Cloning ssh://****@github.com/psf/requests.git (to revision master) to /private/var/folders/my/k74vjg3x1yz39_v3tdd9wrh40000gn/T/pip-req-build-e0309emi
  Running command git clone -q 'ssh://****@github.com/psf/requests.git' /private/var/folders/my/k74vjg3x1yz39_v3tdd9wrh40000gn/T/pip-req-build-e0309emi
Collecting urllib3<1.27,>=1.21.1
  Using cached urllib3-1.26.6-py2.py3-none-any.whl (138 kB)
Collecting certifi>=2017.4.17
  Using cached certifi-2021.5.30-py2.py3-none-any.whl (145 kB)
Collecting charset_normalizer~=2.0.0
  Using cached charset_normalizer-2.0.3-py3-none-any.whl (35 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.2-py3-none-any.whl (59 kB)
     |████████████████████████████████| 59 kB 1.3 MB/s
Building wheels for collected packages: requests
  Building wheel for requests (setup.py) ... done
  Created wheel for requests: filename=requests-2.26.0-py2.py3-none-any.whl size=62462 sha256=46380a886a8f77e47ebe9b42ea8e9d1704ba4235eef2d067096d0c87939a87f8
  Stored in directory: /private/var/folders/my/k74vjg3x1yz39_v3tdd9wrh40000gn/T/pip-ephem-wheel-cache-mxfhppp8/wheels/7b/89/0b/ee7eeb21bf18417fc5b84b57197ad2b1035cf8d45d9ab4ecee
Successfully built requests
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2021.5.30 charset-normalizer-2.0.3 idna-3.2 requests-2.26.0 urllib3-1.26.6
```

An example using a requirements.txt file:

```bash
echo git+ssh://git@github.com/psf/requests.git@master#egg=requests >> requirements.txt
pip install -r requirements.txt
```

## With docker

You can use ssh forwarding to run your private pip packages in docker.

First, add keys to ssh-agent (this add all in $HOME/.ssh but can be modified to be more specific:

```bash
ssh-add
```

You should see an output saying "Identity added:" and the name of any keys.

Make sure ssh-agent is actually running:

```bash
eval `ssh-agent`
echo $SSH_AUTH_SOCK
```

You should see the path to the ssh socket.

You can use this docker setup as an example:

```dockerfile
FROM python:3.8-slim

RUN apt update && apt install -y \
    openssh-client \
    git \
    && rm -rf /var/cache/apt/*  /var/lib/apt/lists/*

# download public key for github.com
RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

RUN --mount=type=ssh pip install git+ssh://git@github.com/psf/requests.git@master
```

# A private registry

# jfrog
