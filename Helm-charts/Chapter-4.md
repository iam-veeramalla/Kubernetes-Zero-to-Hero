Deploy a simple python application by using Helm chat:
------------------------------------------------------

Toipc:
-------

    Designing a Python application with REST API
    Docker Container creation and management
    Pushing Docker images to repositories
    Crafting Helm Charts for your Python REST API app
    Helm chart installation and post-installation verification


* Once we have created a Docker image we have to create a helm chat with a required name.

      helm create <chat_name>
* once we created the chat go to the chat folder and edit the chart.yaml file.

       in the chart.yaml file we have to comment our the application version cause we are not using it now.
* then we have to open values.yaml file to modify the values as per our project.



         Fields:
         --------

         1. Image -> hear we have to change repo name as per our docker / ecr registry.

         By provideing this repo name helm will know about where to pull the image.

         2. Service- > hear we have to change the service type as per the requirment.


* Now we have to edit deployment.yaml file which is available in template folder



         Fields:
         --------

         1. Ports:--> hear we have to edit the containerPort what our application is useing

         2. Probes: -> if you'r using edit it. if not comment out the probes

         3. image -> in this we have to remove the app version if you are not using and if you remenber we comment it out in the
                     charts.yaml file as well.

                      if you are using app version don't comment it our in both charts.yaml file and  deployment.yaml file
