Helm CLI CMD'S:
-----------------

Toipc:
-----

        helm create
        helm install
        helm upgrade
        helm rollback
        Mastering debugging with helm --debug --dry-run
        helm template, helm lint, helm uninstall and more!

To Create a Helm Chat:
----------------------

     helm create <chart_name>

To intsall helm chat:
----------------------

     helm install <release_name> <chart_name>


To see list of helm chats:
--------------------------

      heml list -a


To Upgrade helm chat:
--------------------- 

      helm upgrade <release_name> <chat_name>

      if you make any changes in the helm chat at that time whave to use this CMD to upgrade our helm chat.

To RollBack:
-------------

      helm rollback <relese_name> <revision_number>

To debug and dryrun of helm chat:
---------------------------------


* These two CMD like debug and dryrun we have to use before intsallting / upgrading the helm chat after creation / after modification.
* we can easily identify the bugs before installting and upgrading.

       helm install <release_name> --debug --dry-run <chat_name>

To validate Helm chat yamls:
----------------------------

      helm template <chart_name>

* This CMD is used to validate our halm chat yaml files mainly.
* we can see all the yaml files along with the configration details.

To check the errors or misconfigations:
---------------------------------------

     helm lint <chart_name>

* This CMD will shot is there any missconfigrations / error in our helm chat

To uninstall the helm chat:
--------------------------


    helm uninstall <release_name>

      


