---
title: "LadybugDB v0.12.0 Release"
description: "LadybugDB v0.12.0 Release"
pubDate: "Nov 5 2025"
categories: ["release"]
heroImage: "../../../public/img/2025-11-05-release/release.png"
authors: ["team"]
tags: ["release"]
---
This the first [release](https://github.com/LadybugDB/ladybug/releases/tag/v0.12.0) of LadybugDB after announcing the fork of kuzu a few weeks ago. The focus of this release cycle has been to move the CI/CD pipelines from self-hosted infra to widely used GitHub runners.

A top concern for our users is continuity—what happens if LadybugDB the project or the people working on it can't continue supporting it? Adopting a well-understood CI/CD pipeline helps address this risk.

The functionality is equivalent to kuzu v0.11.3. The only change would be to rename kuzu to the correct package name in your language. Please refer to [documentation](https://docs.ladybugdb.com/) for examples.

## Future Direction

By dropping the CLA and letting contributors retain copyright, we are signaling the intention of running the project as an open community project. Please start reviewing code and start participating in design discussions. After a period of sustained contributions, expect yourself to be added to the org as a collaborator.

## What about Other Forks?

We fully intend to collaborate with any other fork of kuzu and incorporate ideas and bug fixes to the extent possible.

## Next Steps

We will follow up with a v0.12.1 release which will fix a number of gaps:

* npmjs packages
* wasm build
* extensions
* High priority bug fixes
* ladybug-explorer visualization

Looking for help with Go and Swift packages. We will also work with the maintainers of other popular packages that use kuzu as a graph storage backend to migrate over to ladybug.
