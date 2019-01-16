# How to report a commit between branches



## GeoNetwork

Imagine we commit to the georchestra-gn3.4-18.06 branch of [geOrchestra's GeoNetwork](https://github.com/georchestra/geonetwork): ["Add a French translation for addServiceLayersToMaponlinesrc string"](https://github.com/georchestra/geonetwork/pull/105/commits/a827225f8a320f731b998cb6150d6452de40deeb).

We then have to:

- get the last commit into the geonetwork submodule from the geOrchestra repository root:

    ```bash
    git checkout 18.06
    git submodule update
    cd geonetwork
    git checkout georchestra-gn3.4-18.06
    git fetch -p origin
    git merge origin/georchestra-gn3.4-18.06 georchestra-gn3.4-18.06
    cd ..
    ```

- create a dedicated geOrchestra branch, push it to `<my_remote>` and go creating a Pull Request:

    ```bash
    git checkout -b "add-a-translation-string-to-geonetwork"
    git add -p
    git commit -m "geonetwork - add a translation string to geonetwork"
    git push --set-upstream <my_remote> add-a-translation-string-to-geonetwork
    ```

- once the PR is accepted, get it in the local 18.06 branch:

    ```bash
    git checkout 18.06
    git fetch -p origin
    git merge origin/18.06 18.06

    ```

- if it corresponds to do so, report the commit to the next branches in the georchestra/geonetwork repository:

    ```bash
    git checkout master
    git submodule update
    cd geonetwork
    git checkout georchestra-gn3.4-master
    git checkout -b translate-a-french-string
    git merge georchestra-gn3.4-18.06 translate-a-french-string
    git push --set-upstream <my_remote> transate-a-french-string
    ```

    and then do the same steps to integrate it into the master branch of georchestra/georchestra.
