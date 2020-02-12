---
layout: post
title: "SnapScheduler v1.1.0 released"
author: John Strunk
categories: snapscheduler release
---

Today marks the release of snapscheduler v1.1.0. This release brings support for
Kubernetes 1.17 and the `snapshot.storage.k8s.io/v1beta1` API for CSI snapshots.

In the switch from 1.16 to 1.17, the snapshot API moved from `v1alpha1` to
`v1beta1`. Since snapscheduler directly manipulates the VolumeSnapshot objects,
it required some code updates. However, snapscheduler retains compatibility with
the older alpha API, so it continues to function on pre-1.17 clusters.

With these changes, official support for 1.12 has been removed, largely due to a
lack of testing.

## Behind the scenes

Some of the other highlights of this release are less user-visible.

During the development of v1.1, CI was moved from Travis CI over to GitHub
Actions. This has improved the speed and reliability of the tests on PRs. Each
PR is now also being tested against Kubernetes minor versions from 1.13 through
1.17 with the use of [Kind](http://kind.sigs.k8s.io/) and the [CSI Hostpath
Driver](https://github.com/kubernetes-csi/csi-driver-host-path).

In order to support both the `v1alpha1` and `v1beta1` VolumeSnapshot objects, an
abstraction layer was introduced in the code to permit the same scheduling and
expiration logic to work independent of the snapshot API version. This will
hopefully also ease the transition to the eventual `v1` API.

Snapscheduler is built on the Operator Framework's
[operator-sdk](https://github.com/operator-framework/operator-sdk). As a part of
this release, the operator-sdk was updated to `v0.15.1`.

For a full list of changes, feel free to browse the repository
[CHANGELOG](https://github.com/backube/snapscheduler/blob/master/CHANGELOG.md).

## Installing and updating

The easiest way to get up and running is via the Helm chart, available from Helm
Hub: https://hub.helm.sh/charts/backube/snapscheduler

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

The full documentation is available at https://backube.github.io/snapscheduler/

Comments, questions, feedback? Come say 'hi' on gitter: [![Gitter
chat](https://badges.gitter.im/backube/snapscheduler.png)](https://gitter.im/backube/snapscheduler)

{% include twitter-follow.html %}
