configmap and secret
=====================

1. ConfigMaps:
   ------------

   * By using the configmaps we can maintain out application realted env variables and application related configration seperatly in configmaps.
   * we will keep out the configration data from the container.


To create a configMap in imparative way:
----------------------------------------

cmd: kubectl create cm cm_name --from-literal=key_1=value_1 \ --from-literal=key_2=value_2


To inject the configMap to Pod:
--------------------------------

1. Best way is Mount the configMap as a volume in pod.yaml file.
2. Use env block to specify a single env value from the configmap.
3. we can use the entire configMap so all the env varibles load into the container.
