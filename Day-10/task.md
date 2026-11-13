Task details:
=============
1. Create two namespaces and name them ns1 and ns2

ns-test.yaml:

          apiVersion: v1
          kind: Namespace
          metadata:
            name: test-ns


ns-demo.yaml:

       apiVersion: v1
      kind: Namespace
      metadata:
        name: demo-ns 


2. Create a deployment with a single replica in each of these namespaces with the image as nginx and name as deploy-ns1 and deploy-ns2, respectively

deploy-ns1.yaml: 

      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: deploy-ns1
        labels:
          dp: ns-1-deploy
        namespace: test-ns
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

deploy-ns2.yaml:

      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: deploy-ns2
        labels:
          dp: ns-2-deploy
        namespace: demo-ns
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
              -  name: deploy-ns2-cont
                 image: nginx
                 ports:
                   -  containerPort: 80


3. Get the IP address of each of the pods (Remember the kubectl command for that?)

        kubectl get po -n=test-ns -o wide

        kubectl get po -n=demo-ns -o wide

   
4. Exec into the pod of deploy-ns1 and try to curl the IP address of the pod running on deploy-ns2



        kubectl exec -it deploy-ns1-bcd649cfd-2dvx4 -n=test-ns -- /bin/bash

        curl Pod_IP




5. Your pod-to-pod connection should work, and you should be able to get a successful response back.

Out Put:
---------
            <!DOCTYPE html>
            <html>
            <head>
            <title>Welcome to nginx!</title>
            <style>
            html { color-scheme: light dark; }
            body { width: 35em; margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif; }
            </style>
            </head>
            <body>
            <h1>Welcome to nginx!</h1>
            <p>If you see this page, nginx is "**successfully**" installed and working.
            Further configuration is required for the web server, reverse proxy,
            API gateway, load balancer, content cache, or other features.</p>
            
            <p>For online documentation and support please refer to
            <a href="https://nginx.org/">nginx.org</a>.<br/>
            To engage with the community please visit
            <a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
            For enterprise grade support, professional services, additional
            security features and capabilities please refer to
            <a href="https://f5.com/nginx">f5.com/nginx</a>.</p>
            
            <p><em>Thank you for using nginx.</em></p>
            </body>
            </html>


6. Now scale both of your deployments from 1 to 3 replicas.

        kubectl scale deploy deploy-ns2 -n=demo-ns --current-replicas=1 --replicas=3

        kubectl scale deploy deploy-ns1 -n=test-ns --current-replicas=1 --replicas=3

   
7. Create two services to expose both of your deployments and name them svc-ns1 and svc-ns2.


test-ns-depl-svc:

      apiVersion: v1
      kind: Service
      metadata:
        name: dep-ns1-svc
        labels:
          sv: dep-ns1-svc
        namespace: test-ns
      spec:
        type: ClusterIP
        selector:
          app: nginx
        ports:
          - port: 80
            targetPort: 80


demo-ns-deploy-svc:

      apiVersion: v1
      kind: Service
      metadata:
        name: cl-sv
        labels:
          sv: cl-svc-prod
        namespace: demo-ns
      spec:
        type: ClusterIP
        selector:
          app: nginx
        ports:
          - port: 80
            targetPort: 80



8. exec into each pod and try to curl the IP address of the service running on the other namespace.


        kubectl exec -it deploy-ns1-bcd649cfd-2dvx4 -n=test-ns -- /bin/bash
        curl Service_IP --> another ns serviceIP




9. This curl should work.

OutPut:
-------

      <!DOCTYPE html>
      <html>
      <head>
      <title>Welcome to nginx!</title>
      <style>
      html { color-scheme: light dark; }
      body { width: 35em; margin: 0 auto;
      font-family: Tahoma, Verdana, Arial, sans-serif; }
      </style>
      </head>
      <body>
      <h1>Welcome to nginx!</h1>
      <p>If you see this page, nginx is successfully installed and working.
      Further configuration is required for the web server, reverse proxy,
      API gateway, load balancer, content cache, or other features.</p>
      
      <p>For online documentation and support please refer to
      <a href="https://nginx.org/">nginx.org</a>.<br/>
      To engage with the community please visit
      <a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
      For enterprise grade support, professional services, additional
      security features and capabilities please refer to
      <a href="https://f5.com/nginx">f5.com/nginx</a>.</p>
      
      <p><em>Thank you for using nginx.</em></p>
      </body>
      </html>

10. Now try curling the service name instead of IP. You will notice that you are getting an error and cannot resolve the host.

output:
-------

      root@deploy-ns1-bcd649cfd-2dvx4:/# curl cl-sv
      curl: (6) Could not resolve host: cl-sv

11. Now use the FQDN of the service and try to curl again, this should work.

    * in this step when i try to clur the FQSN i will unable to reach the service.
   
    * Refer the issue in k8s-troublesoot_repo
In the end, delete both the namespaces, which should delete the services and deployments underneath them.
