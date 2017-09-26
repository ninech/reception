# Reception

A dashboard and reverse proxy for your
[**docker-compose**](https://docs.docker.com/compose/) projects. It does not require any dependencies but _Docker_.

![screenshot](https://user-images.githubusercontent.com/804532/30865946-41f08066-a2d8-11e7-86d1-fbe28a418c71.png)

## About

This program shows all _docker-compose_ projects that are running on a handy overview page.
It has a built-in reverse-proxy, so that any container's exposed port are accessible as '_http://container.composeproject.docker_'.
In order to be able to resolve '_anything_.docker' to *localhost*, this tool also ships a tiny tiny DNS server.

As a result, you'll be able to access your _docker-compose_ projects as *container.compose-project.docker*,
and the traffic will automatically be forwarded to the corresponding port,
even as you fire up and shut down new _docker-compose_ projects.

## Installation

For resolving `*.docker` to `localhost` (i.e. `127.0.0.1` or `::1`), changes to your local configuration are required.

Btw, we assume you have _Docker_ and _docker-compose_ already installed.

### Linux

Fetch and build reception:

    go get github.com/ninech/reception

Make sure that *reception* is responsible for resolving the `.docker` TLD:

    sudo -s
    mkdir /etc/resolver
    echo "nameserver ::1" > /etc/resolver/docker
    echo "nameserver 127.0.0.1" >> /etc/resolver/docker

Now run *reception*:

    sudo reception

Try to go to http://reception.docker.

### macOS

Install *reception* using [homebrew](https://brew.sh/):

    brew install ninech/reception/reception

Next you need to register *reception* as the resolver for the `docker` TLD. Run the
following on your command-line

    sudo -s
    mkdir /etc/resolver
    echo "nameserver ::1" > /etc/resolver/docker
    echo "nameserver 127.0.0.1" >> /etc/resolver/docker

At last, start the service:

    sudo brew services start ninech/reception/reception

Now try to go to http://reception.docker.

### Windows

¯\_(ツ)_/¯

## Configuration

_reception_ is customizable to some extend. See `reception -h` for a complete list of configuration parameters.

    $ reception -h
    (c) 2017 Nine Internet Solutions AG
    Usage of reception:
      -dns.address string
        	Defines on which address and port the HTTP daemon listens. (default "localhost:53")
      -docker.endpoint string
        	How reception talks to Docker. (default "unix:///var/run/docker.sock")
      -http.address string
        	Defines on which address and port the HTTP daemon listens. (default "localhost:80")
      -tld string
        	Defines on which TLD to react for HTTP and DNS requests. Should end with a "." . (default "docker.")

## Tips & Tricks

### "Main" container

The "main" container defines, where the project address ends up (i.e. http://yourproject.docker):
The container should either have a docker-compose label of `reception.main` or should be called `app`:

    version: '2'
    services:
      app:    <----- like this
        image: nginx
        labels:
          reception.main: true  <--- or like this
        ports:
          - 80

### Ports

In your `docker-compose.yaml` file, we advice to not specific a local port and to not export any
unnecessary ports either. _docker-compose_ will bind your exported port to any available local port,
and _reception_ will make sure, that there's a url for it.

This way, you can launch several containers that expose the same port without conflict and therefore 
avoid port collisions across projects.

**Do**

    version: '2'
    services:
      app:
        image: nginx
        depends_on: pgsql
        ports:
          - 80    <----- like this
      pgsql:
        image: postgresql

**Don't**

    version: '2'
    services:
      app:
        image: nginx
        depends_on: pgsql
        volume:
          - ./php:/usr/share/nginx/html
        ports:
          - 80:80    <----- and _not_ like this
      pgsql:
        image: postgresql
        ports:
          - 5432:5432    <----- and _not_ like this

### HTTP Port

In order to detect which port "the http port" of you container is, _reception_ looks for the well-known ports
80, 8080 and 3000. You can override this behaviour by setting the label `reception.http-port` to a port of your choice:

    version: '2'
    services:
      app:
        image: special
        labels:
          reception.http-port: 1234  <--- like this
        ports:
          - 1234

## Troubleshooting

### Reception can't bind to the ports

You must run reception as privileged user (i.e. `root`) for it to be able to bind to port 53 (dns) and port 80 (http). 

### _docker-compose_ projects can't start because of port conflicts

Most probably you assigned a fixed port mapping for any exposed ports. Look for something like the following:

```yml
version: 2
services:
  app:
    ports:
      -- "80:80"
```

In the case above, you would just replace `"80:80"` with `80`.

### Is the DNS cache outdated?

**Note:** _reception_ must be running at this stage already!

`nslookup foobar.docker` should resolve to `127.0.0.1` or `::1`.

If it doesn't, please flush the DNS cache:

```shell
# macOS
sudo killall -HUP mDNSResponder

# Linux
systemctl restart named
# or
systemctl restart nscd
```

## License

This program is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

## About

This piece of software is currently maintained and funded by [nine.ch](https://nine.ch).

[![nine.ch Logo](https://blog.nine.ch/assets/logo.png)](https://nine.ch)

We run your Linux server infrastructure – without interruptions, around the clock.