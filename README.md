![Tidepool Logo](assets/images/Tidepool_Logo_Light_Large.png)

[![Build Status](https://travis-ci.com/tidepool-org/TidepoolApi.svg?branch=master)](https://travis-ci.com/tidepool-org/TidepoolApi)

# TidepoolApi

This repository contains Tidepool Platform API documentation in [OpenAPI v3](https://www.openapis.org/) format with additional narrative content in [Stoplight-flavored](https://meta.stoplight.io/docs/studio/docs/Documentation/03a-stoplight-flavored-markdown.md) [CommonMark](https://commonmark.org/) format which in turn is a less ambiguous formal definition of [Markdown](https://www.markdownguide.org/).
These API definitions can be used to generate stub code for either server or client side.

We have an account with [Stoplight Next](https://next.stoplight.io/tidepool) where we can publish and host API specifications with a nice web UI.
However, that service will reach end-of-life on Dec 31, 2021. We will either migrate to the new [Stoplight Platform](https://tidepool.stoplight.io) solution, or possibly leverage the open-source [Stoplight Elements](https://stoplight.io/open-source/elements/) renderer to embed the documentation directly into Tidepool's [Developer](https://developer.tidepool.org) portal.

A good directory for OpenAPI tools can be at https://openapi.tools/.

This repo contains a `Rakefile` that can automate several tasks such as validating Markdown and OpenAPI 3 specifications, and generate code stubs clients and servers. Use `rake -T` to see a list of available tasks, or `rake -P` to see the dependency tree.

## Recommended Editing Tools

* [Stoplight Studio](https://stoplight.io/studio/) is a great tool for offline editing OpenAPI v3 specifications.
* Microsoft [Visual Studio Code](https://code.visualstudio.com/) and GitHub [Atom](https://atom.io/) offer lots of plug-ins for validating and rendering OpenAPI v3 specifications.

## Recommended Validation Tools

* [Stoplight Spectral](https://stoplight.io/open-source/spectral/) is embedded in Stoplight Studio and Stoplight Platform, so they work well together.
* [Swagger CLI](https://github.com/APIDevTools/swagger-cli) also validates OpenAPI v3 specifications. It can be used to bundle multiple files into a single combined file. Some other tools do not handle `$ref` properly, so this may be a required preprocessing step:
  > `$ swagger-cli bundle --dereference --type yaml --outfile <outfile> <infile>`
* [Markdownlint](https://github.com/igorshubovych/markdownlint-cli) validates Markdown files.

## Recommended Code Generation Tools

* [oapi-codegen](https://github.com/deepmap/oapi-codegen) can generate server stubs in [Go](https://golang.org/). It requires bundled OpenAPI v3 specification (see above).
* [openapi-generator](https://github.com/OpenAPITools/openapi-generator) can generate client SDKs, and server stubs and documentation. It requires bundled OpenAPI v3 specification (see above).
