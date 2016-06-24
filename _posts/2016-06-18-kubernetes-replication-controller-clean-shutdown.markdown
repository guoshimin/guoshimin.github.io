---
layout: post
title: Kubernetes pod clean shutdown
---
We run [Promtheus](http://prometheus.io) on Kubernetes and we had a problem where every time we
restart Prometheus, it would decide that the disk files were dirty and initiate the crash recovery
process, which would take a long time.

Some further investigation showed that this always happened when we restarted Prometheus by deleting
the Prometheus pod and letting the replication controller start another one. If we manually killed the
Prometheus process by sending it a SIGTERM, it would shut down cleanly and start up fast.

After cross examining various logs, finally some seemingly unrelated errors in the log revealed what
really happened: [https://github.com/kubernetes/kubernetes/blob/v1.2.4/pkg/controller/node/nodecontroller.go#L365](https://github.com/kubernetes/kubernetes/blob/v1.2.4/pkg/controller/node/nodecontroller.go#L365)

This error (although logged at INFO level) is printed when Kuberentes can't parse the kubelet
version reported by a minion. Kubernetes expected the version to be
[semver](http://semver.org/)-compliant. However, the version reported by our kubelets,
`v1.2.4.1+e191e9760c2dbd` was not. Failing to parse the kubelet version means that Kubernetes was
not able to determine the version of kubelet, and it proceeded to assume that the minion didn't
support graceful shutdown of pods, so it issued a forceful shutdown command.

So what caused kubelet to report this version number? The version number is embedded into the
kubelet binary during the build. The source is the output of `git describe`. In our case, we added a
patch to the official 1.2.4 release, so `git describe` shows `v1.2.4-1-ge191e9760c2dbd`. That output is then
passed through a `sed` command:
[https://github.com/kubernetes/kubernetes/blob/527cb50583a2599823e1784b0c35deb1e8ad763a/hack/lib/version.sh#L59](https://github.com/kubernetes/kubernetes/blob/527cb50583a2599823e1784b0c35deb1e8ad763a/hack/lib/version.sh#L59),
which is supposed to make the version a semver, but failed at this job.

I reported this issue in
[https://github.com/kubernetes/kubernetes/issues/27699](https://github.com/kubernetes/kubernetes/issues/27699),
but it got closed because "it was never intended to work". I guess he meant "never intended to work
for custom builds". Still, I think that was a poor response. It's an open-source project. People are
going to fork it and play with it. The consequence of it not working leads to problems down the
line, that have manifestations that don't make people think, "Oh, it must be because I didn't set the version correctly."

If I sounded critical, it's because I love the Kubernetes project. In almost every way it is done
with very high standards. Now let me try to come up with a patch to fail fast during build when the
version is wrong.
