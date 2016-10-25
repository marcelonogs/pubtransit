# System Architecture Design

## Data Flow Overview

The System Architecture is very simple: the Web page is static and doesn't
rely on any running appliance. This approach was decided to maximize the site
scalability and simplicity. There is no dynamic data.

The public transit data (downloaded as GPRS .zip files) is instead split in
small compressed binary .gz files (called here feed files) and made available
via HTTP or HTTPS protocol.

The Web browser is in charge of searching by parsing an index and download
those files are required by the user.

![Data Flow](https://cdn.rawgit.com/pubtransit/transit/b6e69741bd31762391f199eada34daa1d36fafae/doc/data-flow.svg)

## Feed files builder

Public [GPRS files](https://developers.google.com/transit/gtfs/) are dawnloaded
from public web sites and then they are eaten by the feed builder implemented
by transit_feed Python package.

GPRS files are zip files containing some CSV file. Every CSV is a relational
table. transit_feed package makes uses of [Pandas](http://pandas.pydata.org/)
to open them and then uses [numpy](http://www.numpy.org/) to make operations
over the big column arrays.

Original table are converted to a stripped version of original one are saved
by column: every column is saved on a separate file.

To make these files easier to digest from the web browser they are stored using
[MessagePack](http://msgpack.org/index.html) format ant then compressed using
[zlib](http://www.zlib.net/).

Two tables (stops and stop_times) are split into sub table each one represent
a rectangular region in the earth, each including data of up to 128 stops.

A [KDTree](https://en.wikipedia.org/wiki/K-d_tree) structure is saved
separately to allow the Web browser to look for the files required for the
region the zone is interested in.