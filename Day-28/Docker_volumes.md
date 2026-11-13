Docker_Volumes:
===============

* In Docker file if you modify the Docker step then it will only recreate that particular layer rest all layes will used from the cache.
* In Docker we have two types of layes
  1. readonly-layer
  2. container layer(writeble layer)
* Readonly-container means which we are mentioned in Docker file.
* Container-Layer means which we are exec\login into the container.in this layer i will make the changes like creating files and folder in a
  conyainer.
* By using these readonly-layers we will created Writable-Layer(simply we can called it as writable copy of readonly layers).
* But these changes and modifications in the containerlayer(writable layer) was temaparary.once container was deleted the data also deleted

Now How we can do the data persistent:
-------------------------------------

1) How is responsible for writing the data in the Container-Later ?
A) Storage Driver.

* by using these storage drivers we will write and srore the data in layers formate
* it will store the readonly-layer data and container layer data into a layers.

* we have many types of storage drivers like overlay2,, zfs, vfs, aufs and, devicemapper so docker will select these driver automatically
  basedup on the drivers

2) How is make that data Persisent ?
A) Volume Drivers

* this volume drivers will have the capability of storing the data permenently

Now we can see how to create a Docker Volumes:
----------------------------------------------

* First create a Docker Volume: docker volume create tf_volume

* the above volume was created on the host mechine with the help of local storage driver.
* the docker related things will store in /var/lib/docker/

* Second to create a container with volume: docker run -v loacal_volume_name:path_inside_a_contianer -d p 3000:3000 --name=con_name --image=con_img
*  

