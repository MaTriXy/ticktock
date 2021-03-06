TickTock
========

TickTock is a timezone data management library for the JVM and Android targeting `java.time.*` APIs 
in Java 8 or above. Use this library if you want to bundle timezone data directly with your 
application rather than rely on the current device timezones (Android) or the default `<java.home>/lib`
version (JVM only).

## Usage

Current version (from [IANA](https://www.iana.org/time-zones)): `2020a`

### Android

Simply add the android tzdb startup dependency:

```gradle
implementation 'dev.zacsweers.ticktock:ticktock-android-tzdb:<version>'
```

This will automatically initialize it appropriately without any configuration needed using `androidx.startup`.
If you don't want automatic initialization, you can use the `-base` version and do it manually.

```java
AndroidTzdbZoneRules.init(<context>)
```

Note that Android usage assumes use of [core library desugaring](https://developer.android.com/studio/write/java8-support#library-desugaring). 
If you are not using it and/or are minSdk 26+, this library is of no use to you!

### JVM

Add the jvm tzdb dependency:

```gradle
implementation 'dev.zacsweers.ticktock:ticktock-jvm-tzdb:<version>'
```

Then call its initializer as early as possible in your application.

```java
JvmTzdbZoneRules.init()
```

This will make `ZoneRulesProvider` use TickTock's implementation with its bundled timezone data.

## Advanced

<details>
<summary>Eager caching</summary>

TickTock's default behavior is to lazily load timezone data on-demand. If you want to eagerly 
load data (for instance - on a background thread), TickTock offers a convenience helper API:

```java
// Synchronously load and cache all timezone rules
EagerZoneRulesLoading.cacheZones();
```

</details>

<details>
<summary>Custom Data Loading</summary>

By default, TickTock will try to load timezone data from Java resources via `ResourcesZoneDataLoader`. 
If you wish to customize this, you can provide your own loading mechanism via implementing a custom 
`ZoneDataLoader` and/or `ZoneDataProvider` and registering them via `TickTockPlugins` _before_ 
using any time APIs that would cause the system `ZoneRulesProvider` to initialize.

Usually, you would only want to implement a custom `ZoneDataLoader` and instantiate one of the built-in
`ZoneRulesProvider` implementations with it. TickTock comes with two: `TzdbZoneDataProvider` (the common case)
 and `LazyZoneDataProvider`. You can also implement your own provider on top of any `ZoneDataLoader`
 type as you see fit.
 
```java
CustomZoneDataLoader loader = new CustomZoneDataLoader();
TzdbZoneDataProvider provider = new TzdbZoneDataProvider(loader);
TickTockPlugins.setZoneDataProvider(() -> provider);
```

The Android artifacts use a custom assets-based loader to avoid the cost of loading from Java resources.

</details>

<details>
<summary>Custom Regions</summary>

By default, TickTock's prepackaged timezone data supports all regions. You can define your own via
implementing a custom `ZoneIdsProvider` and registering it via `TickTockPlugins` _before_ using any
time APIs that would cause the system `ZoneRulesProvider` to initialize.

```java
TickTockPlugins.setZoneIdsProvider(CustomZoneIdsProvider::new);
```

If no provider is specified, TickTock will use `TzdbZoneProvider`.

</details>

<details>
<summary>Lazy Zone Rules</summary>

TickTock's default behavior is focused around using traditional `tzdb.dat` files for timezone data
implemented via `TzdbZoneDataProvider`. Early adopters can try a custom, lazy-loading solution 
via `LazyZoneDataProvider` inspired by [LazyThreeTenBp](https://github.com/gabrielittner/lazythreetenbp).
In theory, this artifact would be lower overhead on startup for devices with slower IO and a lower 
application-lifetime memory impact by only keeping used zones in memory. We're seeking feedback on
whether this is truly worth supporting though, so please let us know!

</details>

## Download

[![Maven Central](https://img.shields.io/maven-central/v/dev.zacsweers.ticktock/ticktock-runtime.svg)](https://mvnrepository.com/artifact/dev.zacsweers.ticktock/ticktock-runtime)
```gradle
// Core runtime artifact
implementation 'dev.zacsweers.ticktock:ticktock-runtime:<version>'

// TZDB artifacts
implementation 'dev.zacsweers.ticktock:ticktock-jvm-tzdb:<version>'
implementation 'dev.zacsweers.ticktock:ticktock-android-tzdb-base:<version>'
implementation 'dev.zacsweers.ticktock:ticktock-android-tzdb:<version>'

// Lazy zone rules artifacts
implementation 'dev.zacsweers.ticktock:ticktock-jvm-lazyzonerules:<version>'
implementation 'dev.zacsweers.ticktock:ticktock-android-lazyzonerules-base:<version>'
implementation 'dev.zacsweers.ticktock:ticktock-android-lazyzonerules:<version>'
```

Snapshots of the development version are available in [Sonatype's `snapshots` repository][snapshots].

## Why?

https://www.zacsweers.dev/ticktock-desugaring-timezones/

License
-------

    Copyright (C) 2020 Zac Sweers & Gabriel Ittner

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

 [snapshots]: https://oss.sonatype.org/content/repositories/snapshots/
