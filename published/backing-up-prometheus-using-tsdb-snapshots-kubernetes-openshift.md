
# Backing up Prometheus using TSDB Snapshots : Kubernetes/OpenShift
![](https://miro.medium.com/max/700/1*7jh5cyHNuv4bP3bg8hQOeQ.png)

> These are my quick-and-dirty brain-dump notes to myself on how to backup prometheus database running on k8s or OpenShift

* Get Token for API Authentication and Prometheus API Route URL

    oc whoami -t
    oc get route -n openshift-monitoring | grep -i prometheus

* Run sample curl request

    
    curl -ks -H ‘Authorization: Bearer 0za4LjX9xPcqDjhWaufkgcQGo4grqA7ws4zvHrqgfY4’ ‘[https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v1/query?query=ALERTS'](https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v1/query?query=ALERTS') | python -m json.tool
    

* Create TSDB Snapshot

    curl -X ‘POST’ -ks -H ‘Authorization: Bearer 0za4LjX9xPcqDjhWaufkgcQGo4grqA7ws4zvHrqgfY4’ ‘[https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v2/admin/tsdb/snapshot'](https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v2/admin/tsdb/snapshot') | python -m json.tool

* You might get an error, so you first need to enable Admin APIs

    {
     “error”: “Admin APIs are disabled”,
     “message”: “Admin APIs are disabled”,
     “code”: 14
    }

* Enable AdminAPI

    oc -n openshift-monitoring patch prometheus k8s \
     — type merge — patch ‘{“spec”:{“enableAdminAPI”:true}}’

* Verify Admin API is enabled

    oc describe po prometheus-k8s-1 | grep -i admin
     — web.enable-admin-api

* Hit TSDB snapshot API to take snapshot

    curl -X ‘POST’ -ks -H ‘Authorization: Bearer 0za4LjX9xPcqDjhWaufkgcQGo4grqA7ws4zvHrqgfY4’ ‘[https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v2/admin/tsdb/snapshot'](https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v2/admin/tsdb/snapshot')
     | python -m json.tool

    {
     “name”: “20210512T162601Z-33415dbd315ae6af”
    }

* Find the snapshot and copy it locally

* The default folder is /prometheus/snapshots/ but you can find the data folder by finding the --storage.tsdb.path config in your deployment.

    curl -X ‘POST’ -ks -H ‘Authorization: Bearer 0za4LjX9xPcqDjhWaufkgcQGo4grqA7ws4zvHrqgfY4’ ‘[https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v1/admin/tsdb/snapshot'](https://prometheus-k8s-openshift-monitoring.apps.ocp4.cp4d.com/api/v1/admin/tsdb/snapshot') | python -m json.tool

* List the the snapshot directory

    oc -n openshift-monitoring exec -it prometheus-k8s-0 -c prometheus — /bin/sh -c “ls /prometheus/snapshots/20210512T162601Z-33415dbd315ae6af”

* Copy the Snapshot from prometheus container to local machine

    oc project openshift-monitoring
    oc rsync prometheus-k8s-0:/prometheus/snapshots/ /home/prometheus
    asdf

* And this is how to back Prometheus Snapshot to a local machine
