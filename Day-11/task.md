Task details
---------------
Create a multi-container pod as per the demo shown in the video.


Multi-container-pod-deployment:
-------------------------------

      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: deploy-ns1
        labels:
          dp: ns-1-deploy
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
              -  name: deploy-ns1-cont
                 image: nginx
                 ports:
                   -  containerPort: 80
                 env:
                  - name: name
                    value: 'kaki'
            initContainers:
             - name: init-cont
               image: busybox
               command: ['sh', '-c']
               args: ['until nslookup my-svc.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done'] 

# args: ['until nslookup my-svc.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done'] --> this args tells nsllokup to the service my-svc

# for every 2 sec and if it reachable it done the work.

# once my-scv service is reached then the nginx container will started.



my-svc.yaml:
------------

    apiVersion: v1
    kind: Service
    metadata:
      name: my-svc
      labels:
        sv: dep-ns1-svc
    spec:
      type: ClusterIP
      selector:
        app: nginx
      ports:
        - port: 80
          targetPort: 80



    
