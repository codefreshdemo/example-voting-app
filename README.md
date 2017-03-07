Example Voting App
=========

Getting started
---------------

Download [Docker](https://www.docker.com/products/overview). If you are on Mac or Windows, [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

Run in this directory:
```
docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:
```
docker swarm init
```
Once you have your swarm, in this directory run:
```
docker stack deploy --compose-file docker-stack.yml vote
```

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time


Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.

Deploy to Remote Swarm
----

Codefresh Docker CI/CD service is able to deply the voting application to specified remote swarm cluster.

Add following step to your `codefresh.yml`

```
version: '1.0'

steps:

...

  deploy_to_swarm:
    image: codefresh/remote-docker
    working_directory: ${{main_clone}}
    commands:
      - rdocker ${{RDOCKER_HOST}} docker stack deploy --compose-file docker-stack.yml ${{STACK_NAME}}
    environment:
      - SSH_KEY=${{SSH_KEY}}
    when:
      branch:
        only:
          - master

```

Where:

- `RDOCKER_HOST` - remote Docker swarm master machine, accessible over SSH (for example, ubuntu@ec2-public-ip)
- `STACK_NAME` - is new Docker stack name (use "vote", for example)
- `SSH_KEY` - private SSH key, used to access Docker swarm master machine
- `SPLIT_CHAR` - split character, you've used to replace `newline` in SSH key. Recommendation: use `,` (`comma` character).

#### Passing SSH key through ENV variable

Currently in order to pass SSH key through Codefresh UI, you need to convert it to single line string (replacing `newline` with `comma`), like this:

    $ SSH_KEY=$(cat ~/.ssh/my_ssh_key_file | tr '\n' ',')
