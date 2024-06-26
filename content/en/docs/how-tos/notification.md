---
title: "Set Up Slack Alerts for Periodic Job Results"
description: How to set up alerting to Slack for a CI job.
---

We can set up Slack alerts for job results by defining `reporter_config` in the job definition.

```yaml
reporter_config:
  slack:
    channel: '#forum'
    job_states_to_report:
    - failure
    report_template: Job {{.Spec.Job}} failed.
```

For example, by the above snippet, a Slack alert will be sent out to `#forum` channel when there is a failure of the job. The alert is formatted by `report_template`.

* The channel has to be in [coreos.slack.com](https://coreos.slack.com/).
* The channel has to be public. If it is not then the `@prow` bot has to be added to it otherwise it won't be able to properly post messages.
* The state in `job_states_to_report` has to be a valid Prow job state. See [upstream documentation](https://pkg.go.dev/sigs.k8s.io/prow/pkg/apis/prowjobs/v1#ProwJobState).
* The value of `report_template` is a [Go template](https://golang.org/pkg/text/template/) and it will be applied to the Prow job instance. The annotations such as `{{.Spec.Job}}` will be replaced by the job name when the alert is received in Slack. See [upstream documentation](https://pkg.go.dev/sigs.k8s.io/prow/pkg/apis/prowjobs/v1#ProwJob) for more fields of a Prow job. Note that no alerts will be sent out if the template is broken, e.g., cannot be parsed or applied successfully.

The following snippet shows an example of embedding the `reporter_config` into a job definition:

```yaml
agent: kubernetes
cron: 0 */6 * * *
decorate: true
name: periodic-job-name
reporter_config:
  slack:
    channel: '#forum'
    job_states_to_report:
    - failure
    report_template: Job {{.Spec.Job}} failed.
spec: {}  # Valid Kubernetes PodSpec.
```

_Note_ that we have to modify the job yaml directly even if the job is generated from a `ci-operator`'s config and regenerating the jobs does not change the existing `reporter_config`. Moreover, we do not support `reporter_config` for **generated** presubmits and postsubmits.
