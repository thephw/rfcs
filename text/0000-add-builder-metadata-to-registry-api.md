# Meta

[meta]: #meta

- Name: Add Builder metadata to Registry API
- Start Date: 2022.03.28
- Author(s): thephw
- Status: Draft
- RFC Pull Request: (leave blank)
- CNB Pull Request: (leave blank)
- CNB Issue: (leave blank)
- Supersedes: N/A

# Summary

[summary]: #summary

This proposal is for adding a set of Builder APIs to the buildpack registry for querying Builder metadata. This will allow Platform Implementers and Platform Operators to programattically update Builder configurations for their platforms.

# Definitions

[definitions]: #definitions

**API** - Application Programmer Interface

**Buildpack Registry** - the CNB centeralized [registry](https://github.com/buildpacks/registry-index) that contains metadata for all the community supported buildpacks

**Service** - an independent web process running in the cloud that provides RESTful endpoints

**Client** - represents a physical end-user or software interface e.g. CLI, web UI, etc.

**Image** - an OCI-compatible container image.

# Motivation

[motivation]: #motivation

As the CNB Buildpack Registry matures, Platform Operaters and Platform Implementers that wish to maintain automated updates to Builder configuration will need to query the Builder metadata to provide formatted configuration for their own platforms. Although this information is currently available in the Builder Image labels it is not as accessible as it would be as a part of the Buildpack Registry API service. Providing easier access to Builder metadata through the Buildpack Registry API will encourage Platform Operators and Platform Implementers to keep up to date with updates to builder configuration allowing them to have the latest features and security updates from CNB community buildpacks.

With Builder metadata available in the API, Platform Implementers and Platform Operators will be able to query builder metadata and create the appropriate Builder Configuration for their platform. This would enables projects like kpack or the pack CLI to programmatically retrieve metadata about the Builder Images from the Buildpacks Registry API and format the configuration as appropriate for their project. This would enable platforms to programmatically create Builder configuration from known community Builders. It would further enable platforms to programmatically update their Builder configuration to maintain the latest features, bug fixes, and security updates from the underlying Buildpacks.

# What it is

[what-it-is]: #what-it-is

An external API service that exposes endpoints for retrieving buildpack metadata against the official CNB Buildpack Registry API. It is not the goal of this RFC to add any additional UI endpoints to the Buildpacks Registry. The API endpoints will be versioned, and follow OpenAPI and/or Json Schema/API standards.

Updates to the spec for builders to include a builder name, namespace, and version in `builder.toml`

## Versioned Endpoints

All the endpoints defined below will be version via a `vN` prefix to the base URL. e.g. `https://registry.buildpacks.io/api/v1/<endpoint>`.

## MIME types

`Accept: application/vnd.buildpacks+json`

## Endpoints

### GET /builders/

This endpoint will fetch a list of builders with basic metadata. The primary use will be to return the latest builder version and a list of previous versions.

```shell
curl -H "Content-Type:application/vnd.buildpacks+json" https://registry.buildpacks.io/api/v1/builders/
```

```json
[
  {
    "description": "Builder for the Flowerwork focal stack",
    "namespace": "flowerworkio",
    "name": "focal",
    "latest": "0.0.1",
    "stack": "io.flowerwork.stacks.focal",
    "versions": ["0.0.1"]
  },
  {
    "description": "Tiny base image (bionic build image, distroless-like run image) with buildpacks for Java, Java Native Image and Go",
    "namespace": "paketo",
    "name": "tiny",
    "latest": "0.0.1",
    "stack": "io.paketo.stacks.tiny",
    "versions": ["0.0.1"]
  },
  {
    "description": "Base builder for Heroku-18 stack, based on ubuntu:18.04 base image",
    "namespace": "heroku",
    "name": "18",
    "latest": "0.0.1",
    "stack": "heroku-18",
    "versions": ["0.0.1"]
  },
  {
    "description": "Ubuntu 18 base image with buildpacks for .NET, Go, Java, Node.js, and Python",
    "namespace": "google",
    "name": "v1",
    "latest": "0.0.1",
    "stack": "google",
    "versions": ["0.0.1"]
  }
]
```

### GET /builders/:namespace/:name

This API endpoint will return the following structure:

```shell
curl -H "Content-Type:application/vnd.buildpacks+json" https://registry.buildpacks.io/api/v1/builders/flowerworkio/focal
```

```json
{
  "buildpack": {
    "layers": {
      "flowerworkio/asdf": {
        "0.0.1": {
          "api": "0.7",
          "stacks": [
            {
              "id": "io.flowerwork.stacks.focal"
            }
          ],
          "layerDiffID": "sha256:9e2ff3c946d1272ec372cd48ef5deb74ff1ad23cf46fc808df799d0f9d687f78",
          "homepage": "https://github.com/flowerworkio/buildpacks"
        }
      },
      "flowerworkio/phoenix": {
        "0.0.1": {
          "api": "0.7",
          "stacks": [
            {
              "id": "io.flowerwork.stacks.focal"
            }
          ],
          "layerDiffID": "sha256:53d8aa52bd95c203d91686cf79634fea38b53e25b5881e2d02c68ff011041922",
          "homepage": "https://github.com/flowerworkio/buildpacks"
        }
      }
    },
    "order": [
      {
        "group": [
          {
            "id": "flowerworkio/asdf",
            "version": "0.0.1"
          },
          {
            "id": "flowerworkio/phoenix",
            "version": "0.0.1"
          }
        ]
      }
    ]
  },
  "builder": {
    "metadata": {
      "description": "",
      "buildpacks": [
        {
          "homepage": "https://github.com/flowerworkio/buildpacks",
          "id": "flowerworkio/asdf",
          "name": "asdf",
          "version": "0.0.1"
        },
        {
          "homepage": "https://github.com/flowerworkio/buildpacks",
          "id": "flowerworkio/phoenix",
          "name": "phoenix",
          "version": "0.0.1"
        }
      ],
      "stack": {
        "runImage": {
          "image": "flowerworkio/run:focal",
          "mirrors": null
        }
      },
      "lifecycle": {
        "version": "0.13.2",
        "api": {
          "buildpack": "0.2",
          "platform": "0.3"
        },
        "apis": {
          "buildpack": {
            "deprecated": [],
            "supported": ["0.2", "0.3", "0.4", "0.5", "0.6", "0.7"]
          },
          "platform": {
            "deprecated": [],
            "supported": ["0.3", "0.4", "0.5", "0.6", "0.7", "0.8"]
          }
        }
      },
      "createdBy": {
        "name": "Pack CLI",
        "version": "0.21.1+git-e09e397.build-2823"
      }
    }
  },
  "stack": {
    "id": "io.flowerwork.stacks.focal",
    "mixins": null
  }
}
```

Another example using an existing builder:

```shell
curl -H "Content-Type:application/vnd.buildpacks+json" https://registry.buildpacks.io/api/v1/builders/heroku/18
```

```json
{
  "builder": {
    "metadata": {
      "description": "Base builder for Heroku-18 stack, based on ubuntu:18.04 base image",
      "buildpacks": [
        {
          "id": "heroku/java",
          "name": "Java",
          "version": "0.3.15",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/jvm",
          "name": "JVM",
          "version": "0.1.14",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/maven",
          "name": "Maven",
          "version": "0.2.6",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/procfile",
          "version": "0.6.2"
        },
        {
          "id": "heroku/gradle",
          "version": "0.0.35",
          "homepage": "https://github.com/heroku/heroku-buildpack-gradle"
        },
        {
          "id": "heroku/scala",
          "name": "Scala",
          "version": "0.0.92",
          "homepage": "https://github.com/heroku/heroku-buildpack-scala"
        },
        {
          "id": "heroku/java-function",
          "name": "Java Function",
          "version": "0.3.28",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/jvm",
          "name": "JVM",
          "version": "0.1.14",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/jvm-function-invoker",
          "name": "JVM Function Invoker",
          "version": "0.6.1",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/maven",
          "name": "Maven",
          "version": "0.2.6",
          "homepage": "https://github.com/heroku/buildpacks-jvm"
        },
        {
          "id": "heroku/ruby",
          "version": "0.1.3"
        },
        {
          "id": "heroku/procfile",
          "version": "0.6.2"
        },
        {
          "id": "heroku/python",
          "name": "Python",
          "version": "0.3.1"
        },
        {
          "id": "heroku/php",
          "name": "PHP",
          "version": "0.3.1"
        },
        {
          "id": "heroku/go",
          "name": "Go",
          "version": "0.3.1"
        },
        {
          "id": "heroku/nodejs",
          "name": "Node.js",
          "version": "0.4.1",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        },
        {
          "id": "heroku/nodejs-engine",
          "name": "Node Buildpack",
          "version": "0.7.5",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        },
        {
          "id": "heroku/nodejs-npm",
          "name": "NPM Buildpack",
          "version": "0.4.5",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        },
        {
          "id": "heroku/nodejs-yarn",
          "name": "Yarn Buildpack",
          "version": "0.1.8"
        },
        {
          "id": "heroku/procfile",
          "version": "0.6.2"
        },
        {
          "id": "heroku/nodejs-function",
          "name": "Node.js Function",
          "version": "0.7.3",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        },
        {
          "id": "heroku/nodejs-engine",
          "name": "Node Buildpack",
          "version": "0.7.5",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        },
        {
          "id": "heroku/nodejs-function-invoker",
          "name": "Node.js Function Invoker",
          "version": "0.2.10",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        },
        {
          "id": "heroku/nodejs-npm",
          "name": "NPM Buildpack",
          "version": "0.4.5",
          "homepage": "https://github.com/heroku/buildpacks-nodejs"
        }
      ],
      "stack": {
        "runImage": {
          "image": "heroku/pack:18",
          "mirrors": null
        }
      },
      "lifecycle": {
        "version": "0.13.1",
        "api": {
          "buildpack": "0.2",
          "platform": "0.3"
        },
        "apis": {
          "buildpack": {
            "deprecated": [],
            "supported": ["0.2", "0.3", "0.4", "0.5", "0.6", "0.7"]
          },
          "platform": {
            "deprecated": [],
            "supported": ["0.3", "0.4", "0.5", "0.6", "0.7", "0.8"]
          }
        }
      },
      "createdBy": {
        "name": "Pack CLI",
        "version": "0.23.0+git-0db2c77.build-3056"
      }
    }
  },
  "buildpack": {
    "layers": {
      "heroku/go": {
        "0.3.1": {
          "api": "0.4",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            }
          ],
          "layerDiffID": "sha256:16788c5b718cd8d5c65e11170ec7c5a7006a71c307db2cdee3f9d9f5669bb295",
          "name": "Go"
        }
      },
      "heroku/gradle": {
        "0.0.35": {
          "api": "0.4",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:378c11869b77a85c04824dddbbd55ebd041936f09a6b20af02be4a0b0b9a8833",
          "homepage": "https://github.com/heroku/heroku-buildpack-gradle"
        }
      },
      "heroku/java": {
        "0.3.15": {
          "api": "0.4",
          "order": [
            {
              "group": [
                {
                  "id": "heroku/jvm",
                  "version": "0.1.14"
                },
                {
                  "id": "heroku/maven",
                  "version": "0.2.6"
                },
                {
                  "id": "heroku/procfile",
                  "version": "0.6.2",
                  "optional": true
                }
              ]
            },
            {
              "group": [
                {
                  "id": "heroku/gradle",
                  "version": "0.0.35"
                },
                {
                  "id": "heroku/procfile",
                  "version": "0.6.2",
                  "optional": true
                }
              ]
            }
          ],
          "layerDiffID": "sha256:9152bc36b91919d244fc0874a7d5c70fcafcd8bbab26ada7ba05613e226c1581",
          "homepage": "https://github.com/heroku/buildpacks-jvm",
          "name": "Java"
        }
      },
      "heroku/java-function": {
        "0.3.28": {
          "api": "0.4",
          "order": [
            {
              "group": [
                {
                  "id": "heroku/jvm",
                  "version": "0.1.14"
                },
                {
                  "id": "heroku/maven",
                  "version": "0.2.6"
                },
                {
                  "id": "heroku/jvm-function-invoker",
                  "version": "0.6.1"
                }
              ]
            }
          ],
          "layerDiffID": "sha256:b501711572986fb34868b057d5ce34445c5f66ce319018d3d6b5ff0b7701648f",
          "homepage": "https://github.com/heroku/buildpacks-jvm",
          "name": "Java Function"
        }
      },
      "heroku/jvm": {
        "0.1.14": {
          "api": "0.4",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:58dc6f2c3f2c952624319382d5714958125083e3741898d05cb58ea4f612cd34",
          "homepage": "https://github.com/heroku/buildpacks-jvm",
          "name": "JVM"
        }
      },
      "heroku/jvm-function-invoker": {
        "0.6.1": {
          "api": "0.6",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:c2cca2fa97b0e82aa11a91b279626b1f41a9c84f4745c08d0efe0a96889c2c7e",
          "homepage": "https://github.com/heroku/buildpacks-jvm",
          "name": "JVM Function Invoker"
        }
      },
      "heroku/maven": {
        "0.2.6": {
          "api": "0.5",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            },
            {
              "id": "*"
            }
          ],
          "layerDiffID": "sha256:799c9660c5e4b1dcd8604f1935b81a9b0fab8252541557dc50c3f4e9eb4b26d9",
          "homepage": "https://github.com/heroku/buildpacks-jvm",
          "name": "Maven"
        }
      },
      "heroku/nodejs": {
        "0.4.1": {
          "api": "0.2",
          "order": [
            {
              "group": [
                {
                  "id": "heroku/nodejs-engine",
                  "version": "0.7.5"
                },
                {
                  "id": "heroku/nodejs-yarn",
                  "version": "0.1.8"
                },
                {
                  "id": "heroku/procfile",
                  "version": "0.6.2",
                  "optional": true
                }
              ]
            },
            {
              "group": [
                {
                  "id": "heroku/nodejs-engine",
                  "version": "0.7.5"
                },
                {
                  "id": "heroku/nodejs-npm",
                  "version": "0.4.5"
                },
                {
                  "id": "heroku/procfile",
                  "version": "0.6.2",
                  "optional": true
                }
              ]
            }
          ],
          "layerDiffID": "sha256:9df224c4c414c1de91d5e30a1845fb7137b890900fe4da6bcf4dea931498f4d9",
          "homepage": "https://github.com/heroku/buildpacks-nodejs",
          "name": "Node.js"
        }
      },
      "heroku/nodejs-engine": {
        "0.7.5": {
          "api": "0.2",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:c50704c00a3f0da071cb800d6e10771f5cfc617d3aa2770dcea06fbc0d2b0aa0",
          "homepage": "https://github.com/heroku/buildpacks-nodejs",
          "name": "Node Buildpack"
        }
      },
      "heroku/nodejs-function": {
        "0.7.3": {
          "api": "0.2",
          "order": [
            {
              "group": [
                {
                  "id": "heroku/nodejs-engine",
                  "version": "0.7.5"
                },
                {
                  "id": "heroku/nodejs-npm",
                  "version": "0.4.5"
                },
                {
                  "id": "heroku/nodejs-function-invoker",
                  "version": "0.2.10"
                }
              ]
            }
          ],
          "layerDiffID": "sha256:79ee8fc9d507fceb5044e16ec5341c74046614e384f7f7ca1a78e886c04052be",
          "homepage": "https://github.com/heroku/buildpacks-nodejs",
          "name": "Node.js Function"
        }
      },
      "heroku/nodejs-function-invoker": {
        "0.2.10": {
          "api": "0.6",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:bbbf399f670279797a8cf31ef9749b5de9176ff74b6439e4cb1b53441926cf58",
          "homepage": "https://github.com/heroku/buildpacks-nodejs",
          "name": "Node.js Function Invoker"
        }
      },
      "heroku/nodejs-npm": {
        "0.4.5": {
          "api": "0.2",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:cbc374b1478b2282c0342d676197bd72a4dabd309aa0255d16178830dffc19c1",
          "homepage": "https://github.com/heroku/buildpacks-nodejs",
          "name": "NPM Buildpack"
        }
      },
      "heroku/nodejs-yarn": {
        "0.1.8": {
          "api": "0.6",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            },
            {
              "id": "*"
            }
          ],
          "layerDiffID": "sha256:99dd054bfb5ed12a839bb3760c6ca48ed021b079cb020b2e7837715d2c40f1b4",
          "name": "Yarn Buildpack"
        }
      },
      "heroku/php": {
        "0.3.1": {
          "api": "0.4",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            }
          ],
          "layerDiffID": "sha256:7399e62017047d4774011222f2c88d3528a561055cd457c96bae8465144ee6cd",
          "name": "PHP"
        }
      },
      "heroku/procfile": {
        "0.6.2": {
          "api": "0.2",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:c794dfb8f6205cd8473f3303584065426775a365c6cabc6713a45eea0bff33e5"
        }
      },
      "heroku/python": {
        "0.3.1": {
          "api": "0.4",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            }
          ],
          "layerDiffID": "sha256:51aac3e8421399c7b40e4fb039969545c30c039bbbed898ccc582fae44a6bec1",
          "name": "Python"
        }
      },
      "heroku/ruby": {
        "0.1.3": {
          "api": "0.2",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            }
          ],
          "layerDiffID": "sha256:32dd1972a7852a8a24f74055c86027296df4cc2a582e0b7e59d5659347da4207"
        }
      },
      "heroku/scala": {
        "0.0.92": {
          "api": "0.4",
          "stacks": [
            {
              "id": "heroku-18"
            },
            {
              "id": "heroku-20"
            },
            {
              "id": "io.buildpacks.stacks.bionic"
            }
          ],
          "layerDiffID": "sha256:63b6b9b0efef35fa683b3472a56b90f335f8fad8a77c4a127f2e5b9b49d8e340",
          "homepage": "https://github.com/heroku/heroku-buildpack-scala",
          "name": "Scala"
        }
      }
    },
    "order": [
      {
        "group": [
          {
            "id": "heroku/ruby",
            "version": "0.1.3"
          },
          {
            "id": "heroku/procfile",
            "version": "0.6.2",
            "optional": true
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/python",
            "version": "0.3.1"
          },
          {
            "id": "heroku/procfile",
            "version": "0.6.2",
            "optional": true
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/scala",
            "version": "0.0.92"
          },
          {
            "id": "heroku/procfile",
            "version": "0.6.2",
            "optional": true
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/php",
            "version": "0.3.1"
          },
          {
            "id": "heroku/procfile",
            "version": "0.6.2",
            "optional": true
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/go",
            "version": "0.3.1"
          },
          {
            "id": "heroku/procfile",
            "version": "0.6.2",
            "optional": true
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/nodejs-function",
            "version": "0.7.3"
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/java-function",
            "version": "0.3.28"
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/nodejs",
            "version": "0.4.1"
          }
        ]
      },
      {
        "group": [
          {
            "id": "heroku/java",
            "version": "0.3.15"
          }
        ]
      }
    ]
  },

  "stack": {
    "id": "heroku-18",
    "mixins": null
  }
}
```

# How it Works

[how-it-works]: #how-it-works

This is the technical portion of the RFC, where you explain the design in sufficient detail.
The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.
- Should we do another repo for builder with a human readable index? How well is this working for buildpacks?
- We could piggy back namespace ownership in this repository as well
- Using a builder from the registry
- Looking up a builder in the registry
- Yanking a builder from the registry
- Buildpack author experience
- Platform implementer experience
- Platform maintainer experience


# Migration

[migration]: #migration

This section should document breaks to public API and breaks in compatibility due to this RFC's proposed changes. In addition, it should document the proposed steps that one would need to take to work through these changes. Care should be give to include all applicable personas, such as platform developers, buildpack developers, buildpack users and consumers of buildpack images.

# Drawbacks

[drawbacks]: #drawbacks

Why should we _not_ do this?

# Alternatives

[alternatives]: #alternatives

- What other designs have been considered?
- Why is this proposal the best?
- What is the impact of not doing this?

# Prior Art

[prior-art]: #prior-art

- [Heroku Buildpack Registry](https://devcenter.heroku.com/articles/buildpack-registry)
- [Cargo](https://doc.rust-lang.org/cargo/) and [crates.io](https://crates.io/)

# Unresolved Questions

[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to be resolved before this gets merged?
- What parts of the design do you expect to be resolved through implementation of the feature?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Spec. Changes (OPTIONAL)

[spec-changes]: #spec-changes

Does this RFC entail any proposed changes to the core specifications or extensions? If so, please document changes here.
Examples of a spec. change might be new lifecycle flags, new `buildpack.toml` fields, new fields in the buildpackage label, etc.
This section is not intended to be binding, but as discussion of an RFC unfolds, if spec changes are necessary, they should be documented here.
