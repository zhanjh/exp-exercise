# Optimize

## Don't use synchronous functions

* Trace the usage of synchronous API by command-line flag `--trace-sync-io`

## Logging & Debugging

* console.log or console.error are not recommended, since they are synchronous.
* Debugging module: [debug](https://www.npmjs.com/package/debug)
* Logging module: [winston](https://www.npmjs.com/package/winston)

## Handle exceptions properly

* Should not to listen for the uncaughtException event.
* Use try-catch
* Use promises

## Things to do in  your environment / setup

* Set NODE_ENV to "production"
* Use a process manager to ensoure app restarts if it crashes.
	* PM2 / Forever / StrongLoop Process Manager
* Use an init system
	* Systemd
* Run you app in a cluster
* Use a load balancer
* Use a reverse proxy


## ref
* [Production best practices: performance and reliability](https://expressjs.com/en/advanced/best-practice-performance.html#dont-use-synchronous-functions)
