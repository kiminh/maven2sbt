# maven2sbt

[![Build Status](https://github.com/Kevin-Lee/maven2sbt/workflows/Build%20All/badge.svg)](https://github.com/Kevin-Lee/maven2sbt/actions?workflow=Build+All)
[![Release Status](https://github.com/Kevin-Lee/maven2sbt/workflows/Release/badge.svg)](https://github.com/Kevin-Lee/maven2sbt/actions?workflow=Release)

[![Download](https://api.bintray.com/packages/kevinlee/maven/maven2sbt-core/images/download.svg)](https://bintray.com/kevinlee/maven/maven2sbt-core/_latestVersion)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.kevinlee/maven2sbt-core_2.13/badge.svg)](https://search.maven.org/artifact/io.kevinlee/maven2sbt-core_2.13)
[![Latest version](https://index.scala-lang.org/kevin-lee/maven2sbt/maven2sbt-core/latest.svg)](https://index.scala-lang.org/kevin-lee/maven2sbt/maven2sbt-core)

A tool to convert Maven `pom.xml` into sbt `build.sbt`


# Standalone CLI App

It requires Java 8 or higher. So JRE should be installed and available to run `maven2sbt-cli`.

## Debian / Ubuntu Linux
If you use Debian or Unbuntu Linux you can download [maven2sbt-cli_0.2.0_all.deb](https://github.com/Kevin-Lee/maven2sbt/releases/download/v0.2.0/maven2sbt-cli_0.2.0_all.deb) and install it.
```shell
$ dpkg -i maven2sbt-cli_0.2.0_all.deb 
```
`maven2sbt-cli` should be available.
e.g.)
```shell
$ which maven2sbt-cli
/usr/bin/maven2sbt-cli
```


## Linux / macOS
### Use `curl`
```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Kevin-Lee/maven2sbt/master/.scripts/install.sh)" 
```
***

### Or use `wget`
```shell
sh -c "$(wget -O- https://raw.githubusercontent.com/Kevin-Lee/maven2sbt/master/.scripts/install.sh)" 
```
***

### Or do it manually (not recommended)
  
Download [maven2sbt-cli-0.2.0.zip](https://github.com/Kevin-Lee/maven2sbt/releases/download/v0.2.0/maven2sbt-cli-0.2.0.zip) or [maven2sbt-cli-0.2.0.tgz](https://github.com/Kevin-Lee/maven2sbt/releases/download/v0.2.0/maven2sbt-cli-0.2.0.tgz) and unzip it.
  
Add an alias for convenience to `~/.zshrc` or `~/.bashrc` or `~/.bash_profile` or the run commands file for your shell. 
```shell
alias maven2sbt-cli='/path/to/maven2sbt-cli-0.2.0/bin/maven2sbt-cli'
```


## Windows

Download and unzip the maven2sbt-cli-0.2.0.zip or maven2sbt-cli-0.2.0.tgz just like Linux or macOS.

You can run `maven2sbt-cli-0.2.0/bin/maven2sbt-cli.bat` file but it hasn't been tested.


# How to Use

Now you can run it like

```shell
$ maven2sbt-cli --help 

Usage:
  Maven2Sbt -s|--scala-version <version> [-o|--out <file>] [--overwrite] <pom-path>
            [-v|--version VERSION] [-h|--help HELP]

A tool to convert Maven pom.xml into sbt build.sbt

Available options:
  --overwrite             Overwrite if the output file already exists.
  -s|--scala-version <version> Scala version
  -o|--out <file>         output sbt config file (default: build.sbt)
  -v|--version VERSION    Prints the application version.
  -h|--help HELP          Prints the synopsis and a list of options and arguments.

Positional arguments:
  <pom-path>              Path to the pom file.

```

e.g.)
```shell
$ maven2sbt-cli --scala-version 2.13.1 pom.xml
# or
$ maven2sbt-cli -s 2.13.1 pom.xml
```
It will generate `build.sbt` may contain content like below
```sbt
val projectBuildSourceEncoding = "UTF-8"
val javaVersion = "1.8"
val junitVersion = "4.11"
val test0ster1Version = "0.0.6"

ThisBuild / organization := "io.kevinlee"
ThisBuild / version := "0.0.11"
ThisBuild / scalaVersion := "2.13.1"
  
lazy val root = (project in file("."))
  .settings(
    name := "j8plus"
  , resolvers ++= Seq(
      "Kevin's Public Releases" at "http://dl.bintray.com/kevinlee/maven"
    , "Kevin's Public Releases 2" at "http://blah.blah.blah/maven"
    )
  , libraryDependencies ++= Seq(
      "junit" % "junit" % "junitVersion" % Test
    , "io.kevinlee" % "test0ster1" % "test0ster1Version" % Test
    , "io.kevinlee" % "nothing" % "0.0.1"
    , "org.assertj" % "assertj-core" % "1.5.0" % Test
    , "some.project" % "the-project1" % "1.9.5" % Test exclude("another.project", "another-project")
    , "some.project" % "the-project2" % "2.10.1" excludeAll(
        ExclusionRule(organization = "blah.blah", artifact = "abc")
      , ExclusionRule(organization = "some.test", artifact = "project-123")
      )
    )
  )
```

Save sbt config in a different file.
```shell
maven2sbt-cli --scala-version 2.13.1 --out something-else.sbt pom.xml
# or
maven2sbt-cli -s 2.13.1 -o something-else.sbt pom.xml
```
It will generate `something-else.sbt`.

I may faile if the output file already eixsts. If you want to overwrite, use the `--overwrite` option.

```shell
# build.sbt already exists and want to overwrite
maven2sbt-cli --scala-version 2.13.1 --overwrite pom.xml
# or
maven2sbt-cli -s 2.13.1 --overwrite pom.xml

# something-else.sbt already exists and want to overwrite
maven2sbt-cli --scala-version 2.13.1 --out something-else.sbt --overwrite pom.xml
# or
maven2sbt-cli -s 2.13.1 -o something-else.sbt --overwrite pom.xml
```

# Use As Library

`maven2sbt-core` supports Scala 2.10, 2.11, 2.12 and 2.13.

 
## Get maven2sbt-core lib

Add it to `build.sbt`.
```sbt
libraryDependencies += "io.kevinlee" %% "maven2sbt-core" % "0.2.0"
```


## How to Use

```scala
import java.io.File

import maven2sbt.core.Maven2Sbt
import maven2sbt.core.ScalaVersion

Maven2Sbt.buildSbtFromPomFile(ScalaVersion("2.13.1", new File("/path/to/pom.xml")))

// or

Maven2Sbt.buildSbtFromInputStream(ScalaVersion("2.13.1", inputStream))
```
