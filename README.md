# typos-json-to-checkstyle

A CLI tool that takes [typos](https://github.com/crate-ci/typos)'s json output and converts it to checkstyle.xml so it can be injested from other tools like [warning-ng](https://plugins.jenkins.io/warnings-ng)

## Installation

Get latest version from github releases

## Usages

```bash
curl -qsL https://github.com/halkeye/typos-json-to-checkstyle/releases/download/v0.1.1/typos-checkstyle-v0.1.1-x86_64 > typos-checkstyle && chmod 0755 typos-checkstyle
typos --format json | ./typos-checkstyle - > checkstyle.xml
```

## Release

```bash
cargo release [patch|minor|major]
```
