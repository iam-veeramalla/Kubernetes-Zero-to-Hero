Helm Repo
==========


Topics:
-------

      How to use helm repo cli ?
      how to search diff charts ?



How to use helm repo cli ?
----------------------------

* These Helm repo's are pre defined charts we can simple add and install those repoes in our cluster.
* All the predefined helm chats will be store in helm repo.

  To search the helm chart in repo:

      helm search hub <actual chat we are looking for>

To see the comple repo url in the search use filter <--max-col-width=0>:

      helm search hub <actual chat we are looking for> --max-col-width=0

To verify the remote repo's in our mechine:

      helm repo list

To add the repo like bitnami it is a famous repo it contains many charts:

      helm repo add bitnami url
To view the readme file of the particular chat in the bitnami repo:

      helm show readme <repo_name>/chart_name --version <version_tag>
