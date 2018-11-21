# Test the console module

To develop and test the [geOrchestra console module](https://github.com/georchestra/georchestra/tree/master/console), we can use the following configuration.

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
    # mapfishapp:
    #    image: georchestra/mapfishapp:latest
    [... until this end of the file]
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

The datadir configuration contains, in `~/dev/docker/config/console/console.properties`:

```bash
ldapUrl=ldap://ldap:389
(...)
psql.url=jdbc:postgresql://database:5432/georchestra
```

There are two options, use the one you prefer:

- modify these two lines, replacing `ldap:389` by `localhost:389` and `database:5432` by `localhost:5432`
- or, add the following line in `/etc/hosts`, in order to consider `ldap` and `postgresql` as domain names of the localhost IP `127.0.0.1`:

    ```
    127.0.0.1       ldap postgresql
    ```
## Launch the console module

On this basis, we enter the console directory and launch jetty:

```bash
cd ~/dev/georchestra/console
mvn -Dgeorchestra.datadir=../../docker/config/ jetty:run
```

The console module will be available at [http://localhost:8286/console/](http://localhost:8286/console).

## Authenticate to the console

There are two alternate ways to get authenticated into the console module.

### Way 1 - add headers and access directly to jetty

The easiest way is to simulate we are logged in, setting headers using the [simple-modify-headers](https://github.com/didierfred/SimpleModifyHeaders) browser extension, and accessing directly to the port exposed by jetty [http://localhost:8286/console/](http://localhost:8286/console).

After installing the browser extension, enter its configuration:

- Url Patterns* : `http://localhost/console/*` (note that it does not allow to filter by port)

create two rules:

- Add / `sec-username` / `testadmin` / `Request`
- Add / `sec-roles` / `ROLE_SUPERUSER` / `Request`

Then save, and start the extension.

Finally test the console entering http://localhost:8286/console/.

Note: this way does not allow to see the header or the analytics data from the console module.

### Way 2 - go through the security-proxy

The second way, that integrates better the console module inside the geOrchestra dockerized modules, implies going through the security proxy. To do so, first we need to find how to access the host network from the docker network. To do so, enter a docker machine via ssh:

```bash
$ ssh -p 2222 geoserver@localhost
Password: [geoserver]
```

- and find the gateway (here `172.18.0.1`):

    ```bash
    $ ip r | grep default
    default via 172.18.0.1 dev eth0
    ```

Then, we need to tell the security-proxy where to route the requests coming on `https://georchestra.mydomain.org/console/*`.

- To do so, we modify the following file:

    ```bash
    cd ~/dev/docker/config/security-proxy
    edit targets-mapping.properties
    ```

    to get the line:

    ```bash
    console=http://172.18.0.1:8286/console/
    ```

- and then restart the security-proxy docker image:

    ```bash
    docker-compose restart proxy
    ```

This way, you should be able to enter [https://georchestra.mydomain.org ](https://georchestra.mydomain.org/), log in, and enter the different views of the console (user details, change password, manage users and groups, etc.)

Note: this way allows to access analytics module data, and to see the header.
