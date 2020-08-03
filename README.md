# openshift-falcon-daemonset
Sample OpenShift DaemonSet to deploy CrowdStrike Falcon within Red Hat OpenShift or OpenShift Origin clusters.

# Repository Structure
| File / Folder | Description |
|:--------------|:------------|
| config-map.yaml | |
| crwd-falcon-agent.yaml | OpenShift DaemonSet Deployment Specification for Falcon Agent |

## Requirements
* Running OpenShift 4.x Cluster
* ``oc`` CLI installed
* Your CrowdStrike Customer ID (CrowdStrike ``CID``)
* Container image of the Falcon sensor. See https://github.com/CrowdStrike/dockerfiles for inspiration on creating your Docker images.


## Deployment Guide

### Required Parameters
* In ``config-map.yaml``, place your CrowdStrike Customer ID (CID) in the ``FALCONCTL_OPT_CID`` field.
* In ``crwd-falcon-agent.yaml``, place the URL to your container image in the ``containers`` -> ``name`` -> ``image`` stanza.

### Deploy the DaemonSet

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

# Many Thanks
Thank you to [Dinesh Subhraveti](https://www.linkedin.com/in/subhraveti/) whose initial code inspired this repo!
