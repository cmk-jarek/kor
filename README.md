![GitHub go.mod Go version (subdirectory of monorepo)](https://img.shields.io/github/go-mod/go-version/yonahd/kor)
![GitHub release (with filter)](https://img.shields.io/github/v/release/yonahd/kor?color=green&link=https://github.com/yonahd/kor/releases)
![Docker Pulls](https://img.shields.io/docker/pulls/yonahdissen/kor)
[![codecov](https://codecov.io/gh/yonahd/kor/branch/main/graph/badge.svg?token=tNKcOjlxLo)](https://codecov.io/gh/yonahd/kor)
[![Discord](https://discord.com/api/guilds/1159544275722321990/embed.png)](https://discord.gg/ajptYPwcJY)
 


# Kor - Kubernetes Orphaned Resources Finder

![Kor Logo](/images/kor_logo.png)

Kor is a tool to discover unused Kubernetes resources. Currently, Kor can identify and list unused:
- ConfigMaps  
- Secrets
- Services
- ServiceAccounts
- Deployments
- StatefulSets
- Roles
- HPAs
- PVCs
- Ingresses
- PDBs
- CRDs
- PVs
- Pods
- Jobs

![Kor Screenshot](/images/screenshot.png)

## Installation

Download the binary for your operating system from the [releases page](https://github.com/yonahd/kor/releases) and add it to your system's PATH.

### Homebrew
For macOS users, you can install Kor using Homebrew:
```sh
brew install kor
```
### Build from source
Install the binary to your `$GOBIN` or `$GOPATH/bin`:
```sh
go install github.com/yonahd/kor@latest
```

### Docker
Run a container with your kubeconfig mounted:
```sh
docker run --rm -i yonahdissen/kor

docker run --rm -i -v "/path/to/.kube/config:/root/.kube/config" yonahdissen/kor all
```

### Helm
Run as a cronjob in your Cluster (with an option for sending slack updates)
```sh
helm upgrade -i kor \
    --namespace kor \
    --create-namespace \
    --set cronJob.enabled=true
    ./charts/kor
```

Run as a deployment in your Cluster exposing prometheus metrics
```sh
helm upgrade -i kor \
    --namespace kor \
    --create-namespace \
    ./charts/kor
```


For more information see [in cluster usage](#in-cluster-usage) 

## Usage

Kor provides various subcommands to identify and list unused resources. The available commands are:

- `all` - Gets all unused resources for the specified namespace or all namespaces.
- `configmap` - Gets unused ConfigMaps for the specified namespace or all namespaces.
- `secret` - Gets unused Secrets for the specified namespace or all namespaces.
- `services` - Gets unused Services for the specified namespace or all namespaces.
- `serviceaccount` - Gets unused ServiceAccounts for the specified namespace or all namespaces.
- `deployments` - Gets unused Deployments for the specified namespace or all namespaces.
- `statefulsets` - Gets unused StatefulSets for the specified namespace or all namespaces.
- `role` - Gets unused Roles for the specified namespace or all namespaces.
- `hpa` - Gets unused HPAs for the specified namespace or all namespaces.
- `pods` - Gets unused Pods for the specified namespace or all namespaces.
- `pvc` - Gets unused PVCs for the specified namespace or all namespaces.
- `pv` - Gets unused PVs in the cluster(non namespaced resource).
- `ingress` - Gets unused Ingresses for the specified namespace or all namespaces.
- `pdb` - Gets unused PDBs for the specified namespace or all namespaces.
- `crd` - Gets unused CRDs in the cluster(non namespaced resource).
- `jobs` - Gets unused jobs for the specified namespace or all namespaces.
- `exporter` - Export Prometheus metrics.

### Supported Flags
```
      --delete                      Delete unused resources
  -l, --exclude-labels string       Selector to filter out, Example: --exclude-labels key1=value1,key2=value2.
  -e, --exclude-namespaces string   Namespaces to be excluded, splited by comma. Example: --exclude-namespace ns1,ns2,ns3. If --include-namespace is set, --exclude-namespaces will be ignored.
  -i, --include-non-namespaced      Include recources not bound to a namespace.
  -h, --help                        help for kor
  -n, --include-namespaces string   Namespaces to run on, splited by comma. Example: --include-namespace ns1,ns2,ns3. 
  -k, --kubeconfig string           Path to kubeconfig file (optional)
      --newer-than string           The maximum age of the resources to be considered unused. This flag cannot be used together with older-than flag. Example: --newer-than=1h2m
      --no-interactive              Do not prompt for confirmation when deleting resources. Be careful using this flag!
      --older-than string           The minimum age of the resources to be considered unused. This flag cannot be used together with newer-than flag. Example: --older-than=1h2m
  -o, --output string               Output format (table, json or yaml) (default "table")
      --slack-auth-token string     Slack auth token to send notifications to. --slack-auth-token requires --slack-channel to be set.
      --slack-channel string        Slack channel to send notifications to. --slack-channel requires --slack-auth-token to be set.
      --slack-webhook-url string    Slack webhook URL to send notifications to
  -v, --verbose                     Verbose output (print empty namespaces)

```

To use a specific subcommand, run `kor [subcommand] [flags]`.

```sh
kor all --include-namespaces my-namespace
```

For more information about each subcommand and its available flags, you can use the `--help` flag.

```sh
kor [subcommand] --help
```

## Supported resources and limitations

| Resource        | What it looks for                                                                                                                                                                                                                 | Known False Positives  ⚠️                                                                                                    |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| ConfigMaps      | ConfigMaps not used in the following places:<br/>- Pods<br/>- Containers<br/>- ConfigMaps used through Volumes<br/>- ConfigMaps used through environment variables                                                                | ConfigMaps used by resources which don't explicitly state them in the config.<br/> e.g Grafana dashboards loaded dynamically OPA policies fluentd configs |
| Secrets         | Secrets not used in the following places:<br/>- Pods<br/>- Containers<br/>- Secrets used through volumes<br/>- Secrets used through environment variables<br/>- Secrets used by Ingress TLS<br/>- Secrets used by ServiceAccounts |    Secrets used by resources which don't explicitly state them in the config                                                                                                                        |
| Services        | Services with no endpoints                                                                                                                                                                                                        |                                                                                                                              |
| Deployments     | Deployments with no Replicas                                                                                                                                                                                                      |                                                                                                                              |
| ServiceAccounts | ServiceAccounts unused by Pods<br/>ServiceAccounts unused by roleBinding or clusterRoleBinding                                                                                                                                    |                                                                                                                              |
| StatefulSets    | Statefulsets with no Replicas                                                                                                                                                                                                     |                                                                                                                              |
| Roles           | Roles not used in roleBinding                                                                                                                                                                                                     |                                                                                                                              |
| PVCs            | PVCs not used in Pods                                                                                                                                                                                                             |                                                                                                                              |
| Ingresses       | Ingresses not pointing at any Service                                                                                                                                                                                             |                                                                                                                              |
| Hpas            | HPAs not used in Deployments<br/> HPAs not used in StatefulSets                                                                                                                                                                   |                                                                                                                              |
| CRDs            | CRDs not used the cluster                                                                                                                                                                                                         |                                                                                                                              |
| Pvs             | PVs not bound to a PVC                                                                                                                                                                                                            |                                                                                                                              |
| Pdbs            | PDBs not used in Deployments<br/> PDBs not used in StatefulSets                                                                                                                                                                   |                                                                                                                              |
| Jobs            | Jobs status is completed                                                                                                                                                                                                          |                                                                                                                              |

## Deleting Unused resources
If you want to delete resources in an interactive way using Kor you can run:
```sh
kor configmap --include-namespaces my-namespace --delete
```
You will be prompted with:
```sh
Do you want to delete ConfigMap test-configmap in namespace my-namespace? (Y/N):
```

To delete with no prompt ( ⚠️ use with caution):
```sh
kor configmap --include-namespaces my-namespace --delete --no-interactive
```

## Ignore Resources
The resources labeled with: 
```sh
kor/used=true
```
will be ignored by kor even if they are unused. You can add this label to resources you want to ignore.

## In Cluster Usage

To use this tool inside the cluster running as a CronJob and sending the results to a Slack Webhook as raw text(has characters limits of 4000) or to a Slack channel by uploading a file(recommended), you can use the following commands:

```sh
# Send to a Slack webhook as raw text
helm upgrade -i kor \
    --namespace kor \
    --create-namespace \
    --set cronJob.slackWebhookUrl=<slack-webhook-url> \
    ./charts/kor
```

```sh
# Send to a Slack channel by uploading a file
helm upgrade -i kor \
    --namespace kor \
    --create-namespace \
    --set cronJob.slackChannel=<slack-channel> \
    --set cronJob.slackToken=<slack-token> \
    ./charts/kor
```
> Note: To send it to Slack as a file it's required to set the `slackToken` and `slackChannel` values.

It's set to run every Monday at 1 a.m. by default. You can change the schedule by setting the `cronJob.schedule` value.

```sh
helm upgrade -i kor \
    --namespace kor \
    --create-namespace \
    --set cronJob.slackChannel=<slack-channel> \
    --set cronJob.slackToken=<slack-token> \
    --set cronJob.schedule="0 1 * * 1" \
    ./charts/kor
```

## Grafana Dashboard
Dashboard can be found [here](https://grafana.com/grafana/dashboards/19863-kor-dashboard/).
![Grafana Dashboard](/grafana/dashboard-screenshot-1.png)

## Contributing

Contributions are welcome! If you encounter any bugs or have suggestions for improvements, please open an issue in the [issue tracker](https://github.com/yonahd/kor/issues).

## License

This open-source project is available under the [MIT License](LICENSE). Feel free to use, modify, and distribute it as per the terms of the license.

