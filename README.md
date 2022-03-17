# typos-json-to-checkstyle

A CLI tool that takes [typos](https://github.com/crate-ci/typos)'s json output and converts it to checkstyle.xml so it can be injested from other tools like [warning-ng](https://plugins.jenkins.io/warnings-ng)

## Installation

Get latest version from github releases

## Usages

```
 typos --format json | ./target/debug/typos-checkstyle - > checkstyle.xml
```

## Release

```
cargo release [patch|minor|major]
```
