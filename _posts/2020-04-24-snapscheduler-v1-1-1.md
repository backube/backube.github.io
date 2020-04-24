---
layout: post
title: "SnapScheduler v1.1.1 released"
author: John Strunk
categories: release snapscheduler
---

We are announcing a new release of SnapScheduler. Version 1.1.1 is a bug fix
release that fixes a crash when the snapschedule CR omits the
`snapshotTemplate`.

There are no new features to announce with this release of the operator, but a
new version of the Helm chart will also be rolling out that includes a default
node selector on the operator's Deployment. This node selector ensures that the
operator is scheduled only to nodes that are amd64 & running Linux. This change
should not cause any noticeable difference for those that have the scheduler
deployed, but it will ensure deployments are successful on clusters with
heterogenous architectures or OSes.

The full list of changes is available in the usual place:
[CHANGELOG](https://github.com/backube/snapscheduler/blob/master/CHANGELOG.md).

**:trumpet: While we're announcing things... :trumpet:**  
SnapScheduler is now [listed on
ArtifactHub.io](https://artifacthub.io/package/chart/backube-helm-charts/snapscheduler),
too!

Other (boring) things behind the scenes include:

- End-to-end testing of the major features in CI with Kube 1.13 -- 1.18
- A few doc fixes to clarify usage

## Installing and updating

The easiest way to get up and running is via the Helm chart, available from Helm
Hub:
[https://hub.helm.sh/charts/backube/snapscheduler](https://hub.helm.sh/charts/backube/snapscheduler)

New installation:

```console
$ helm repo add backube https://backube.github.io/helm-charts/
$ kubectl create ns backube-snapscheduler
$ helm install -n backube-snapscheduler snapscheduler backube/snapscheduler
```

Upgrade:

```console
$ helm repo update
$ helm upgrade -n backube-snapscheduler snapscheduler backube/snapscheduler
```

The full documentation is available at
[https://backube.github.io/snapscheduler/](https://backube.github.io/snapscheduler/)

Comments, questions, feedback? Come say 'hi' on gitter: [![Gitter
chat](https://badges.gitter.im/backube/snapscheduler.png)](https://gitter.im/backube/snapscheduler)

{% include twitter-follow.html %}
