# Use docker images for a branch

A good setup to work on a module, say `extractorapp`, is to:

- run the module (`extractorapp`) with jetty on the host, and
- run all the other modules as docker services

But the docker images provided from https://hub.docker.com/u/georchestra/ are not generated every day, and it's not easy to understand to which commit they have been generated. To avoid any surprise, it's a good idea to generate a set of docker images for the exact state of the branch on which you are working.

## Create docker images

To create docker images, let's say for the [18.06 branch](https://github.com/georchestra/georchestra/tree/18.06), follow the [instructions](https://github.com/georchestra/georchestra/blob/master/docker/README.md).

Once the compilation has finished (it could take time if the [#2269 issue](https://github.com/georchestra/georchestra/issues/2269) has not been resolved), we can see the list of the newly created images (check the date):

```bash
docker images
```

```
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
georchestra/header           latest              4b243148cca0        11 minutes ago      459MB
georchestra/analytics        latest              8633ae8a6803        11 minutes ago      483MB
georchestra/security-proxy   latest              dfe41890fff9        11 minutes ago      468MB
georchestra/atlas            latest              561289e8c39c        12 minutes ago      538MB
georchestra/mapfishapp       latest              798968506b33        12 minutes ago      507MB
georchestra/console          latest              e6f5366c5f09        13 minutes ago      483MB
georchestra/geowebcache      latest              b78c712a45c6        15 minutes ago      496MB
georchestra/extractorapp     latest              df7b3be7a574        16 minutes ago      683MB
georchestra/cas              latest              f953f50af9f0        18 minutes ago      477MB
georchestra/geoserver        latest              4d82222d54ff        19 minutes ago      566MB
georchestra/geonetwork       3-latest            f0c4815b5c15        21 minutes ago      717MB
```

Note that they all get the tag `latest` by default. But maybe you want to manage docker images for various branches at the same time. To do so, we will set a tag to the docker images in order to record when we created it (`20181122`), and from which git branch (in this case `18.06`). Note that there is autocomplete on the docker image hash, so use `<TAB>` key:

```
docker tag 4b243148cca0 georchestra/header:18.06_20181122
```

If we check the images again, we get:

```
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
georchestra/header           18.06_20181122      4b243148cca0        20 minutes ago      459MB
```

We repeat the pattern to all the images we created (tag `18.06_20181122`). Note: this process is awfully manual, there is space for improvements (please propose PR).

## Prepare a dedicated docker composition

In order to use these docker images, we modify the docker composition provided by [georchestra/docker](https://github.com/georchestra/docker):

- create a dedicated file:

    ```bash
    cp docker-compose.yml docker-compose.18.06.yml
    ```

- modify the tags:

    ```bash
    sed -i 's/latest/18.06_20181122/' docker-compose.18.06.yml
    ```

    it's a good idea to open `docker-compose.18.06.yml` to check if the tags replacement has been done correctly and fix possible errors (typically: `geonetwork`, `database`, `ldap`).

- in `docker-compose.18.06.yml`, `docker-compose.override.yml`, and `docker-compose.other-apps.yml`, replace the volumes by new ones (`ldap_data` -> `ldap_data_18.06`). Possibly you could have conflicts with previously existing volumes, so maybe the best solution (I'm too recently working with docker to be subtile) is `docker system prune`.

- in the `config` subdirectory (a submodule versionned in [georchestra/datadir](https://github.com/georchestra/datadir)), use the correct branch, in this case `docker-18.06` (be careful: the `docker-*` branches differ a lot from the other datadir branches, in particular in the way they manage the IP directions and domains):

    ```bash
    cd config
    git checkout docker-18.06
    cd ..
    ```

- launch the new composition:

    ```bash
    docker-compose -f docker-compose.18.06.yml -f docker-compose.override.yml up
    ```

## Update docker images
 
When the code of a module gets an important modification, and we need to update the corresponding docker image:

- [to be done]
