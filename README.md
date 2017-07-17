# Red Hat Fuse 6.3 Workshop - Fuse Integration Service 2.0

This very simple lab will guide you to create your very first Fuse SpringBoot project running on OpenShift. There are 3 sections in the lab.

* Create a project that read from a database
* Expose a restful API endpoint to access data in the database
* Deploy your application on OpenShift

## Installation
Before you begin, please make sure the following software are properly installed

* JBoss Development Suite V1.3 (MacOX/Windows)
	* JBoss Developer Studio 10.3.0.GA with Integration SOA plugin installed
	https://developers.redhat.com/products/devsuite/download/
	* Java Platform, Standard Edition 1.8.0.111
	* Red Hat Container Development Kit 2.4.0.GA
	* Oracle Virtualbox 5.0.26
	* Vagrant 1.8.1

## Installing and setup development environment
Double click on the JBoss Development Suite, log in using your Red Hat Developer Site credentials.

![01-login.png](./img/01-login.png)

Pick an installation folder destination.
The installer guide will detect the components needed, and guide you through the installation process. 

Installed the components with the version specified in the installer and start to download and install.

![02-components.png](./img/02-components.png)

Immediately after installation, you will be prompted to select a workspace for your developer studio project. Select anything path of your choice.

Once inside Red Hat JBoss Developer Studio, select "Software/Update" tag in the middle panel. Check the "JBoss Fuse Development" box and click on Install/Update button.

![04-plugin.png](./img/04-plugin.png)

Red Hat JBoss Developer Studio will restart.

## Installing and setup Container Development Kit

Under the folder where you installed the Development Suite, you will find a folder named **cdk** go to **${DEVSUITE_INSTALLTION_PATH}/cdk/components/rhel/rhel-ose/** edit file **Vagrantfile**

Find the IMAGE_TAG and configure the OCP version to v3.4, then save the file.

```
#Modify IMAGE_TAG if you need a new OCP version e.g. IMAGE_TAG="v3.3.1.3"
IMAGE_TAG="v3.4"
```

In a command line console, start up your local Openshift

```
vagrant up
```

Install and setup oc binary client

```
vagrant service-manager install-cli openshift
export PATH=${vagrant_dir}/data/service-manager/bin/openshift/1.4.0:$PATH
eval "$(VAGRANT_NO_COLOR=1 vagrant service-manager install-cli openshift  | tr -d '\r')"
```

Login as admin 

```
oc login -u admin
Authentication required for https://10.1.2.2:8443 (openshift)
Username: admin
Password: 
Login successful.

```

Install Fuse image stream on OpenShift and Database template for this lab

```
#FIS image
oc create -f https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fis-image-streams.json -n openshift

#MYSQL Database
oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/db-templates/mysql-ephemeral-template.json -n openshift
```

log back in as developer

```
oc login -u developer
Authentication required for https://10.1.2.2:8443 (openshift)
Username: developer
Password: 
Login successful.

```

Access OpenShift console by going to the following URL in the browser.

```
https://10.1.2.2:8443
```

Going back to Red Hat JBoss Developer Studio, in OpenShift Explorer view, click on **New Connection Wizard..** to configure OpenShift setting
Enter **https://10.1.2.2:8443** as the **Server** and click on the **retrieve** link to access the token.

![05-token.png](./img/05-token.png)

In the popup window, log in as Developer using ID/PWD developer/developer. Select ok and check the **Save token** box.

![06-connection.png](./img/06-connection.png)

## Windows Users

- Make sure you disable  Hyper-V functionality under Control Panel 
- Add _config.ssh.insert\_key=false_ to **Vagrantfile** ${DEVSUITE_INSTALLTION_PATH}/cdk/components/rhel/rhel-ose/ 

Thanks to @sigreen 

## FAQ
- How to install Maven?  
	- Go to https://maven.apache.org/install.html for detail instructions
- Maven dependency not found? 
	- ${MAVENI_INSTALLED_DIR} if you are having trouble downloading from the repositories
