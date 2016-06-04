---
layout: post
title: Kubernetes outage post-mortem
---
Issues:

- Docker daemon hanging ([bug](https://github.com/docker/docker/issues/5618),
  [bug](https://github.com/docker/docker/issues/9605))
- Kubelet not detecting docker problem, continuing to accept pods, which were stuck in pending
  forever ([bug](https://github.com/kubernetes/kubernetes/issues/17525))
- Systemd not reaping docker daemon after it was killed. The docker daemon process remained in
  "defunct" state. ([bug](https://bugzilla.redhat.com/show_bug.cgi?id=958505))
- Disk full on Kubernetes master
  - root partition too small (8G)
  - weird symptoms: can't `docker inspect` any images
- On dev master, kubelet died due to dependencies and didn't restart due to my misunderstanding of certain aspects of how systemd dependencies work.

Timeline for the dev master outage: (times are UTC)

- Jun 04 02:38:08 flanneld process died. flanneld.service entered failed state.
- Jun 04 02:38:13 systemd started to try to restart flanneld. It was 5 seconds after flanneld died because `RestartSec=5s` was configured for flanneld.service
  - As part of the restart process, systemd stopped docker.service and kubelet.service, because...
  - When systemd restarts unit X, it will also restart units that depend on X, that is, have `Requires=X`. Further, if unit Y has `After=X`, it will be stopped before X is stopped, and started after X is started. (X dying alone won't cause systemd to stop services dependent on X.)
  - flanneld.service failed to come up the first time. Since it was configured with `Restart=always`, systemd kept trying, and it was eventually brought up. However, it had the effect of leaving docker.service and kubelet.service permanently in the failed state. See below for a detailed explanation.

kubelet.service was configured with

```
Requires=docker.service
After=docker.service
```

Correct thing to do is make kubelet.service not require docker.service. Systemd would then restart kubelet.service when it dies, and it would bring up docker.service via socket activation if it's not already running.


### Why docker daemon was killed and not restarted

Consider two systemd services, A.service and B.service.

A.service

```
[Service]
Restart=always
```

B.service

```
[Unit]
Requires=A.service
After=A.service
```

Suppose initially A and B are both running. When systemd restarts A, either because someone types the command `systemctl restart A.service` or becuase A's process dies, the following sequence of events happens:

1. stopping B (systemd stops B first because B is configured to run <b>After</b> A)
2. B stopped
3. stopping A
4. A stopped
5. starting A (systemd starts A before B for the same reason)

From this point on, there are two possibilities:

- A started successfully. Systemd will start B. Or
- A fails to start. B will fail with reason 'dependency'.

In the second case, if after some retries A is eventually started successfully, <b>systemd will not attempt to start B</b>, regardless of whether B is set to automatically restart.

In our case, A is flanneld, and B is docker.


[another bug](https://github.com/kubernetes/kubernetes/issues/20096)
