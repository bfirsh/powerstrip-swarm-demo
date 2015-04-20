# powerstrip-swarm-demo

A demo of a [Compose](https://github.com/docker/compose) app spanning multiple hosts using [powerstrip-flocker](https://github.com/clusterhq/powerstrip-flocker) and [powerstrip-weave](https://github.com/binocarlos/powerstrip-weave) working with [swarm](https://github.com/docker/swarm).

## install

First you need to install:

 * [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
 * [Vagrant](http://www.vagrantup.com/downloads.html)
 * [Compose](https://docs.docker.com/compose/)

## start vms

To run the demo:

```bash
$ git clone https://github.com/binocarlos/powerstrip-swarm-demo
$ cd powerstrip-swarm-demo
$ vagrant up
# Configure Compose and Docker client to talk to Swarm master
$ export DOCKER_HOST=tcp://172.16.255.250:2375
```

## running with compose

```bash
$ docker-compose up -d
```

You can view the application running on [http://172.16.255.251:8080](http://172.16.255.251:8080).

Run `docker-compose ps` to see the processes that are running or `docker-compose logs` to see the log output.

The `api` service is currently on `node1`. Compose doesn't output the node a container is running on, but you can see this by running `docker ps | grep api`.

## moving data and IP from one host to another

The `api` service has a constraint to be on a node tagged `disk`. `node1` has the tag `disk`, and `node2` has the tag `ssd`. To move the `api` service from `node1` to `node2`, we can update the constraint to `ssd`.

Under the `api` service in `docker-compose.yml`, update `constraint:storage==disk` to read `constraint:storage==ssd`. Then run these commands:

```bash
$ docker-compose stop api
$ docker-compose rm api
$ docker-compose up -d api
```

If you run `docker ps | grep api` you will see that the `api` service is now running on `node2`, but it has kept its data and IP address!

## info

You can see the state of the swarm by doing this :

```bash
$ docker info
$ docker ps -a
```

This displays the containers used for powerstrip, flocker and weave

You can see the state of the weave network by doing this on node1 or node2:

```bash
$ vagrant ssh node1
node1$ sudo bash /vagrant/install.sh weave status
```

## about

This demo consists of 3 servers - a master and 2 nodes.

The master runs the swarm deamon and the flocker control server - the 2 nodes each run powerstrip and the 2 powerstrip adapters (for flocker and weave).

```
             compose

                |

           swarm deamon
          flocker-control

           /          \

       node1          node2
       disk            SSD

         |              |

    powerstrip       powerstrip
     flocker          flocker
      weave            weave

        |               |

      docker          docker

```

We run a database and a HTTP server on node1.  We then decide to upgrade our database container to run on a server that has an SSD.

The IP address to connect to the database is hardcoded - when we move the database container we are migrating the IP address AND the data!

#### Initial Layout:

 * NODE1 (disk)
  * HTTP server
  * Database server (10.255.0.10 + /flocker/data)

 * NODE2 (SSD)

#### Final Layout:

 * NODE1 (disk)
  * HTTP server

 * NODE2 (SSD)
  * Database server (10.255.0.10 + /flocker/data)

## conclusion

We have moved both an IP address and a data volume across hosts using nothing more than compose!
