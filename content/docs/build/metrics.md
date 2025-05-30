---
title: Build Controller Metrics
linkTitle: Metrics
---
<!--
Copyright The Shipwright Contributors

SPDX-License-Identifier: Apache-2.0
-->

The Build component exposes several metrics to help you monitor the health and behavior of your build resources.

Following build metrics are exposed on port `8383`.

| Name                                                 | Type      | Description                                       | Labels                                                                                                                                                                           | Status       |
|:-----------------------------------------------------|:----------|:--------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------|
| `build_builds_registered_total`                      | Counter   | Number of total registered Builds.                | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup>                                          | experimental |
| `build_buildruns_completed_total`                    | Counter   | Number of total completed BuildRuns.              | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup><br>buildrun=<buildrun_name> <sup>1</sup> | experimental |
| `build_buildrun_establish_duration_seconds`          | Histogram | BuildRun establish duration in seconds.           | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup><br>buildrun=<buildrun_name> <sup>1</sup> | experimental |
| `build_buildrun_completion_duration_seconds`         | Histogram | BuildRun completion duration in seconds.          | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup><br>buildrun=<buildrun_name> <sup>1</sup> | experimental |
| `build_buildrun_rampup_duration_seconds`             | Histogram | BuildRun ramp-up duration in seconds              | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup><br>buildrun=<buildrun_name> <sup>1</sup> | experimental |
| `build_buildrun_taskrun_rampup_duration_seconds`     | Histogram | BuildRun taskrun ramp-up duration in seconds.     | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup><br>buildrun=<buildrun_name> <sup>1</sup> | experimental |
| `build_buildrun_taskrun_pod_rampup_duration_seconds` | Histogram | BuildRun taskrun pod ramp-up duration in seconds. | buildstrategy=<build_buildstrategy_name> <sup>1</sup><br>namespace=<buildrun_namespace> <sup>1</sup><br>build=<build_name> <sup>1</sup><br>buildrun=<buildrun_name> <sup>1</sup> | experimental |

<sup>1</sup> Labels for metric are disabled by default. See [Configuration of metric labels](#configuration-of-metric-labels) to enable them.

## Configuration of histogram buckets

Environment variables can be set to use custom buckets for the histogram metrics:

| Metric                                               | Environment variable               | Default                                  |
|------------------------------------------------------|------------------------------------|------------------------------------------|
| `build_buildrun_establish_duration_seconds`          | `PROMETHEUS_BR_EST_DUR_BUCKETS`    | `0,1,2,3,5,7,10,15,20,30`                |
| `build_buildrun_completion_duration_seconds`         | `PROMETHEUS_BR_COMP_DUR_BUCKETS`   | `50,100,150,200,250,300,350,400,450,500` |
| `build_buildrun_rampup_duration_seconds`             | `PROMETHEUS_BR_RAMPUP_DUR_BUCKETS` | `0,1,2,3,4,5,6,7,8,9,10`                 |
| `build_buildrun_taskrun_rampup_duration_seconds`     | `PROMETHEUS_BR_RAMPUP_DUR_BUCKETS` | `0,1,2,3,4,5,6,7,8,9,10`                 |
| `build_buildrun_taskrun_pod_rampup_duration_seconds` | `PROMETHEUS_BR_RAMPUP_DUR_BUCKETS` | `0,1,2,3,4,5,6,7,8,9,10`                 |

The values have to be a comma-separated list of numbers. You need to set the environment variable for the build controller for your customization to become active. When running locally, set the variable right before starting the controller:

```bash
export PROMETHEUS_BR_COMP_DUR_BUCKETS=30,60,90,120,180,240,300,360,420,480
make local
```

When you deploy the build controller in a Kubernetes cluster, you need to extend the `spec.containers[0].spec.env` section of the sample deployment file, [controller.yaml](https://github.com/shipwright-io/build/blob/v0.15.2/deploy/500-controller.yaml). Add another entry:

```yaml
[...]
  env:
  - name: PROMETHEUS_BR_COMP_DUR_BUCKETS
    value: "30,60,90,120,180,240,300,360,420,480"
[...]
```

## Configuration of metric labels

As the amount of buckets and labels has a direct impact on the number of Prometheus time series, you can selectively enable labels that you are interested in using the `PROMETHEUS_ENABLED_LABELS` environment variable. The supported labels are:

* buildstrategy
* namespace
* build
* buildrun

Use a comma-separated value to enable multiple labels. For example:

```bash
export PROMETHEUS_ENABLED_LABELS=namespace
make local
```

or

```bash
export PROMETHEUS_ENABLED_LABELS=buildstrategy,namespace,build
make local
```

When you deploy the build controller in a Kubernetes cluster, you need to extend the `spec.containers[0].spec.env` section of the sample deployment file, [controller.yaml](https://github.com/shipwright-io/build/blob/v0.15.2/deploy/500-controller.yaml). Add another entry:

```yaml
[...]
  env:
  - name: PROMETHEUS_ENABLED_LABELS
    value: namespace
[...]
```
