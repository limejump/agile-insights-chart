# Agile insights helm chart

This chart includes the templates for deploying the agile insights system, which is composed of:
- `dashboard` (a webapp)
- `mongodb` (a document store)
- `dataCollection` (two cronjobs)

## Main Components

### dashboard

A Python Dash application which presents agile metrics stored in **mongodb**. This is deployed as a `Deployment`

### mongodb

The backing database for the **dashboard**. Configuration options are described in detail [here](https://github.com/bitnami/charts/tree/master/bitnami/mongodb).

### dataCollection

Two ETLs which are deployed as `Cronjobs`. They fetch and transform data from an issue tracking tool (currently only Jira is supported) and store the transformed data in **mongodb**.

## Other Resources

### Jira Config

The `config` subsection is deployed as a `ConfigMap` which is mounted as a json file into both the **dataCollection** `Cronjobs` and the **dashboard** `Pods`. It should follow the same format as provided in [this example](https://github.com/limejump/agile-insights/blob/main/config_files/example-config.json).

### Secrets

This chart requires that `existingSecrets` are created separately to this chart for:
- `jira.credentials`
- `mongodb.auth`

In the case of **mongodb** this helps prevent you from a situation that arises when you allow mongodb to auto-generate it's secret. In this scenario, if you ever need to re-release the mongodb chart, whilst keeping the existing `PersistentVolumeClaim` (underlying data), The mongodb `Deployment` will use a newly generated secrets, but the data volume will continue using the previously generated secrets. You essentially get locked out of your own data if you're not careful.


| Value name                      | Description                                                  | Default                                                      |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| dashboard.replicaCount          | Number of dashboard pods                                     | `1`                                                          |
| dashboard.image.repository      | Dashboard image repository                                   | `sprints-dashboard`                                          |
| dashboard.image.pullPolicy      | Dashboard image pullPolicy                                   | `IfNotPresent`                                               |
| dashboard.image.tag             | Dashboard image tag                                          | `local`                                                      |
| dashboard.imagePullSecrets      | Dashboard image pull secrets                                 | `[]`                                                         |
| dashboard.podSecurityContext    | Dashboard pod security context                               | `{}`                                                         |
| dashboard.SecurityContext       | Dashboard container security context                         | `{}`                                                         |
| dashboard.service.type          | Dashboard service type                                       | `ClusterIP`                                                  |
| dashboard.service.annotations   | Dashboard service annotations                                | `service.beta.kubernetes.io/aws-load-balancer-type: externa`l<br />`service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip`<br />`service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing` |
| dashboard.service.port          | Port that the dashboard service is reachable on              | `80`                                                         |
| dashboard.service.targetPort    | Port that the dashboard container exposes                    | `8000`                                                       |
| dashboard.ingress.enabled       | Whether to enable an ingress resource or not                 | `false`                                                      |
| dashboard.ingress.hosts         | if enabled, the default hosts                                | path `/` on `localhost`                                      |
| dashboard.ingress.tls           | if enabled, the default tls config                           | `[]`                                                         |
| dashboard.resources             | Dashboard resource requests and limits                       | `{}`                                                         |
| dashboard.nodeSelector          | Dashboard node selector                                      | `{}`                                                         |
| dashboard.tolerations           | Dashboard taint tolerations                                  | `[]`                                                         |
| dashboard.affinity              | Dashboard pod affinity                                       | `{}`                                                         |
| jira.config                     | Jira config file as described [here](#jira-config)           | `{}`                                                         |
| jira.credentials.existingSecret | An existing jira secret containing credentials ***required** | `{}`                                                         |
| dataCollection.enabled          | Whether to deploy the data collection `Cronjobs` or not      | `false`                                                      |
| dataCollection.image.repository | Dashboard image repository                                   | `sprints-dashboard`                                          |
| dataCollection.image.pullPolicy | Dashboard image pullPolicy                                   | `IfNotPresent`                                               |
| dataCollection.image.tag        | Dashboard image tag                                          | `local`                                                      |

## MongoDB Values

| Value name                      | Description                                                            | Default                    |
| ------------------------------- | ---------------------------------------------------------------------- | -------------------------- |
| mongodb.architecture            | mongodb architecture                                                   | `standalone`               |
| mongodb.useStatefulSet          | Whether or not to persist the mongodb data                             | `true`                     |
| mongodb.initContainers          | mongodb initialisation containers                                      | `[]`                       |
| mongodb.auth.existingSecret     | mongodb exitsing secret [see](#secrets)                                | `[]`                       |
