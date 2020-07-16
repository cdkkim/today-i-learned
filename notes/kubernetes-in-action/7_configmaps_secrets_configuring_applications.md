# ConfigMaps and Secrets: configuring applications

Kubernetes allows separating configuration options into a separate object called a ConfigMap, which is a map containing key/value pairs with the values ranging from short literals to full config files.

Regardless if you’re using a ConfigMap to store configuration data or not, you can configure your apps by

- Passing command-line arguments to containers
- Setting custom environment variables for each container
- Mounting configuration files into containers through a special type of volume

#### Understanding `ENTRYPOINT` and `CMD`
In a Dockerfile, two instructions define the two parts:

- `ENTRYPOINT` defines the executable invoked when the container is started.
- `CMD` specifies the arguments that get passed to the `ENTRYPOINT`.

#### Understanding the difference between the shell and exec forms
Both instructions support two different forms:

- `shell` form—For example, ENTRYPOINT node app.js.
- `exec` form—For example, ENTRYPOINT ["node", "app.js"].

The difference is whether the specified command is invoked inside a shell or not.

#### Overriding the command and arguments in Kubernetes

    kind: Pod
    spec:
      containers:
      - image: some/image
        command: ["/bin/command"]
        args: ["arg1", "arg2", "arg3"]

> The command and args fields can’t be updated after the pod is created.

### Referring to other environment variables in a variable’s value

    env:
    - name: FIRST_VAR
      value: "foo"
    - name: SECOND_VAR
      value: "$(FIRST_VAR)bar"

#### Using ConfigMap entries as arguments

    apiVersion: v1
    kind: Pod
    metadata:
      name: fortune-args-from-configmap
    spec:
      containers:
      - image: luksa/fortune:args
        env:
        - name: INTERVAL
          valueFrom:
            configMapKeyRef:
              name: fortune-config
              key: sleep-interval
        args: ["$(INTERVAL)"]
    ...

#### A pod with ConfigMap entries mounted as files

    apiVersion: v1
    kind: Pod
    metadata:
      name: fortune-configmap-volume
    spec:
      containers:
      - image: nginx:alpine
        name: web-server
        volumeMounts:
        ...
        - name: config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        ...
      volumes:
      ...
      - name: config
        configMap:
          name: fortune-config
      ...

#### A pod with a specific ConfigMap entry mounted into a file directory

    volumes:
      - name: config
    configMap:
      name: fortune-config
      items:
      - key: my-nginx-config.conf
        path: gzip.conf

### Updating an app’s config without having to restart the app
#### Editing a ConfigMap

    $ kubectl edit configmap fortune-config
    $ kubectl exec fortune-configmap-volume -c web-server
    ㄴ  cat /etc/nginx/conf.d/my-nginx-config.conf

#### Signaling Nginx to reload the config

    $ kubectl exec fortune-configmap-volume -c web-server -- nginx -s reload

One big caveat relates to updating ConfigMap-backed volumes. If you’ve mounted a single file in the container instead of the whole volume, the file will not be updated.

## Using Secrets to pass sensitive data to containers
To store and distribute such information, Kubernetes provides a separate object called a Secret. Secrets are much like ConfigMaps—they’re also maps that hold key-value pairs. They can be used the same way as a ConfigMap. You can
- Pass Secret entries to the container as environment variables
- Expose Secret entries as files in a volume

#### Creating a Secret

    $ openssl genrsa -out https.key 2048
    $ openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj CN=www.kubia-example.com

    $ kubectl create secret generic fortune-https --from-file=https.key
    ㄴ   --from-file=https.cert --from-file=foo
    secret "fortune-https" created

### ConfigMap vs Secret
The contents of a Secret’s entries are shown as Base64-encoded strings, whereas those of a ConfigMap are shown in clear text. 
The maximum size of a Secret is limited to 1MB.

#### stringData field

    kind: Secret
    apiVersion: v1
    stringData:
      foo: plain text
    data:
      https.cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCekNDQ...
      https.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcE...

The stringData field is write-only (note: write-only, not read-only). It can only be used to set values. When you retrieve the Secret’s YAML with kubectl get -o yaml, the stringData field will not be shown. Instead, all entries you specified in the stringData field (such as the foo entry in the previous example) will be shown under data and will be Base64-encoded like all the other entries.
