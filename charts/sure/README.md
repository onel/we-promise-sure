--- a/charts/sure/README.md
+++ b/charts/sure/README.md
@@ -358,6 +358,60 @@
             app.kubernetes.io/instance: sure  # verify labels on your cluster
 ```
 
+#### Rolling update strategy
+
+When using topology spread constraints with `whenUnsatisfiable: DoNotSchedule`, the default Kubernetes rolling update strategy (`maxUnavailable=0`, `maxSurge=25%`) can cause deployment deadlocks. This happens because:
+
+1. With `maxSurge > 0`, Kubernetes tries to create new pods before terminating old ones
+2. If all nodes are occupied and topology constraints require spreading, new pods cannot be scheduled
+3. Old pods won't terminate until new pods are ready, creating a deadlock
+
+To prevent this, the chart now configures a rolling update strategy for web and worker deployments with safer defaults:
+
+```yaml
+# Default strategy (changed from Kubernetes defaults)
+strategy:
+  type: RollingUpdate
+  rollingUpdate:
+    maxUnavailable: 1  # was 0
+    maxSurge: 0        # was 25%
+```
+
+This strategy allows one pod to be unavailable during updates, ensuring old pods terminate before new ones are created, which prevents scheduling deadlocks when using strict topology constraints.
+
+**Customizing the strategy:**
+
+You can override these defaults per workload:
+
+```yaml
+web:
+  strategy:
+    type: RollingUpdate
+    rollingUpdate:
+      maxUnavailable: 2
+      maxSurge: 0
+
+worker:
+  strategy:
+    type: RollingUpdate
+    rollingUpdate:
+      maxUnavailable: 1
+      maxSurge: 1  # safe if you have spare node capacity
+```
+
+**Important considerations:**
+
+- **With `DoNotSchedule` topology constraints**: Keep `maxSurge: 0` to avoid deadlocks when all nodes are occupied
+- **With `ScheduleAnyway` topology constraints**: You can safely use `maxSurge > 0` since pods can be scheduled even if spreading is violated
+- **High availability**: Setting `maxUnavailable: 1` with multiple replicas ensures at least one pod remains available during updates
+- **Faster updates**: If you have spare node capacity, you can increase `maxSurge` to speed up deployments, but be aware of the deadlock risk with strict topology constraints
+
+**Testing your configuration:**
+
+```bash
+# Verify the strategy is applied
+kubectl get deployment sure-web -n sure -o yaml | yq '.spec.strategy'
+```
+
 Security note on label selectors:
 - Choose selectors that uniquely match the intended pods to avoid cross-app interference. Good candidates are:
   - CNPG: `cnpg.io/cluster: <cluster-name>` (CNPG labels its pods)
