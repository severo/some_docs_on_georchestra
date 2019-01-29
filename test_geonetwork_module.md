# Test the geonetwork module

To develop and test the [geOrchestra geonetwork module](https://github.com/georchestra/georchestra/tree/master/geonetwork), we can use the following configuration.

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

- commenting the geonetwork image:

    ```yml
    # geonetwork:
    #   image: georchestra/geonetwork:latest
    #   depends_on:
    #     - ldap
    #     - database
    #   volumes:
    #     - ./config:/etc/georchestra
    #     - geonetwork_datadir_lastest:/mnt/geonetwork_datadir
    #   environment:
    #     - XMS=256M
    #     - XMX=6G
    ```

- and adding the following ones, to expose the PostgreSQL database and the LDAP directory ports:

    ```yml
      database:
        (...)
        ports:
          - "5432:5432"
    (...)
      ldap:
        (...)
        ports:
          - "389:389"
    ```

Then deploy the docker images:

```bash
docker-compose up
```

We can verify that the database and ldap ports are exposed:

```bash
sudo ss -platn | grep -e 5432 -e 389
```

## Prepare the domains

The datadir configuration contains, in `~/dev/docker/config/geonetwork/geonetwork.properties`:

```bash
jdbc.host=database
jdbc.port=5432
(...)
ldap.base.provider.url=ldap://ldap:389
```

There are two options, use the one you prefer:

- modify these two lines, replacing `ldap:389` by `localhost:389` and `database` by `localhost`
- or, add the following line in `/etc/hosts`, in order to consider `ldap` and `postgresql` as domain names of the localhost IP `127.0.0.1`:

    ```
    127.0.0.1       ldap postgresql
    ```

## Prepare the minimal geonetwork datadir

Install the minimal geonetwork datadir into `/mnt/geonetwork_datadir`. Be careful with the branch when cloning.  By default, it clones the default branch which is the good one for 18.06 and master.


```bash
sudo mkdir /mnt/geonetwork_datadir
sudo chmod -R 777 /mnt/geonetwork_datadir
git clone git@github.com:georchestra/geonetwork_minimal_datadir.git /mnt/geonetwork_datadir
```

## Launch the geonetwork module

On this basis, we enter the geonetwork directory and launch jetty:

```bash
cd ~/dev/georchestra/geonetwork/web/
mvn -Dgeorchestra.datadir=/etc/georchestra -Dgeonetwork.dir=/mnt/geonetwork_datadir -Dgeonetwork.jeeves.configuration.overrides.file=/etc/georchestra/geonetwork/config/config-overrides-georchestra.xml jetty:run
```

The geonetwork module will be available at [http://localhost:8080/geonetwork/](http://localhost:8080/geonetwork).


## Authenticate into GeoNetwork

There are two alternate ways to get authenticated into the GeoNetwork module.

### Way 1 - add headers and access directly to jetty

The easiest way is to simulate we are logged in, setting headers using the [simple-modify-headers](https://github.com/didierfred/SimpleModifyHeaders) browser extension, and accessing directly to the port exposed by jetty [http://localhost:8080/geonetwork/](http://localhost:8080/geonetwork).

After installing the browser extension, enter its configuration:

- Url Patterns* : `http://localhost/geonetwork/*` (note that it does not allow to filter by port)

create two rules:

- Add / `sec-username` / `testadmin` / `Request`
- Add / `sec-roles` / `ROLE_GN_ADMIN` / `Request` (see [ldap/README.md](https://github.com/georchestra/georchestra/blob/18.06/ldap/README.md) for the meaning of the roles)

Then save, and start the extension.

Finally test the GeoNetwork entering http://localhost:8080/geonetwork/.

Note: this way does not allow to see the header or the analytics data from the GeoNetwork module.

### Way 2 - go through the security-proxy

The second way, that integrates better the GeoNetwork module inside the geOrchestra dockerized modules, implies going through the security proxy. To do so, first we need to find how to access the host network from the docker network. To do so, enter a docker machine via ssh:

```bash
$ ssh -p 2222 geoserver@localhost
Password: geoserver
```

- find the gateway (here `172.18.0.1`):

    ```bash
    $ ip r | grep default
    default via 172.18.0.1 dev eth0
    ```

- and exit

    ```bash
    $ exit
    ```

Then, we need to tell the security-proxy where to route the requests coming on `https://georchestra.mydomain.org/geonetwork/*`.

- To do so, we modify the following file:

    ```bash
    cd ~/dev/docker/config/security-proxy
    edit targets-mapping.properties
    ```

    to get the line:

    ```bash
    geonetwork=http://172.18.0.1:8080/geonetwork/
    ```

- and then restart the security-proxy docker image:

    ```bash
    docker-compose restart proxy
    ```

**Be careful: this IP may change when you restart the docker images.**

This way, you should be able to enter [https://georchestra.mydomain.org ](https://georchestra.mydomain.org/), log in (`testadmin`/`testadmin`), and enter the different views of the geonetwork module.

Note: this way allows to access analytics module data, and to see the header.
