## Lab three - Deploying to OpenShift

Now it's time to deploy the application onto OpenShift, we have been testing with the H2 Database in memeory, now it's time to run it with a real database. Add the following datasource setting under *src/main/resources* in **application.properties**

```
#mysql specific
mysql.service.name=mysql
mysql.service.database=sampledb
mysql.service.username=dbuser
mysql.service.password=password

#Database configuration
spring.datasource.url = jdbc:mysql://${${mysql.service.name}.service.host}:${${mysql.service.name}.service.port}/${mysql.service.database}
spring.datasource.username = ${mysql.service.username}
spring.datasource.password = ${mysql.service.password}
```

Since we will be using MYSQL database, add the driver dependency in **pom.xml**

```
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <scope>runtime</scope>
</dependency>
```

In Red Hat JBoss Developer Studio,open the OpenShift Explorer view by clicking on **Window -> Show View -> Other** and search for OpenShift Explorer to open it.

Click on "New Connection Wizard" to connect to the local OpenShift instance. As Server type select "OpenShift 3". In the Server field enter your local OpenShift host, such as 'https://10.0.2.15:8443'. For Authentication, select 'Basic' with the user name 'developer' and the password 'developer'.

Right click on the connection that connects to current OpenShift, and create a new project. **NEW** -> **Project**

![01-newproject.png](./img/01-newproject.png)

And create Project Name: **myfuseproject** with Display Namw: **My Fuse Project**

![02-projectname.png](./img/02-projectname.png)

Inside the project we are going to first create a MYSQL database for our application, right click on the new project name **myfuseproject** -> **New** -> **Application**

![03-newapp.png](./img/03-newapp.png)

Under Server application source, select **mysql-ephemeral (database, mysql) - openshift** and click next.

![04-mysql.png](./img/04-mysql.png)

Make sure to configure the following parameters

```
MYSQL_PASSWORD = password
MYSQL_USER = dbuser
```
![05-param.png](./img/05-param.png)

Click Finish, and you should see the mysql instance running in OpenShift explorer.

![06-mysqlcreated.png](./img/06-mysqlcreated.png)

Now we can finally push our application to OpenShift by right click on your project in project explorer. Select **Run As** -> **Run Configurations...**

![07-runmvn.png](./img/07-runmvn.png)

In the pop-up menu, select **Deploy myfuselab on OpenShift** on the left panel. Go to  **JRE** tab on the right, inside VM arguments, update the kubernetes.master to your OpenShift host name, update kuberenets.namespace to **myfuseproject** and username/password to **developer/developer**. And **RUN**.

![08-runconfig.png](./img/08-runconfig.png)

To see everything running, in your browser, go to *https://10.1.2.2:8443/console* and login with **developer/developer**. Select **My Fuse Project**. And you will see both application in the overview page.

![09-overview.png](./img/09-overview.png)

To access the service outside OpenShift, go to **Application** -> **Service** on the left menu, and click **camel-ose-springboot-xml** in the service page.

![10-service.png](./img/10-service.png)

Click on **Create route**.

![11-createroute.png](./img/11-createroute.png)

Don't change anything and hit Create.

Access the API endpoint by going to following URL of your route and invoking the API, for example

```
curl http://camel-ose-springboot-xml-sample.rhel-cdk.10.1.2.2.xip.io/myfuselab/customer/all
```

Verify that it is returning customer data in JSON format
```
[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000},{"CUSTOMERID":"A02","VIPSTATUS":"Gold","BALANCE":500}]
```
To see the Camel route in action, in your OpenShift console, go to **Application** -> **pod** and select the first **camel-ose-springboot-xml-1-xxxxx** pod.

![12-podlist.png](./img/12-podlist.png)

Click **Open Java Console**, and it's going to take you to the indiviual console that show how your Camel route is doing.

![13-pod.png](./img/13-pod.png)

Click on **Route Diagram** and hit the URL couple of times to see what happens.

![14-javaconsole.png](./img/14-javaconsole.png)

For those of you who wants to see what is going on in database, login to the MYSQL database in your command line console.

```
oc project myfuseproject

oc get pods
NAME                                   READY     STATUS    RESTARTS   AGE
camel-ose-springboot-xml-s2i-1-build   1/1       Running   0          15s
mysql-1-xxxxx                          1/1       Running   0          2m

oc rsh mysql-1-xxxxx

sh-4.2$ mysql -udbuser -p sampledb
Enter password: 

mysql> select * from customerdemo;
+------------+-----------+---------+
| customerID | vipStatus | balance |
+------------+-----------+---------+
| A01        | Diamond   |    1000 |
| A02        | Gold      |     500 |
+------------+-----------+---------+
2 rows in set (0.00 sec)
```

Now go back into the 3scale Admin Console and to your Customers API.

In the "Integration" tab replace the "Private Base URL" with the URL of the route created earlier and port 8080, for example

      http://camel-ose-springboot-xml-sample.rhel-cdk.10.1.2.2.xip.io:8080

Then click on "Update the Staging Environment".

Test the API invocation through the 3scale Staging Environment again. You should see the list of customers, this time coming from the OpenShift instance using the Fuse Integration Services instead of through JBoss Developer Studio.
