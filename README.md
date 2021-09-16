# prometheus-stackdriver-gke

Follow instructions at [Monitoring apps running on multiple GKE clusters using Prometheus and Cloud Monitoring](https://cloud.google.com/architecture/monitoring-apps-running-on-multiple-gke-clusters-using-prometheus-and-stackdriver).

When it comes to applying the k8s config files from this repo, use these commands instead of those in the instructions:

```sh
envsubst < prometheus-service-account.yaml | kubectl apply -f -
```
```sh
envsubst < prometheus-configmap.yaml | kubectl apply -f -
```
```sh
envsubst < gke-prometheus-deployment.yaml | kubectl apply -f -
```
Additional iam policy bindings may be required (from [hixichen](https://github.com/hixichen)'s comment on [Issue 142](https://github.com/Stackdriver/stackdriver-prometheus-sidecar/issues/142) of the [stackdriver-prometheus-sidecar](https://github.com/Stackdriver/stackdriver-prometheus-sidecar) repository):

```sh
export GCP_PROJECT=my-project
export GCP_SA=prometheus
export K8S_SA=prometheus
export K8S_NS=prometheus

gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:${GCP_PROJECT}.svc.id.goog[${K8S_NS}/${K8S_SA}]" \
  ${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding ${GCP_PROJECT} \
  --member "serviceAccount:${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com" \
  --role roles/monitoring.metricWriter

gcloud projects add-iam-policy-binding ${GCP_PROJECT} \
  --member "serviceAccount:${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com" \
  --role roles/monitoring.viewer


gcloud projects add-iam-policy-binding ${GCP_PROJECT} \
  --member "serviceAccount:${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com" \
  --role roles/logging.logWriter


gcloud projects add-iam-policy-binding ${GCP_PROJECT} \
  --member "serviceAccount:${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com" \
  --role roles/stackdriver.resourceMetadata.writer


kubectl annotate serviceaccount ${K8S_SA} \
  iam.gke.io/gcp-service-account="${GCP_SA}@${GCP_PROJECT}.iam.gserviceaccount.com" \
  -n ${K8S_NS}
```
