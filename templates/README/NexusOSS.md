# Nexus OSS

## Setup

On initial install, monitor the container logs, it will likely appear to hang as it provisions the users and roles in the built-in database.
It will continue once this process has finished, and present a message saying it is online.

    -------------------------------------------------
    
    Started Sonatype Nexus OSS 3.30.1-01
    
    -------------------------------------------------

Open a terminal window on unraid, and run the following command to retreive the intiial admin password for initial login. You will have to update the path if you have change it from the default.

    cat /mnt/user/appdata/nexus-data/admin.password


With the intiial admin password in hand, open the Web GUI and click sign in at the top right, using admin for username, with your intial admin password. Follow the wizard to set a new password for admin, as well as enable/disable anonymous read access. You most likely want to enable anonymous read access, unless this is to be hosted publicly with private images/packages.

### Docker Registry Setup

[Nexus Documentation](https://help.sonatype.com/repomanager3/formats/docker-registry)

If you didn't define a Docker Registry port when installing this app on unraid, then you will need to edit the app and define one. You should use a 1:1 port mapping between Host and Container (ex. 5000 Host and 5000 Container).

With a Docker Registry port defined, sign into Nexus OSS as admin, and click the settings cog on the top bar, then Repositories on the sidebar. On the Repositories page, click the Create Repository button, and select docker (hosted).

* Provide Repository Name (this is used for Nexus only)
* Check HTTP and provide your Docker Registry port defined earlier in unraid.
* (Optional) Check Allow anonymous docker pull (if enabled, follow additional steps below)
* Check Enable Docker V1 API
* Create Repository

If you have selected to Allow anonymous docker pull, click Realms under the Security section on the sidebar, and set Docker Bearer Token Realm to Active. 

Nexus OSS does not allow for anonymous push by default, so you will need to create a role and user, and add permissions to that user to enable push to the registry. Click Roles under the Security section on the sidebar, and then Create Role -> Nexus Role.

* Provide an ID and name (such as docker-push and Docker Push)
* Add the `nx-repository-view-docker-*-add` or `nx-repository-view-docker-<your-registry-name>-add` permission
* Add the `nx-repositroy-view-docker-*-edit` or `nx-repository-view-docker-<your-registry-name>-edit` permission. 
* Create role

Click Users under the Security section on the sidebar, and add a new user, and add the docker push role you just created. The registry is now ready to use as follows:

* Add unraid IP:Port to `insecure-registries` on your Docker installation on your machine that you will push/pull from. [See documentation](https://docs.docker.com/registry/insecure/#deploy-a-plain-http-registry)
* Run `docker login IP:Port` using your unraid IP and Docker Registry port. Provide the new Nexus username and password.
* Build/Tag image with unraid IP:Port/image:tag `docker tag ubuntu:20.04 my.unraid.ip.here:5000/ubuntu:20.04` (tags can contain repo prefix such as repo/ubuntu:20.04)
* Push image `docker push my.unraid.ip.here:5000/ubuntu:20.04`
