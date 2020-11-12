# openshift-falcon-daemonset
Sample OpenShift DaemonSet to deploy CrowdStrike Falcon within Red Hat OpenShift or OpenShift Origin clusters.

# Repository Structure
| File / Folder | Description |
|:--------------|:------------|
| config-map.yaml | OpenShift Config Map, containers options passed to Falcon sensor (such as CID). |
| crwd-falcon-agent.yaml | OpenShift DaemonSet Deployment Specification for Falcon Agent. |

# Requirements
* Falcon Linux Sensor that supports the kernel on the worker nodes. For RHEL 8.3 systems, as of November 2020, this will require pre-release access to the Falcon sensor. Contact your CrowdStrike account team for access.
* Functioning OpenShift Cluster. This daemonset has been tested on:
  * OpenShift 4.4.29
  * OpenShift 4.5.17
  * OpenShift 4.6.1
* ``oc`` CLI installed.
* Sufficient permissions on your OpenShift cluster to import/create containers (e.g. Image Streams), create ConfigMaps, and create DaemonSets.
* Your CrowdStrike Customer ID (CrowdStrike ``CID``).
* Container image of the Falcon sensor. See https://github.com/CrowdStrike/dockerfiles for inspiration on creating your Docker images.


# Deployment Guide

## Required Parameters
* In ``config-map.yaml``, place your CrowdStrike Customer ID (CID) in the ``FALCONCTL_OPT_CID`` field.
* In ``crwd-falcon-agent.yaml``, place the URL to your container image in the ``containers`` -> ``name`` -> ``image`` stanza.

## Deploy the DaemonSet

* Login to your OpenShift Cluster
  * ``oc login --token=<<token>> --server=https://api.YourOpenShiftCluster.com:6443``

* Create Falcon Service Account
  * ``oc create serviceaccount falcon``
  * ``oc policy add-role-to-user privileged falcon``
  * ``oc adm policy add-scc-to-user privileged falcon``

* Configure the DaemonSet by adjusting variables in the ``config-map.yaml`` file:

| Variable | Mandatory / Optional | Description |
|:---------|:--------------------:|:------------|
| ``FALCONCTL_OPT_CID`` | **mandatory** | Your CrowdStrike Customer ID (``CID``). This can be found at https://falcon.crowdstrike.com/hosts/sensor-downloads. |
| ``FALCONCTL_OPT_AID`` | *optional* | |
| ``FALCONCTL_OPT_APD`` | *optional* | |
| ``FALCONCTL_OPT_APH`` | *optional* | |
| ``FALCONCTL_OPT_APP`` | *optional* | |
| ``FALCONCTL_OPT_TRACE`` | *optional* | |
| ``FALCONCTL_OPT_FEATURE`` | *optional* | |
| ``FALCONCTL_OPT_MESSAGE_LOG`` | *optional* | |
| ``FALCONCTL_OPT_BILLING`` | *optional* | |
| ``FALCONCTL_OPT_ASSERT`` | *optional* | |
| ``FALCONCTL_OPT_MEMFAIL_GRACE_PERIOD`` | *optional* | |
| ``FALCONCTL_OPT_MEMFAIL_EVERY_N`` | *optional* | |

* Configure the URI of your Falcon container in ``crwd-falcon-agent.yaml``:
`````
      containers:
      - name: sensor-container
        image: <<your container image path>>
`````
* Deploy the Falcon DaemonSet
  * ``$ oc create -f crwd-falcon-agent.yaml``

# Troubleshooting
Retrieve the names of the pods relating to your Falcon sensor deployment:

```shell
$ oc get pods

NAME                  READY   STATUS             RESTARTS   AGE
falcon-sensor-82lzw   0/2     CrashLoopBackOff   5          6m8s
falcon-sensor-f4jrr   0/2     Error              6          6m8s
falcon-sensor-rln6k   0/2     CrashLoopBackOff   5          6m8s
```

Look at the previous pod logs of an instance of the sensor container:
```shell
$ oc logs falcon-sensor-f4jrr -c sensor-container -p
```

If no logs appear, or if you receive an error about ``previous terminated container "sensor-container" in pod "falcon-sensor-f4jrr" not found``, debug the pod. The following command should create a copy of the pod and provide a shell:
```shell
$ oc debug pod/falcon-sensor-f4jrr
```

Common issues:
1. DaemonSet deploys, pods are running, but no detections are generated!
``oc debug`` into the pod (or access terminal via OpenShift Console) to verify your sensor is not in Reduced
Functionality Mode (RFM). This is often caused by the variance between CoreOS and RHEL kernels, where CoreOS often
releases different kernel versions than those present in RHEL. File a bug with CrowdStrike requesting a new
kernel version to be supported.

# Many Thanks
Thank you to [Dinesh Subhraveti](https://www.linkedin.com/in/subhraveti/) whose initial code inspired this repo!
