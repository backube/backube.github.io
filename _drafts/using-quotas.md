---
layout: post
title: "Using quotas to prevent resource exhaustion"
author: John Strunk
categories: snapscheduler
---

So... You're a good admin and you've given your developers access to the
[snapscheduler](https://backube.github.io/snapscheduler/) so they can protect
their data from accidents. They like it, you like it, everyone is happy!  
Then one day...

At 2am your phone comes alive, waking you from a deep slumber...

The storage backing your Kubernetes cluster is near full!

After some quick checks, you discover thousands and thousands of snapshots are
eating up all the free space. Somebody likes the scheduler a bit too much.

You clear out the majority of them to get the system back to normal; then you go
sleuthing...

One conscientious developer decided to protect his data with hourly snapshots:

```yaml
---
apiVersion: snapscheduler.backube/v1
kind: SnapshotSchedule
metadata:
  name: hourly
spec:
  schedule: "@hourly"
  snapshotTemplate:
    snapshotClassName: ebs-csi
```

Did you notice the problem with the above schedule?

Yep... no retention specification. The above schedule will never prune the
snapshots. They will accumulate until manually deleted... _(likely by you, at
2am)_

In addition to encouraging your developers to include a proper retention spec,
let's look at how to implement a fail-safe.

BTW, here's a better schedule:

```yaml
---
apiVersion: snapscheduler.backube/v1
kind: SnapshotSchedule
metadata:
  name: hourly
spec:
  retention:
    # This schedule keeps a max of 7 snaps per PVC
    maxCount: 7
  schedule: "@hourly"
  snapshotTemplate:
    snapshotClassName: ebs-csi
```

## Using resource quotas

We want to let developers define their snapshot policies, but we also want to
avoid middle of the night surprises. That means we need some guardrails.

Starting with Kubernetes 1.15, [ResourceQuota objects can be used to limit the
object count of custom
resources](https://kubernetes.io/docs/concepts/policy/resource-quotas/#object-count-quota).
We can use this to our advantage by limiting the number of VolumeSnapshot
objects that can be created per Namespace.

Here's an example of limiting the number of snapshots to 50:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: snapshots
spec:
  hard:
    count/volumesnapshots.snapshot.storage.k8s.io: "50"
```

Since the ResourceQuota object is Namespace-scoped, the total number of
snapshots are limited at the Namespace level. In the above example, a developer
would be free to define whatever set of schedules (frequency, retention policy,
and selectors) they choose. The only restriction would be that there can be no
more than 50 total snapshots in the Namespace at once.

A small caveat is required here: When choosing the total number of snapshots, be
sure to leave a bit of headroom because the scheduler creates new snapshots
before removing old ones. This means that a schedule with
`spec.retention.maxCount: 7` will actually have a maximum of 8 total snapshots.

### Checking quota consumption

Once you've created a ResourceQuota to constrain VolumeSnapshots, you can look
at how the current consumption compares to the imposed limit:

```console
$ kubectl -n myns describe resourcequota/snapshots
Name:                                          snapshots
Namespace:                                     myns
Resource                                       Used  Hard
--------                                       ----  ----
count/volumesnapshots.snapshot.storage.k8s.io  3     50


$ kubectl -n myns get volumesnapshots
NAME                      AGE
pvc-minute-202002111646   2m23s
pvc-minute-202002111647   83s
pvc-minute-202002111648   23s
```

## Summary

While allowing your application developers to protect their own data with
scheduled snapshots is a good practice, it is important to have guardrails in
place to prevent resource exhaustion.

So, go on... create some ResourceQuotas in your cluster. While I can't guarantee
you'll sleep better, it will be one less thing to worry about.

{% include twitter-follow.html %}
