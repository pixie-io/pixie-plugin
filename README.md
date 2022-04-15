# Pixie Community Plugins

This repository is for community-submitted plugins for the [Pixie Plugin system](https://docs.px.dev/reference/plugins/plugin-system/). The Pixie Plugin system allows other tools to provide additional capabilities within Pixie:

- Longterm Data Retention: Pixie is designed for a real-time debugging use-case and does not guarantee data storage for over 24 hours. Leverage an external datastore for longterm data retention by sending Pixie data in the [OpenTelemetry](https://opentelemetry.io/) format. Future support will be added for querying longterm data from within the Pixie UI, allowing you to use PxL and Pixie's versatile live views.
- Alerts (Coming Soon!): Power alerts using Pixie's rich dataset, all configurable from within Pixie's UI.

For any questions, reach out to us on [Slack](https://slackin.px.dev).

## Submitting a plugin

Before submitting a plugin, please review our [Code of Conduct](https://github.com/pixie-io/pixie/blob/main/CODE_OF_CONDUCT.md).

Plugins should be submitted as a pull request to this repository. The pull request will be carefully reviewed by the Pixie maintainers to ensure it meets quality and security standards.

## Plugin Format 

Plugins should be added as a directory within `/plugins` directory. The name of the directory should match the plugin's ID. For example: `/plugins/cool-plugin` for a plugin with the ID `cool-plugin`.
A plugin consists of the following:

### plugin.yaml

This YAML contains general configuration information about the plugin. This contains basic metadata which informs how the plugin is presented within Pixie and which features it supports.

| **Field**            | **Type** | **Required** | **Description**                                             |
|----------------------|----------|--------------|-------------------------------------------------------------|
| name                 | string   | true         | Human-readable name for the plugin                          |
| id                   | string   | true         | Unique identifier for the plugin                            |
| description          | string   | true         | A description about the plugin                              |
| logo                 | object   | false        | SVG images that should be used for the plugin icon          |
| version              | string   | true         | Version of the plugin. Should follow semVer.                |
| updated              | string   | true         | Date when the plugin version was released                   |
| keywords             | string[] | false        | Keywords that can be used to search for the plugin          |
| dataRetentionEnabled | boolean  | true         | Whether or not this plugin supports longterm data retention |

### retention.yaml

This should be provided only if the plugin supports longterm data retention.


| **Field**            | **Type**          | **Required** | **Description**                                                                                                                                                                                                                                                           |
|----------------------|-------------------|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| documentationURL     | string            | false        | URL that can be used to direct user to documentation about data retention (on the plugin provider’s side)                                                                                                                                                                 |
| configurations       | map[string]string | false        | A list of configurable values that should be presented to the user upon enabling the plugin.  Key: The name of the configuration field. Value: A description for the configuration field.  These fields will be attached to the context metadata in the GRPC export call. |
| defaultExportURL     | string            | true         | URL which the OTel data should be exported to. This is expected to host the OTel GRPC service.                                                                                                                                                                            |
| allowCustomExportURL | boolean           | false        | Let the user provide a custom export URL.                                                                                                                                                                                                                                 |
| allowInsecureTLS | boolean           | false        | Allow the user to connect to the export URL with an insecure SSL connection.                                                                                                                                                                                                                                |
| presetScripts        | object[]          | false        | A set of scripts, prewritten by the plugin provider, which they can use to provide extra functionality.                                                                                                                                                                   |

*Note: We recommend that sensitive configuration fields, such as API keys, be scoped to only required permissions for writing to the datastore.*

The presetScripts object should adhere to the following format:

| **Field**         | **Type** | **Required** | **Description**                                                                                                                       |
|-------------------|----------|--------------|---------------------------------------------------------------------------------------------------------------------------------------|
| name              | string   | true         | The name of the script, displayed to the user.                                                                                        |
| description       | string   | false        | A description about what the script exports, and the extra functionality it powers.                                                   |
| script            | string   | true         | The PxL script.                                                                                                                       |
| defaultFrequencyS | int      | false        | The default interval, in seconds, at which the script should be rerun. The user has the ability to change this interval if they wish. |

## Testing a Plugin

Your plugin should be manually tested prior to submission.

1. Create a fork of this repo.
2. Add the necessary files for your plugin (See [Plugin Format](#plugin-format) above).
3. Fork the official [Pixie repo](https://github.com/pixie-io/pixie).
4. Update `k8s/cloud/public/plugin_db_updater_job.yaml`'s `PL_PLUGIN_REPO` environment variable to point to your plugin repo fork.
5. Deploy self-hosted Pixie Cloud by following this [guide](https://docs.px.dev/installing-pixie/install-guides/self-hosted-pixie/). These steps will automatically pull your plugin into Pixie Cloud's database.
6. Verify that your plugin appears in the `Plugins` page in the Admin UI.
7. Deploy a Vizier, enable the plugin, and validate that behavior is as expected.
