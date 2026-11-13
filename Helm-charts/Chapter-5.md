Helm File
==========

Topics:
--------

    What is Helmfile? And why is it crucial?
    Installing Helmfile seamlessly
    Effective Helmchart management using Helmfile
    Installing Helmcharts from Git Repos and managing multiple Helmcharts with Helmfile


 What is Helmfile? And why is it crucial?
 ----------------------------------------

 * Helm file will not come with helm installtion by default. we have to install it seperatly.
 * By using this helm file CMD we can manage our helm files very easily insted of using helm chat CMD like helm install, delete, upgrade .. ect.

Eg:
---
 * if you want to install and uninstall the helm chat we have to use two helm chat CMS:

        To install:     helm install <resource_name> <chat_name>
         To uninstall: helm uninstall <resource_name>

* By by using the helmfile CDM we can do above two actions with a single a CMD:

          helmfile sync

* To manage a helm chats with the helm file we have to create a helmfile.yaml

           ---
           releases:
             - name: <release_name_of_chart>
               chart: <actual_chat_name_with_directory_path>
               installed: true ----------------------------------------->set true for installtion. if you set <false> it will uninstall the chat.

Installation of helmfile:
---------------------------

Step: 1 

      Go to helmfile installation github repo ---> release binaries --copy the below link:
      wget https://github.com/helmfile/helmfile/releases/download/v1.8.0/helmfile_1.8.0_darwin_amd64.tar.gz


Step: 2

      untar -zxvf file_name
      after untar it just renama the folder to helmfile

step: 3

      give the executable permissions to the folder:
        chmod +x helmfile

step: 4

      move file to /use/local/bin
      mv helmfile /usr/local/bin

step: 5

      check the hemlfile version

      helmfile -version


Now we can see how we can manage the helmchart with the help of helmfile:
-------------------------------------------------------------------------

Step: 1 

      create the basic helm chat with CMD:

      helm create <hellowworld>

step: 2


     create a helmfile.yaml:

     ---
     releases:
             - name: <release_name_of_chart>
               chart: <actual_chat_name_with_directory_path>
               installed: true

Step: 3

    after modifing the helmfile.yaml file with as per you'r helm chat

    just run the CMD:

       helmfile sync


Step: 4

      to unstall the helm chat:

       change the value in helmfile of the installed from true to false.
       then run again the helmfile sync CMD.
     

Now we can see How to download and install the helm chat from the git hub and install it.
------------------------------------------------------------------------------------------

Step: 1

       we have to install a plugin called helm-git

       Go to https://github.com/aslafy-z/helm-git and install the plugin

       Use helm CLI to install this plugin:
       helm plugin install https://github.com/aslafy-z/helm-git --version 1.5.2

       Uninstall:
       helm plugin remove helm-git

Usage: 

* helm-git will package any chart that is not so you can directly reference paths to original charts.

Here's the Git urls format, followed by examples:

         git+https://[username[:password]@]provider.com/<path/to/repo>[@path/to/charts][?[ref=git-ref][&sparse=1][&depupdate=0][&package=0]]
        git+ssh://[username@]provider.com/<path/to/repo>[@path/to/charts][?[ref=git-ref][&sparse=1][&depupdate=0][&package=0]]
        git+file://<path/to/repo>[@path/to/charts][?[ref=git-ref][&sparse=1][&depupdate=0][&package=0]]
        
        git+https://github.com/jetstack/cert-manager@deploy/charts?ref=v0.6.2&sparse=0
        git+ssh://git@github.com/jetstack/cert-manager@deploy/charts?ref=v0.6.2&sparse=1
        git+ssh://git@github.com/jetstack/cert-manager@deploy/charts?ref=v0.6.2
        git+https://github.com/istio/istio@install/kubernetes/helm?ref=1.5.4&sparse=0&depupdate=0
        git+https://github.com/bitnami/charts@bitnami/wordpress?ref=master&sparse=0&depupdate=0&package=0
        git+https://gitlab.com/one-touch-pipeline/weskit/helm-deployment?ref=ee259f65191cef10855438321ce99e37873918b6

Step: 2


      Just add the repo block in helmfile.yaml

      ---
          repositories:
            - name: <helmchat_name>
              url: git+url of repo
           releases:
             - name: <release_name_of_chart>
               chart: <actual_chat_name_with_directory_path>
               installed: true


Step: 3

       now if you run the helmfile sync CMD it will automatically download and install the helm chat from git


Now we can see How to install the multiple helm chats with single helmfile.yaml:
-----------------------------------------------------------------------------------

* by using the below helfile.yaml file we can install the two diff helm chat's at a time by simple divideing the charts like below.

          ---
         releases:
             - name: <release_name_of_chart_1>
               chart: <actual_chat_name_with_directory_path>
               installed: true

             - name: <release_name_of_chart_2>
               chart: <actual_chat_name_with_directory_path>
               installed: true

  
