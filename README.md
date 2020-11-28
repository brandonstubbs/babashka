<img src="logo/babashka.svg" width="425px">

[![CircleCI](https://circleci.com/gh/borkdude/babashka/tree/master.svg?style=shield)](https://circleci.com/gh/borkdude/babashka/tree/master)
[![project chat](https://img.shields.io/badge/slack-join_chat-brightgreen.svg)](https://app.slack.com/client/T03RZGPFR/CLX41ASCS)
[![Financial Contributors on Open Collective](https://opencollective.com/babashka/all/badge.svg?label=financial+contributors)](https://opencollective.com/babashka) [![Clojars Project](https://img.shields.io/clojars/v/borkdude/babashka.svg)](https://clojars.org/borkdude/babashka)
[![twitter](https://img.shields.io/badge/twitter-%23babashka-blue)](https://twitter.com/search?q=%23babashka&src=typed_query&f=live)

A Clojure [babushka](https://en.wikipedia.org/wiki/Headscarf) for the grey areas of Bash.

<blockquote class="twitter-tweet" data-lang="en">
    <p lang="en" dir="ltr">Life's too short to remember how to write Bash code. I feel liberated.</p>
    &mdash;
    <a href="https://github.com/laheadle">@laheadle</a> on Clojurians Slack
</blockquote>

## Survey

Please participate in the [babashka survey](https://nl.surveymonkey.com/r/H2HK3RC)!

It only takes a few minutes to fill out. The survey will run until December 1.
I'll publish the results afterwards.

## Introduction

The main idea behind babashka is to leverage Clojure in places where you would
be using bash otherwise.

As one user described it:

> I’m quite at home in Bash most of the time, but there’s a substantial grey area of things that are too complicated to be simple in bash, but too simple to be worth writing a clj/s script for. Babashka really seems to hit the sweet spot for those cases.

### Goals

* **Fast starting** Clojure scripting alternative for JVM Clojure
* **Easy installation:** grab the self-contained binary and run. No JVM needed.
* **Familiar:** targeted at JVM Clojure users
* **Cross-platform:** supports linux, macOS and Windows
* **Interop** with commonly used classes (`System`, `File`, `java.time.*`, `java.nio.*`)
* **Multi-threading** support (`pmap`, `future`)
* **Batteries included** (tools.cli, cheshire, ...)

### Non-goals

* Performance
* Provide a mixed Clojure/Bash DSL (see portability).
* Replace existing shells. Babashka is a tool you can use inside existing shells like bash and it is designed to play well with them. It does not aim to replace them.

### Setting expectations

Babashka uses [sci](https://github.com/borkdude/sci) for interpreting
Clojure. Sci implements a substantial subset of Clojure. Interpreting code is in
general not as performant as executing compiled code. If your script takes more
than a few seconds to run or has lots of loops, Clojure on the JVM may be a
better fit, since the performance of Clojure on the JVM outweighs its startup
time penalty. Read more about the differences with Clojure
[here](#differences-with-clojure).

### Talk

To get an overview of babashka, you can watch this talk ([slides](https://speakerdeck.com/borkdude/babashka-and-the-small-clojure-interpreter-at-clojured-2020)):

[![Babashka at ClojureD 2020](https://img.youtube.com/vi/Nw8aN-nrdEk/0.jpg)](https://www.youtube.com/watch?v=Nw8aN-nrdEk)

## Quickstart

For installation options check [Installation](https://github.com/borkdude/babashka#installation).
For quick installation use:

``` shell
$ bash <(curl -s https://raw.githubusercontent.com/borkdude/babashka/master/install)
```

or grab a binary from [Github
releases](https://github.com/borkdude/babashka/releases) yourself and place it
anywhere on the path.

Then you're ready to go:

``` shellsession
$ ls | bb -i '(filter #(-> % io/file .isDirectory) *input*)'
("doc" "resources" "sci" "script" "src" "target" "test")
bb took 4ms.
```

## Babashka book

Read the [babashka book](https://book.babashka.org) to learn more about how to
get the most out of babashka.

## Examples

Read the output from a shell command as a lazy seq of strings:

``` shell
$ ls | bb -i '(take 2 *input*)'
("CHANGES.md" "Dockerfile")
```

Read EDN from stdin and write the result to stdout:

``` shell
$ bb '(vec (dedupe *input*))' <<< '[1 1 1 1 2]'
[1 2]
```

Read more about input and output flags [here](doc/io-flags.md).

Execute a script. E.g. print the current time in California using the
`java.time` API:

File `pst.clj`:
``` clojure
#!/usr/bin/env bb

(def now (java.time.ZonedDateTime/now))
(def LA-timezone (java.time.ZoneId/of "America/Los_Angeles"))
(def LA-time (.withZoneSameInstant now LA-timezone))
(def pattern (java.time.format.DateTimeFormatter/ofPattern "HH:mm"))
(println (.format LA-time pattern))
```

``` shell
$ pst.clj
05:17
```

More examples can be found [here](examples/README.md).

You can try babashka online with Nextjournal's babashka [notebook
environment](http://nextjournal.com/try/babashka?cm6=1).

## Status

Functionality regarding `clojure.core` and `java.lang` can be considered stable
and is unlikely to change. Changes may happen in other parts of babashka,
although we will try our best to prevent them. Always check the release notes or
[CHANGELOG.md](CHANGELOG.md) before upgrading.

## Installation

### Brew

Linux and macOS binaries are provided via brew.

Install:

    brew install borkdude/brew/babashka

Upgrade:

    brew upgrade babashka

### Arch (Linux)

`babashka` is [available](https://aur.archlinux.org/packages/babashka-bin/) in the [Arch User Repository](https://aur.archlinux.org). It can be installed using your favorite [AUR](https://aur.archlinux.org) helper such as
[yay](https://github.com/Jguer/yay), [yaourt](https://github.com/archlinuxfr/yaourt), [apacman](https://github.com/oshazard/apacman) and [pacaur](https://github.com/rmarquis/pacaur). Here is an example using `yay`:

    yay -S babashka-bin

### Windows

On Windows you can install using [scoop](https://scoop.sh/) and the
[scoop-clojure](https://github.com/littleli/scoop-clojure) bucket.

### Installer script

Install via the installer script:

``` shell
$ curl -sLO https://raw.githubusercontent.com/borkdude/babashka/master/install
$ chmod +x install
$ ./install
```

By default this will install into `/usr/local/bin` (you may need `sudo` for
this). To change this, provide the directory name:

``` shell
$ ./install --dir /tmp
```

To install a specific version, the script also supports `--version`:

``` shell
$ ./install --dir /tmp --version 0.2.1
```

### Github releases

You may also download a binary from
[Github](https://github.com/borkdude/babashka/releases). For linux there is a
static binary available which can be used on Alpine.

## News

Check out the [news](doc/news.md) page to keep track of babashka-related news items.

## Built-in namespaces

Go [here](https://book.babashka.org/#built-in-namespaces) to see the full list of built-in namespaces.

## [Libraries, pods and projects](doc/projects.md)

A list of projects (scripts, libraries, pods and tools) known to work with babashka.

## Pods

Pods are programs that can be used as a Clojure library by
babashka. Documentation is available in the [library
repo](https://github.com/babashka/babashka.pods).

## Docker

Check out the image on [Docker hub](https://hub.docker.com/r/borkdude/babashka/).

## Differences with Clojure

Babashka is implemented using the [Small Clojure
Interpreter](https://github.com/borkdude/sci). This means that a snippet or
script is not compiled to JVM bytecode, but executed form by form by a runtime
which implements a substantial subset of Clojure. Babashka is compiled to
a native binary using [GraalVM](https://github.com/oracle/graal). It comes with
a selection of built-in namespaces and functions from Clojure and other useful
libraries. The data types (numbers, strings, persistent collections) are the
same. Multi-threading is supported (`pmap`, `future`).

Differences with Clojure:

- A pre-selected set of Java classes are supported. You cannot add Java classes
  at runtime.

- Interpretation comes with overhead. Therefore loops are slower than in Clojure
  on the JVM. In general interpretation yields slower programs than compiled
  programs.

- No `deftype`, `definterface` and unboxed math.

- `defprotocol` and `defrecord` are implemented using multimethods and regular
  maps. Ostensibly they work the same, but under the hood there are no Java
  classes that correspond to them.

- Currently `reify` works only for one class at a time

- The `clojure.core.async/go` macro is not (yet) supported. For compatibility it
  currently maps to `clojure.core.async/thread`. More info [here](#coreasync).

## Package babashka script as a AWS Lambda

AWS Lambda runtime doesn't support signals, therefore babashka has to disable
handling of SIGINT and SIGPIPE. This can be done by setting
`BABASHKA_DISABLE_SIGNAL_HANDLERS` to `true`.

## Articles, podcasts and videos

- [Writing Clojure on the Command Line with Babashka](https://youtu.be/RogyxI-GaGQ), a talk by Nate Jones.
- [Using Clojure in Command Line with Babashka](http://www.karimarttila.fi/clojure/2020/09/01/using-clojure-in-command-line-with-babashka.html), a blog article by Kari Marttila.
- [Babashka and GraalVM; taking Clojure to new places](https://youtu.be/3EUMA6bd-xQ), a talk by Michiel Borkent at [Clojure/NYC](https://www.meetup.com/Clojure-NYC/).
- [Import a CSV into Kafka, using Babashka](https://blog.davemartin.me/posts/import-a-csv-into-kafka-using-babashka/) by Dave Martin
- [Learning about babashka](https://amontalenti.com/2020/07/11/babashka), a blog article by Andrew Montalenti
- [Babashka Pods](https://www.youtube.com/watch?v=3Q4GUiUIrzg&feature=emb_logo) presentation by Michiel Borkent at the [Dutch Clojure Meetup](http://meetup.com/The-Dutch-Clojure-Meetup).
- [AWS Logs using Babashka](https://tech.toyokumo.co.jp/entry/aws_logs_babashka), a blog published by [Toyokumo](https://toyokumo.co.jp/).
- [The REPL podcast](https://www.therepl.net/episodes/36/) Michiel Borkent talks about [clj-kondo](https://github.com/borkdude/clj-kondo), [Jet](https://github.com/borkdude/jet), Babashka, and [GraalVM](https://github.com/oracle/graal) with Daniel Compton.
- [Implementing an nREPL server for babashka](https://youtu.be/0YmZYnwyHHc): impromptu presentation by Michiel Borkent at the online [Dutch Clojure Meetup](http://meetup.com/The-Dutch-Clojure-Meetup)
- [ClojureScript podcast](https://soundcloud.com/user-959992602/s3-e5-babashka-with-michiel-borkent) with Jacek Schae interviewing Michiel Borkent
- [Babashka talk at ClojureD](https://www.youtube.com/watch?v=Nw8aN-nrdEk) ([slides](https://speakerdeck.com/borkdude/babashka-and-the-small-clojure-interpreter-at-clojured-2020)) by Michiel Borkent
- [Babashka: a quick example](https://juxt.pro/blog/posts/babashka.html) by Malcolm Sparks
- [Clojure Start Time in 2019](https://stuartsierra.com/2019/12/21/clojure-start-time-in-2019) by Stuart Sierra
- [Advent of Random
  Hacks](https://lambdaisland.com/blog/2019-12-19-advent-of-parens-19-advent-of-random-hacks)
  by Arne Brasseur
- [Clojure in the Shell](https://lambdaisland.com/blog/2019-12-05-advent-of-parens-5-clojure-in-the-shell) by Arne Brasseur
- [Clojure Tool](https://purelyfunctional.tv/issues/purelyfunctional-tv-newsletter-351-clojure-tool-babashka/) by Eric Normand

## [Building babashka](doc/build.md)

## [Developing Babashka](doc/dev.md)

## Including new libraries or classes

Before new libraries or classes go into the standardly distributed babashka
binary, these evaluation criteria are considered:

- The library or class is useful for general purpose scripting.
- Adding the library or class would make babashka more compatible with Clojure
  libraries relevant to scripting.
- The library cannot be interpreted by with babashka using `--classpath`.
- The functionality can't be met by shelling out to another CLI or can't be
  written as a small layer over an existing CLI (like `babashka.curl`) instead.
- The library cannot be implemented a
  [pod](https://github.com/babashka/babashka.pods).

If not all of the criteria are met, but adding a feature is still useful to a
particular company or niche, adding it behind a feature flag is still a
possibility. This is currently the case for `next.jdbc` and the `PostgresQL` and
`HSQLDB` database drivers. Companies interested in these features can compile an
instance of babashka for their internal use. Companies are also free to make
forks of babashka and include their own internal libraries. If their customized
babashka is interesting to share with the world, they are free to distribute it
using a different binary name (like `bb-sql`, `bb-docker`, `bb-yourcompany`,
etc.). See the [feature flag documentation](doc/build.md#feature-flags) and the
implementation of the existing feature flags ([example
commit](https://github.com/borkdude/babashka/commit/02c7c51ad4b2b1ab9aa95c26a74448b138fe6659)).

## Related projects

- [planck](https://planck-repl.org/)
- [joker](https://github.com/candid82/joker)
- [closh](https://github.com/dundalek/closh)
- [lumo](https://github.com/anmonteiro/lumo)

## Examples
[A collection of example scripts](examples/README.md).

## Thanks

- [adgoji](https://www.adgoji.com/) for financial support
- [CircleCI](https://circleci.com/) for CI and additional support
- [Nikita Prokopov](https://github.com/tonsky) for the logo
- [contributors](https://github.com/borkdude/babashka/graphs/contributors) and
  other users posting issues with bug reports and ideas

## Contributors

### Code Contributors

This project exists thanks to all the people who contribute. [[Contribute](doc/dev.md)].
<a href="https://github.com/borkdude/babashka/graphs/contributors"><img src="https://opencollective.com/babashka/contributors.svg?width=890&button=false" /></a>

### Financial Contributors

Become a financial contributor and help us sustain our community. [[Contribute](https://opencollective.com/babashka/contribute)]

#### Individuals

<a href="https://opencollective.com/babashka"><img src="https://opencollective.com/babashka/individuals.svg?width=890"></a>

#### Organizations

Support this project with your organization. Your logo will show up here with a link to your website. [[Contribute](https://opencollective.com/babashka/contribute)]

<a href="https://opencollective.com/babashka/organization/0/website"><img src="https://opencollective.com/babashka/organization/0/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/1/website"><img src="https://opencollective.com/babashka/organization/1/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/2/website"><img src="https://opencollective.com/babashka/organization/2/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/3/website"><img src="https://opencollective.com/babashka/organization/3/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/4/website"><img src="https://opencollective.com/babashka/organization/4/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/5/website"><img src="https://opencollective.com/babashka/organization/5/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/6/website"><img src="https://opencollective.com/babashka/organization/6/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/7/website"><img src="https://opencollective.com/babashka/organization/7/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/8/website"><img src="https://opencollective.com/babashka/organization/8/avatar.svg"></a>
<a href="https://opencollective.com/babashka/organization/9/website"><img src="https://opencollective.com/babashka/organization/9/avatar.svg"></a>

## License

Copyright © 2019-2020 Michiel Borkent

Distributed under the EPL License. See LICENSE.

This project contains code from:
- Clojure, which is licensed under the same EPL License.
