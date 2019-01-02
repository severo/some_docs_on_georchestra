# Miscellaneous

Some notes to be grouped and/or developed later.

## Debug

geOrchestra Javascript files are minimized and concatenated to reduce the number of HTTP calls and their weights. See for example:

- [JS minimization in mapfishapp](https://github.com/georchestra/georchestra/blob/18.06/mapfishapp/jsbuild/build.sh)
- [JS minimization in GeoNetwork](https://github.com/georchestra/geonetwork/tree/georchestra-gn3.4-18.06/wro4j)

During the development, it's good to have access to the original JS files in order to debug, step by step. To get these unminimized files, add the `?debug` parameter in the URL. Be careful, include the parameter before the `#` character if the URL contains it.
