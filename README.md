# Scala MaxMind Geo-IP [![Build Status](https://travis-ci.org/snowplow/scala-maxmind-geoip.png)](https://travis-ci.org/snowplow/scala-maxmind-geoip)

## Introduction

This is a Scala wrapper for the MaxMind [Java Geo-IP] [java-lib] library. The main benefits of using this wrapper over directly calling the Java library from Scala are:

1. **Easier to setup/test** - the SBT project definition automatically pulls down the latest MaxMind Java code and `GeoLiteCity.dat`
2. **Better type safety** - the MaxMind Java library is somewhat null-happy. This wrapper uses Option boxing wherever possible
3. **Better performance** - as well as or instead of using MaxMind's own caching (`GEOIP_MEMORY_CACHE`), you can also configure an LRU (Least Recently Used) cache of variable size

## Installation

The latest version of scala-maxmind-geoip is **0.0.5** and is dual-published to be Scala 2.9 and 2.10 compatible.

Add this to your SBT config:

```scala
// Resolvers
val snowplowRepo = "SnowPlow Repo" at "http://maven.snplow.com/releases/"
val twitterRepo  = "Twitter Maven Repo" at "http://maven.twttr.com/"

// Dependency
val maxmindGeoip = "com.snowplowanalytics"  %% "scala-maxmind-geoip"  % "0.0.5"
```

Note the double percent (`%%`) between the group and artifactId. That'll ensure you get the right package for your Scala version.

Retrieve the `GeoLiteCity.dat` file from the [MaxMind downloads page] [maxmind-downloads] ([direct link] [geolitecity-dat]).

## Usage

Here is a simple usage example:

```scala
import com.snowplowanalytics.maxmind.geoip.IpGeo

val ipGeo = IpGeo(dbFile = "/opt/maxmind/GeoLiteCity.dat", memCache = false, lruCache = 20000)

for (loc <- ipGeo.getLocation("213.52.50.8")) {
  println(loc.countryCode)   // => "NO"
  println(loc.countryName)   // => "Norway" 
}
```

Note that `GeoLiteCity.dat` is updated by MaxMind each month - see [maxmind-geolite-update] [maxmind-geolite-update] for a Python script that pings MaxMind regularly to keep your local copy up-to-date.

For further usage examples for Scala MaxMind Geo-IP, please see the tests in [`IpGeoTest.scala`] [ipgeotest-scala]. The test suite downloads its own copy of `GeoLiteCity.dat` from MaxMind for testing purposes.

## Implementation details

### IpGeo constructor

The signature is as follows:

```scala
class IpGeo(dbFile: File, memCache: Boolean = true, lruCache: Int = 10000)
```

In the `IpGeo` companion object there is an alternative constructor which takes a String `dbFile` instead:

```scala
def apply(dbFile: String, memCache: Boolean = true, lruCache: Int = 10000)
```

In both signatures, the `memCache` flag is set to `true` by default. This flag enables MaxMind's own caching (`GEOIP_MEMORY_CACHE`).

The `lruCache` value defaults to `10000` - meaning Scala MaxMind Geo-IP will maintain an LRU cache of 10,000 values, which it will check prior to making a MaxMind lookup. To disable the LRU cache, set its size to zero, i.e. `lruCache = 0`.

### IpLocation case class

The `getLocation(ip)` method returns an `IpLocation` case class with the following structure:

```scala
case class IpLocation(
  countryCode: String,
  countryName: String,
  region: Option[String],
  city: Option[String],
  latitude: Float,
  longitude: Float,
  postalCode: Option[String],
  dmaCode: Option[Int],
  areaCode: Option[Int],
  metroCode: Option[Int]
  )
```

### LRU cache

We recommend trying different LRU cache sizes to see what works best for you.

Please note that the LRU cache is **not** thread-safe ([see this note] [twitter-lru-cache]). Switch it off if you are working with threads.

## Building etc

Assuming you already have SBT installed:

    $ git clone git://github.com/snowplow/scala-maxmind-geoip.git
    $ cd scala-maxmind-geoip
    $ sbt test
    <snip>
    [info] Passed: : Total 186, Failed 0, Errors 0, Passed 186, Skipped 0

If you want to build a 'fat jar':

    $ sbt assembly 

The fat jar is now available as:

    target/scala-maxmind-geoip-0.0.3-fat.jar

## Roadmap

Nothing planned currently, although we want to look into Specs2's data tables and see if they would be a better fit for the unit tests.

## Copyright and license

Copyright 2012-2013 Snowplow Analytics Ltd.

Licensed under the [Apache License, Version 2.0] [license] (the "License");
you may not use this software except in compliance with the License.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[java-lib]: http://www.maxmind.com/download/geoip/api/java/

[ipgeotest-scala]: https://github.com/snowplow/scala-maxmind-geoip/blob/master/src/test/scala/com/snowplowanalytics/maxmind/geoip/IpGeoTest.scala

[twitter-lru-cache]: http://twitter.github.com/commons/apidocs/com/twitter/common/util/caching/LRUCache.html

[maxmind-downloads]: http://dev.maxmind.com/geoip/legacy/geolite
[geolitecity-dat]: http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
[maxmind-geolite-update]: https://github.com/psychicbazaar/maxmind-geolite-update

[license]: http://www.apache.org/licenses/LICENSE-2.0
