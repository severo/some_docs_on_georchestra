# Test the cas module

To develop and test the [geOrchestra cas module](https://github.com/georchestra/georchestra/tree/master/cas-server-webapp), we can use the following configuration.

## Code base

Download the code base of geOrchestra:


```bash
mkdir ~/dev
cd ~/dev
git clone git@github.com:georchestra/georchestra.git
cd ~/dev/georchestra
git checkout master
```

## Docker

Download the code base of the docker composition:

```bash
cd ~/dev
git clone git@github.com:georchestra/docker.git
cd docker
git checkout master
git submodule update --init --remote
docker-compose pull
```

Disable the modules we will not use:

```bash
cd ~/dev/docker
edit docker-compose.yml
```

- commenting the following lines:

    ```yml
    # cas:
    #    image: georchestra/cas:latest
    [... until this end of the block]
    ```

- and adding the following ones, to expose the LDAP directory port:

    ```yml
    ldap:
        (...)
        ports:
          - "389:389"
    ```

Then edit

```bash
edit docker-compose.override.yml
```

- commenting the following lines:

    ```yml
    # cas:
    #   labels:
    #     - "traefik.enable=true"
    #     - "traefik.backend=cas"
    #     - "traefik.frontend.rule=Host:georchestra.mydomain.org;PathPrefix:/cas"
    ```

Then deploy the docker images:

```bash
docker-compose up
```

We can verify that the ldap port is exposed:

```bash
sudo ss -platn | grep -e 389
```

## Prepare the domains

The datadir configuration contains, in `~/dev/docker/config/cas/cas.properties`:

```bash
ldapUrl=ldap://ldap:389
```

There are two options, use the one you prefer:

- modify this line, replacing `ldap:389` by `localhost:389`
- or, add the following line in `/etc/hosts`, in order to consider `ldap` as a domain name of the localhost IP `127.0.0.1`:

    ```
    127.0.0.1       ldap
    ```
## Launch the cas module

On this basis, we enter the cas directory and launch jetty:

```bash
cd ~/dev/georchestra/cas
mvn jetty:run
```

Note: if your datadir is not linked to `/etc/georchestra/`, you should add the Java Server variable (and maybe other ones, see the module README.md):

```bash
mvn -Dgeorchestra.datadir=../../docker/config/ jetty:run
```

The cas module will be available at [http://localhost:8181/cas/](http://localhost:8181/cas).

## Get access to the header and security-proxy

The CAS module is closely related to the security-proxy module, and testing it generally requires to be able to perform actions with the security-proxy. To do it, it's also a good idea to have access to the header module since it contains the links and displays the authentication status. So, ideally, to test the CAS module, it should be running into the Maven's embedded Jetty, while the other modules would be running as Docker containers.

First we need to find how to access the host network from the docker network. To do so, enter a docker machine via ssh:

```bash
$ ssh -p 2222 geoserver@localhost
Password: [geoserver]
```

- and find the gateway (here `172.18.0.1`):

    ```bash
    $ ip r | grep default
    default via 172.18.0.1 dev eth0
    ```

- and exit

    ```bash
    $ exit
    ```

[HERE - Work in progress]

Then, we need to tell the security-proxy where to route the requests destinated to the CAS module.

- To do so, we modify the following file:

    ```bash
    cd ~/dev/docker/config/security-proxy
    edit security-proxy.properties
    ```

    to get the line:

    ```bash
    casTicktValidation=http://172.18.0.1:8181/cas
    ```

- and then restart the security-proxy docker image:

    ```bash
    docker-compose restart proxy
    ```

**Be careful: this IP may change when you restart the docker images.**

This way, you should be able to enter [https://georchestra.mydomain.org ](https://georchestra.mydomain.org/), log in, and enter the different views of the cas (user details, change password, manage users and groups, etc.)

Note: this way allows to access analytics module data, and to see the header.


- specify headerUrl = https://georchestra.mydomain.org
- Delete X-Frame-Options header from responses coming from https://georchestra.mydomain.org - https://stackoverflow.com/a/44856473/7351594
- Problem: https://github.com/georchestra/georchestra/issues/2363
