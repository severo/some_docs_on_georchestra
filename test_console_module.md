# Test the console module

To develop and test the [geOrchestra console module](https://github.com/georchestra/georchestra/tree/master/console), we can use the following configuration, that does not use the security proxy, but the [simple-modify-headers](https://github.com/didierfred/SimpleModifyHeaders) browser extension.

## Codebase

Download the codebase of geOrchestra:


```bash
mkdir ~/dev
cd ~/dev
git clone git@github.com:georchestra/georchestra.git
cd ~/dev/georchestra
git checkout master
```

## Docker

Download the codebase of the docker composition:

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

- and adding the following ones, to expose the postgresql database and the LDAP directory ports:

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

## Prepare the headers

In order to simulate we are logged in, we will set the headers in the [simple-modify-headers](https://github.com/didierfred/SimpleModifyHeaders) browser extension. After installing, enter the configuration:

- Url Patterns* : `http://localhost/console/*` (note taht it does not allow to filter by port)

create two rules:

- Add / `sec-username` / `testadmin` / `Request`
- Add / `sec-roles` / `ROLE_SUPERUSER` / `Request`

save and start the extension.

## Launch the console module

On this basis, we enter the console directory and lanch jetty:

```bash
cd ~/dev/georchestra/console
mvn -Dgeorchestra.datadir=../../docker/config/ jetty:run
```

## Test in the browser

In the browser, enter http://localhost:8286/console/.
