---
title: Docker Desktop
description: Instructions to set up Docker Desktop for Istio.
weight: 15
skip_seealso: true
aliases:
    - /docs/setup/kubernetes/prepare/platform-setup/docker-for-desktop/
    - /docs/setup/kubernetes/prepare/platform-setup/docker/
    - /docs/setup/kubernetes/platform-setup/docker/
keywords: [platform-setup,kubernetes,docker-desktop]
owner: istio/wg-environments-maintainers
test: no
---

1. To run Istio with Docker Desktop, install a version which contains a [supported Kubernetes version](/es/docs/releases/supported-releases#support-status-of-istio-releases)
    ({{< supported_kubernetes_versions >}}).

1. If you want to run Istio under Docker Desktop's built-in Kubernetes, you need to increase Docker's memory limit
    under the *Resources->Advanced* pane of Docker Desktop's *Settings...*. Set the resources to at least 8.0 `GB` of memory and 4 `CPUs`.

    {{< image width="60%" link="./dockerprefs.png"  caption="Docker Preferences"  >}}

    {{< warning >}}
    Minimum memory requirements vary.  8 `GB` is sufficient to run
    Istio and Bookinfo.  If you don't have enough memory allocated in Docker Desktop,
    the following errors could occur:

    - image pull failures
    - healthcheck timeout failures
    - kubectl failures on the host
    - general network instability of the hypervisor

    Additional Docker Desktop resources may be freed up using:

    {{< text bash >}}
    $ docker system prune
    {{< /text >}}

    {{< /warning >}}
