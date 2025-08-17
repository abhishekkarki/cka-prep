# Config Maps

### What is a ConfigMap?

A ConfigMap is a Kubernetes object used to store non-sensitive configuration data in key-value pairs.
It lets you decouple application configuration from the container image.

Without ConfigMaps:
- You’d hardcode configs into your app or bake them into your container image → bad for flexibility.

With ConfigMaps:
- You can inject config dynamically into pods without rebuilding images.

For sensitive data (passwords, API keys), you should use Secrets, not ConfigMaps.


## Ways to Create a ConfigMap

### From literal key-values
```bash
kubectl create configmap app-config \
  --from-literal=APP_MODE=production \
  --from-literal=APP_PORT=8080
```

### From a file

`kubectl create configmap app-config --from-file=config.properties`

(where config.properties contains APP_MODE=production etc.)

### From a directory

kubectl create configmap app-config --from-file=./configs/

(all files in the dir become keys in the ConfigMap)


## ConfigMap Manifest Example

Here’s a YAML definition:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_MODE: "production"
  APP_PORT: "8080"
  LOG_LEVEL: "debug"
```

## Using ConfigMaps in Pods

There are 3 main ways to use ConfigMaps in pods:

### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: app-container
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

All keys in app-config become environment variables inside the container:

```bash
$ echo $APP_MODE    # production
$ echo $APP_PORT    # 8080
```

You can also reference individual keys:
```yaml
env:
  - name: MODE
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_MODE

```


### As Volumes (mounted files)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: app-container
      image: busybox
      command: ["sh", "-c", "cat /etc/config/APP_MODE; sleep 3600"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```
Inside container:

```bash
/etc/config/APP_MODE   → contains "production"
/etc/config/APP_PORT   → contains "8080"
```

You can also mount only specific keys:

```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
        - key: LOG_LEVEL
          path: loglevel.txt
```


### As Command-line arguments

Since env vars can be expanded in commands:

`command: ["nginx", "-g", "daemon off;", "-c", "$(APP_MODE)"]`

## ConfigMap Updates
- If you update a ConfigMap, the changes are picked up differently:
- Env vars: pods must be restarted to see new values.
- Mounted volumes: updates are eventually reflected in files (typically within seconds, depending on CNI/volume implementation).

This makes volume mounts better for dynamic configs.


## Common Use Cases
- Application settings (mode, port, log level).
- External service endpoints (URLs, hostnames).
- Feature flags.
- Non-sensitive credentials (though Secrets are safer).



## Common Pitfalls
-	Don’t put sensitive data in ConfigMaps → use Secrets.
-	Large ConfigMaps (MBs) are inefficient; keep them small.
-	Don’t forget pod restart requirement for env var changes.
-	ConfigMaps are namespace-scoped — a pod in ns-A can’t use a ConfigMap in ns-B (unless you copy it).
-


## Summary
ConfigMaps let you inject configuration into pods without rebuilding images.
You can consume them as env vars, volumes, or command args.
They’re great for keeping apps flexible and portable.
