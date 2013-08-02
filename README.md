StatsD [![Build Status](https://secure.travis-ci.org/datadog/statsd.png)](http://travis-ci.org/datadog/statsd)
======

A fork of [Etsy][etsy]'s network daemon for aggregating statistics (counters and timers), rolling them up, then sending them to [Datadog][datadog].

Key Concepts
--------

* *buckets*
  Each stat is in its own "bucket". They are not predefined anywhere. Buckets
can be named anything that will translate to Graphite (periods make folders,
etc)

* *values*
  Each stat will have a value. How it is interpreted depends on modifiers. In
general values should be integer.

* *flush*
  After the flush interval timeout (defined by `config.flushInterval`,
  default 10 seconds), stats are aggregated and sent to an upstream backend service.


Installation and Configuration
------------------------------

 * Install node.js
 * Clone the project
 * Create a config file from exampleConfig.js and put it somewhere
 * Start the Daemon:

    node stats.js /path/to/config

More Specific Topics
--------
* [Metric Types][docs_metric_types]
* [Graphite Integration][docs_graphite]
* [Supported Backends][docs_backend]
* [Admin TCP Interface][docs_admin_interface]
* [Backend Interface][docs_backend_interface]
* [Metric Namespacing][docs_namespacing]


Debugging
---------

There are additional config variables available for debugging:

* `debug` - log exceptions and print out more diagnostic info
* `dumpMessages` - print debug info on incoming messages

For more information, check the `exampleConfig.js`.

Supported Backends
------------------

StatsD supports multiple, pluggable, backend modules that can publish
statistics from the local StatsD daemon to a backend service or data
store. Backend services can retain statistics for
longer durations in a time series data store, visualize statistics in
graphs or tables, or generate alerts based on defined thresholds. A
backend can also correlate statistics sent from StatsD daemons running
across multiple hosts in an infrastructure.

StatsD includes the following backends:

* [Graphite][graphite] (`graphite`): Graphite is an open-source
  time-series data store that provides visualization through a
  web-browser interface.

By default, the `graphite` backend will be loaded automatically. To
select which backends are loaded, set the `backends` configuration
variable to the list of backend modules to load.

Backends are just npm modules which implement the interface described in
section *Backend Interface*. In order to be able to load the backend, add the
module name into the `backends` variable in your config. As the name is also
used in the `require` directive, you can load one of the provided backends by
giving the relative path (e.g. `./backends/graphite`).

Graphite Schema
---------------

Graphite uses "schemas" to define the different round robin datasets it houses (analogous to RRAs in rrdtool). Here's what Etsy is using for the stats databases:

    [stats]
    priority = 110
    pattern = ^stats\..*
    retentions = 10:2160,60:10080,600:262974

That translates to:

* 6 hours of 10 second data (what we consider "near-realtime")
* 1 week of 1 minute data
* 5 years of 10 minute data

This has been a good tradeoff so far between size-of-file (round robin databases are fixed size) and data we care about. Each "stats" database is about 3.2 megs with these retentions.

TCP Stats Interface
-------------------

A really simple TCP management interface is available by default on port 8126 or overriden in the configuration file. Inspired by the memcache stats approach this can be used to monitor a live statsd server.  You can interact with the management server by telnetting to port 8126, the following commands are available:

* stats - some stats about the running server
* counters - a dump of all the current counters
* timers - a dump of the current timers

The stats output currently will give you:

* uptime: the number of seconds elapsed since statsd started
* messages.last_msg_seen: the number of elapsed seconds since statsd received a message
* messages.bad_lines_seen: the number of bad lines seen since startup

Each backend will also publish a set of statistics, prefixed by its
module name.

Graphite:

* graphite.last_flush: the number of seconds elapsed since the last successful flush to graphite
* graphite.last_exception: the number of seconds elapsed since the last exception thrown whilst flushing to graphite

A simple nagios check can be found in the utils/ directory that can be used to check metric thresholds, for example the number of seconds since the last successful flush to graphite.

Installation and Configuration
------------------------------

1. Install node.js
2. Clone the project
3. Create a config file from exampleConfig.js and put it somewhere
4. Get your Datadog API key and generate an app key and stick them into your config file
5. Start the Daemon: `node stats.js /path/to/config`

Tests
-----

A test framework has been added using node-unit and some custom code to start
and manipulate statsd. Please add tests under test/ for any new features or bug
fixes encountered. Testing a live server can be tricky, attempts were made to
eliminate race conditions but it may be possible to encounter a stuck state. If
doing dev work, a `killall statsd` will kill any stray test servers in the
background (don't do this on a production machine!).

Tests can be executed with `./run_tests.sh`.


Meta
---------
- IRC channel: `#statsd` on freenode
- Mailing list: `statsd@librelist.com`


Contribute
---------------------

You're interested in contributing to StatsD? *AWESOME*. Here are the basic steps:

fork StatsD from here: http://github.com/etsy/statsd

1. Clone your fork
2. Hack away
3. If you are adding new functionality, document it in the README
4. If necessary, rebase your commits into logical chunks, without errors
5. Verfiy your code by running the test suite, and adding additional tests if able.
6. Push the branch up to GitHub
7. Send a pull request to the etsy/statsd project.

We'll do our best to get your changes in!

[datadog]: http://datadoghq.com

Contributors
-----------------

In lieu of a list of contributors, check out the commit history for the project:
http://github.com/etsy/statsd/commits/master and https://github.com/DataDog/statsd/commits/master
