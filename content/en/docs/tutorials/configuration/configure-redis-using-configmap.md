---
reviewers:
- eparis
- pmorie
title: Configuring Redis using a ConfigMap
content_type: tutorial
weight: 30
---

<!-- overview -->

This page provides a real world example of how to configure Redis using a ConfigMap and builds upon the [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) task. 



## {{% heading "What you'll learn" %}}

In this tutorial, you'll learn how to do the following:
* Create a ConfigMap with Redis configuration values
* Create a Redis Pod that mounts and uses the created ConfigMap
* Verify that the configuration was correctly applied



## {{% heading "Requirements" %}}


{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

* The example shown on this page works with `kubectl` 1.14 and above.
* Understand [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/).



<!-- lessoncontent -->


## Real World Example: Configuring Redis using a ConfigMap

Follow the steps below to configure a Redis cache using data stored in a ConfigMap.

1. Create a ConfigMap with an empty configuration block:

    ```shell
    cat <<EOF >./example-redis-config.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: example-redis-config
    data:
      redis-config: ""
    EOF
    ```

2. Apply the ConfigMap created above, along with a Redis pod manifest:

    ```shell
    kubectl apply -f example-redis-config.yaml
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
    ```

3. Examine the contents of the Redis pod manifest and note the following:

    * A volume named `config` is created by `spec.volumes[1]`.
    * The `key` and `path` under `spec.volumes[1].configMap.items[0]` exposes the `redis-config` key from the 
  `example-redis-config` ConfigMap as a file named `redis.conf` on the `config` volume.
    * The `config` volume is mounted at `/redis-master` by `spec.containers[0].volumeMounts[1]`.

      This exposes the data in `data.redis-config` from the `example-redis-config`
  ConfigMap above as `/redis-master/redis.conf` inside the Pod.

        {{% code_sample file="pods/config/redis-pod.yaml" %}}

4. Examine the created objects.
    1. Type the following:
        ```shell
        kubectl get pod/redis configmap/example-redis-config 
        ```

       Review the output:

        ```
        NAME        READY   STATUS    RESTARTS   AGE
        pod/redis   1/1     Running   0          8s

        NAME                             DATA   AGE
        configmap/example-redis-config   1      14s
        ```

    2. Type the following:

        ```shell
        kubectl describe configmap/example-redis-config
        ```

        Reviw the output and confirm that the `redis-config` key is empty, since it was blank in the `example-redis-config` ConfigMap.

        ```shell
        Name:         example-redis-config
        Namespace:    default
        Labels:       <none>
        Annotations:  <none>

        Data
        ====
        redis-config:
        ```

5. Review the current configuration.
    1. Type the following:
        ```shell
        kubectl exec -it redis -- redis-cli
        ```
    
    2. Review `maxmemory` and confirm that the default value is 0:

        ```shell
        127.0.0.1:6379> CONFIG GET maxmemory
        ```

    3. Review `maxmemory-policy` and confirm the default value is `noeviction`:

        ```shell
        127.0.0.1:6379> CONFIG GET maxmemory-policy
        ```

6. Add configuration values to the `example-redis-config` ConfigMap:

    {{% code_sample file="pods/config/example-redis-config.yaml" %}}

7. Apply the updated ConfigMap:

    ```shell
    kubectl apply -f example-redis-config.yaml
    ```

8. Confirm that the ConfigMap was updated:

    ```shell
    kubectl describe configmap/example-redis-config
    ```

9. Review the output for the configuration values:

    ```shell
    Name:         example-redis-config
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    redis-config:
    ----
    maxmemory 2mb
    maxmemory-policy allkeys-lru
    ```

10. Make sure that the configuration was applied:
    1. Type the following:
        ```shell
        kubectl exec -it redis -- redis-cli
        ```

    2. Review `maxmemory` and confirm the default value of 0:

        ```shell
        127.0.0.1:6379> CONFIG GET maxmemory
        ```

    3. Review `maxmemory-policy` and confirm the default value is `noeviction`:

        ```shell
        127.0.0.1:6379> CONFIG GET maxmemory-policy
        ```
    4. The configuration values haven't changed because the Pod needs to be restarted to pull updated
values from associated ConfigMaps. 

11. Delete and recreate the Pod:

    ```shell
    kubectl delete pod redis
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
    ```

12. Review the configuration values:
    1. Type the following: 
        ```shell
        kubectl exec -it redis -- redis-cli
        ```

    2. Review `maxmemory` and confirm that the value is 2097152: 

        ```shell
        127.0.0.1:6379> CONFIG GET maxmemory
        ```

    3. Review `maxmemory-policy` and confirm that the value is now `allkeys-lru`:

        ```shell
        127.0.0.1:6379> CONFIG GET maxmemory-policy
        ```

13. Optional. Type the following to delete the created resources:

    ```shell
    kubectl delete pod/redis configmap/example-redis-config
    ```

## {{% heading "whatsnext" %}}


* Learn more about [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/).
* Follow an example of [Updating configuration via a ConfigMap](/docs/tutorials/configuration/updating-configuration-via-a-configmap/).
