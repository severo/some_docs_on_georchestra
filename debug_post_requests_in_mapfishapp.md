# How to debug POST requests in mapfishapp

I recently had to work on a bug when opening a layer into mapfishapp from a metadata page in GeoNetwork catalogue. To see what I mean:

- go to a [random metadata page](https://www.geograndest.fr/geonetwork/srv/fre/catalog.search#/metadata/fr-120066022-srv-b2408a22-3fd9-4737-8593-22f44a191831),
- click on "addServiceLayersToMaponlinesrc",
- see that mapfishapp is open in a new tab to load the layer.

In order to investigate, I wanted to:

- use a mapfishapp instance running on localhost,
- use the debug mode,
- use the exact same HTTP method, headers and body,
- use Mozilla Firefox

## Launch a localhost mapfishapp instance

There is no difficulty here:

- ensure you have access to the [geOrchestra source code](https://github.com/georchestra/georchestra/), in the right branch (say [18.06](https://github.com/georchestra/georchestra/tree/18.06/) for the above example),
- ensure the [datadir](https://github.com/georchestra/datadir/) is installed in `/etc/georchestra/` and with the right branch ([18.06](https://github.com/georchestra/datadir/tree/18.06/) in our example),
- ensure you can compile the geOrchestra source code,
- go to the `mapfishapp/` directory in the georchestra and launch the Maven's embedded Jetty application server:

    ```bash
    mvn -Dgeorchestra.datadir=/etc/georchestra -Dmapfish-print-config=/etc/georchestra/mapfishapp/print/config.yaml jetty:run
    ```

The local mapfishapp instance should be running on http://localhost:8287/mapfishapp/.

## Modify the request to use the locahost

When clicking on the GeoNetwork link, Firefox opened a new tab showing the remote mapfishapp instance. In order to investigate, we want to access our local mapfishapp instance (note that you could also do the rest of this "tutorial" on the remote mapfishapp instance, but you would not be able to modify the mapfishapp code obviously). To do so, we apply the following operations:

- open the Developer Tools Network tab (Ctrl + Shift + E, or F12 + click on the "Network" tab)
- refresh the page (F5)
- a popup should appear, with something like `To display this page, Firefox Developer Edition must send information that will repeat any action (such as a search or order confirmation) that was performed earlier.`. Accept (the "Resend" button), because it means it will send the exact same request (same method: POST, same headers, same body, etc.).
- a lot of requests will appear. The first one is our POST request. To see it easier, it's possible to filter with `method:post` in the "Filter URLs" field above,
- click on the POST request: it will show all the parameters of the request and the response.
- right click on the POST request and select "Edit and Resend": it opens a form to modify the request.
- change the "URL" field from `https://www.geograndest.fr/mapfishapp/` to `http://localhost:8287/mapfishapp/`
- in the "Request Headers" field, replace `Host: www.geograndest.fr` by `Host: localhost:8287`
- and click "Send". It should reload the page with the new parameters. If it does not, right click on the new request that appeared in the list, and click "Open in New Tab", and in the new tab, repeat "Ctrl + Shift + E", "F5", "Resend" (and then you can close the previous tab).

## Modify the request to do to debug mode

If we want to inspect the JavaScript code of mapfishapp, it's useful to pass in debug mode to access the original source code, instead of the minified one. To do so, repeat the above steps, only changing this one:

- change the "URL" field from `https://www.geograndest.fr/mapfishapp/` to `http://localhost:8287/mapfishapp/?debug`

With that change, you will be able to use the Firefox Debugger (Ctrl + Shift + S, or F12 + click on the "Debugger" tab) to inspect the JavaScript code, set breakpoints, etc.

