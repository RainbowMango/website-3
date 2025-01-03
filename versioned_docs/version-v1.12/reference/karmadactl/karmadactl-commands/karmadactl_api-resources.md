---
title: karmadactl api-resources
---

Print the supported API resources on the server

### Synopsis

Print the supported API resources on the server.

```
karmadactl api-resources [flags]
```

### Examples

```
  # Print the supported API resources in Karmada control plane
  karmadactl api-resources
  
  # Print the supported API resources with more information in cluster(member1)
  karmadactl api-resources -o wide --operation-scope=members --cluster=member1
  
  # Print the supported API resources sorted by a column in Karmada control plane
  karmadactl api-resources --sort-by=name
  
  # Print the supported namespaced resources in Karmada control plane
  karmadactl api-resources --namespaced=true
  
  # Print the supported non-namespaced resources in Karmada control plane
  karmadactl api-resources --namespaced=false
  
  # Print the supported API resources with a specific APIGroup in Karmada control plane
  karmadactl api-resources --api-group=rbac.authorization.k8s.io
```

### Options

```
      --api-group string                 Limit to resources in the specified API group.
      --cached                           Use the cached list of resources if available.
      --categories strings               Limit to resources that belong to the specified categories.
      --cluster string                   Used to specify a target member cluster and only takes effect when the command's operation scope is members, for example: --operation-scope=members --cluster=member1
  -h, --help                             help for api-resources
      --karmada-context string           The name of the kubeconfig context to use
      --kubeconfig string                Path to the kubeconfig file to use for CLI requests.
      --namespaced                       If false, non-namespaced resources will be returned, otherwise returning namespaced resources by default. (default true)
      --no-headers                       When using the default or custom-column output format, don't print headers (default print headers).
  -s, --operation-scope operationScope   Used to control the operation scope of the command. The optional values are karmada and members. Defaults to karmada. (default karmada)
  -o, --output string                    Output format. One of: (wide, name).
      --sort-by string                   If non-empty, sort list of resources using specified field. The field can be either 'name' or 'kind'.
      --verbs strings                    Limit to resources that support the specified verbs.
```

### Options inherited from parent commands

```
      --add-dir-header                   If true, adds the file directory to the header of the log messages
      --alsologtostderr                  log to standard error as well as files (no effect when -logtostderr=true)
      --log-backtrace-at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log-dir string                   If non-empty, write log files in this directory (no effect when -logtostderr=true)
      --log-file string                  If non-empty, use this log file (no effect when -logtostderr=true)
      --log-file-max-size uint           Defines the maximum size a log file can grow to (no effect when -logtostderr=true). Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --logtostderr                      log to standard error instead of files (default true)
      --one-output                       If true, only write logs to their native severity level (vs also writing to each lower severity level; no effect when -logtostderr=true)
      --skip-headers                     If true, avoid header prefixes in the log messages
      --skip-log-headers                 If true, avoid headers when opening log files (no effect when -logtostderr=true)
      --stderrthreshold severity         logs at or above this threshold go to stderr when writing to files and stderr (no effect when -logtostderr=true or -alsologtostderr=true) (default 2)
  -v, --v Level                          number for the log level verbosity
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [karmadactl](karmadactl.md)	 - karmadactl controls a Kubernetes Cluster Federation.

#### Go Back to [Karmadactl Commands](karmadactl_index.md) Homepage.


###### Auto generated by [spf13/cobra script in Karmada](https://github.com/karmada-io/karmada/tree/master/hack/tools/genkarmadactldocs).