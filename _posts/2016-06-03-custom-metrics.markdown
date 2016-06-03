---
layout: post
title: Custom Metrics
---
In Kubernetes there is a way for a container to expose arbitrary application-specific metrics to
cAdvisor, which are in turn consumed by kubelet's stats/summary API.

To expose custom metrics, do the following:

- Turn on `--enable-custom-metrics` on each kubelet.
- Instrument your application using the Prometheus client library, so that metrics are exported via
  the `/metrics` HTTP endpoint.
- Map the container's port to a `hostPort` in the pod definition.
- In the pod definition, mount a volume at `/etc/custom-metrics` for the container, and the volume
  should contain a file named `definition.json`.
- `definition.json` should have the following content:

  ```json
  {
    "endpoint": "http://localhost:<hostport>/metrics",
    "metrics_config": [
      "<metric_name_foo>",
      "<metric_name_bar>",
      "..."
    ]
  }
```

You can view the custom metrics at the following endpoints:

- cAdvisor: `http://<kubelet_host>:4194/api/v2.0/stats/system.slice/docker-<container_id>.scope`
  - You can find `container_id` from `docker ps --no-trunc`
- Kubelet stats/summary API: `http://<kubelet_host>:10255/stats/summary`

Caveats:

- If a metric you export has labels, the cAdvisor endpoint will correctly show an array of all
  available labelsets, but the stats/summary API will just show the value for whatever happens to be
  the first. For this reason, you should only use custom metrics to show non-labeled metrics.


How it works:

- When you turn on `--enable-custom-metrics`, kubelet looks for the following when starting a
  container:
  - Does it have a volume mounted at `/etc/custom-metrics`?
  - Is there a file named `definition.json` in the volume?

  If both are true, it adds docker label 
  `io.cadvisor.metric.prometheus=/etc/custom-metrics/definition.json` to the container. You can
  verify by looking at the output of `docker inspect <container_id>`.
- This label signals to cAdvisor that this container has custom metrics in prometheus format, and
  that the detail for how to retrieve the metrics are defined in that file inside the container.
- cAdvisor parses the file, and periodically issues HTTP requests to the endpoint defined in the
  file. Since cAdvisor runs in the host network namespace, the HTTP queries will only work if the
  port is open on the host, hence the need for `hostPort`.
