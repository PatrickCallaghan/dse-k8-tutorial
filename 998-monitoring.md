# Monitoring 

Opscenter isn't advised on k8s. It's best to use another monitoring tool, like prometheus or an ELK stack.

Datastax collector metrics can be used and/or a jmx agent can be deployed as sidecar.

To deploy an agent as sidecar, we can simply deploy multiple container in a single pod. network is shared in a single pod so there is no issue to access to a local JMX port.

With the collectd reporter, we can potentially send the collectd metrics to a volume shared between the DSE container and the agent container.

Example of a pod with a side car and a shared volume.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-sidecar
spec:
  # Create a volume called 'shared-logs' that the
  # app and sidecar share.
  volumes:
  - name: shared-logs 
    emptyDir: {}

  # In the sidecar pattern, there is a main application
  # container and a sidecar container.
  containers:

  # Main application container
  - name: app-container
    # Simple application: write the current date
    # to the log file every five seconds
    image: alpine # alpine is a simple Linux OS image
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /var/log/app.txt; sleep 5;done"]

    # Mount the pod's shared log file into the app 
    # container. The app writes logs here.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  # Sidecar container
  - name: sidecar-container
    # Simple sidecar: display log files using nginx.
    # In reality, this sidecar would be a custom image
    # that uploads logs to a third-party or storage service.
    image: nginx:1.7.9
    ports:
      - containerPort: 80

    # Mount the pod's shared log file into the sidecar
    # container. In this case, nginx will serve the files
    # in this directory.
    volumeMounts:
    - name: shared-logs
      mountPath: /usr/share/nginx/html # nginx-specific mount path
```
