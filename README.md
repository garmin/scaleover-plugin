# Scaleover Plugin
This CF CLI Plugin rolls traffic from one app to another over a specified time interval. It is useful for blue green deployments or other situations where start / stop is not enough or too abrupt. It was created by Guido Westenberg and Josh Kruck.

[![wercker status](https://app.wercker.com/status/f5f8d90193968cce6f5d60583d85be3c/s "wercker status")](https://app.wercker.com/project/bykey/f5f8d90193968cce6f5d60583d85be3c)

## Assumptions
This plugin makes no assumptions about either application, nor does it attempt to "help" you. Route mapping to both applications, and the relationship between those applications is left to you. There is a simple default check to verify that the applications share a route which can be disabled with `--no-route-check`. Please use care when using `cf scaleover`, it is possible to hurt yourself.

## Requirements
Both applications must exist within the same space, and by default should share a route.

## Usage

Select two apps in the same space, and roll traffic between them.

### Options
* `--no-route-check` (default TRUE) - Since both apps are live at the same time, there is assumed use of a shared route.
* `--leave N` (default 0/stopped) - How many `blue` instances should remain running when using scaleover to `green`?
* `--wait-for-start` (defaults FALSE) - Should scaleover wait for confirmation that the scaled up instace(s) are `started` before scaling down?
* `--post-start-sleep Ns` (default '0s') - How long should scaleover wait after the new instances are considered 'started' for the app itself to initialize/bootstrap? Supports standard duration strings (eg '10s', '1m', etc). Used ONLY in conjunction with `--wait-for-start`.
* `--batch-size N` (default 1) - How many instances should be scaled (both up/down) at a time?

```
➜  scaleover-plugin git:(master) ✗ cf apps                                                                                                                    $
Getting apps in org test-org / space test-space as admin...
OK

name        requested state   instances   memory   disk   urls
node_v1.0   started           10/10       128M     1G     node_v1.0.10.244.0.34.xip.io, node-prod.10.244.0.34.xip.io
node_v1.1   stopped           0/1         128M     1G     node_v1.1.10.244.0.34.xip.io, node-prod.10.244.0.34.xip.io
➜  scaleover-plugin git:(master) ✗ cf scaleover node_v1.0 node_v1.1 20s
node_v1.0 (started) <<<<<<<<<<  node_v1.1 (stopped)
node_v1.0 (started) <<<<<< >>>> node_v1.1 (started)
node_v1.0 (started) <<< >>>>>>> node_v1.1 (started)
node_v1.0 (stopped)  >>>>>>>>>> node_v1.1 (started)
➜  scaleover-plugin git:(master) ✗ cf apps                                                                                                                    $
Getting apps in org test-org / space test-space as admin...
OK

name        requested state   instances   memory   disk   urls
node_v1.0   stopped           0/1         128M     1G     node_v1.0.10.244.0.34.xip.io, node-prod.10.244.0.34.xip.io
node_v1.1   started           10/10       128M     1G     node_v1.1.10.244.0.34.xip.io, node-prod.10.244.0.34.xip.io

```

This `node_v1.0 (started) <<< >>>>>>> node_v1.1 (started)` bit in the middle is a way cool ascii art animation that's worth the price of admission alone.

## Installation
### Install from CLI
  ```
  $ cf add-plugin-repo CF-Community http://plugins.cloudfoundry.org/
  $ cf install-plugin scaleover -r CF-Community
  ```


### Install from Source (need to have [Go](http://golang.org/dl/) installed)
  ```
  $ go get github.com/cloudfoundry/cli
  $ go get github.com/krujos/scaleover-plugin
  $ cd $GOPATH/src/github.com/krujos/scaleover-plugin
  $ go build
  $ cf install-plugin scaleover-plugin
  ```
