<!DOCTYPE html>
<html lang="en">
<head>
  <link rel="stylesheet" href="https://asciidoclive.com/assets/asciidoctor.js/css/asciidoctor.css">
  <link rel="stylesheet" href="./resources/asciidoctor.css">
</head>

<body class="cbody article toc2 toc-left">
<div id="header">
<h1>Configure Red Hat Process Automation Manager with external database</h1>
<div id="toc" class="toc2">
<div id="toctitle">Table of Contents</div>
<ul class="sectlevel1">
<li><a href="#_background">Background</a></li>
<li><a href="#_prerequisites">Prerequisites</a></li>
<li><a href="#_steps">Steps</a></li>
</ul>
</div>
</div>
<div id="content">
<div class="sect1">
<h2 id="_background">Background</h2>
<div class="sectionbody">
<div class="paragraph">
<p>Business processes are long running and stateful in nature, which means they will require some kind of persistence to store process related data. Red Hat Process Automation manager supports many database for persisting process related data.
<BR><BR>
In this article we will look at how to use an external database with Red Hat Process Automation Manager. Red Hat Process Automation Manager running on JBoss EAP uses the JDBC client connectivity via JBoss EPA to establish connection with an external database such as MySql, Postgres etc. For the purpose of this article we will be using the below software versions:
<ul>
<li>Jboss EAP 7.3</li>
<li>Red Hat Process Automation Manager 7.10</li>
<li>MySQL Server 5.7</li>
<li>MySQL JDBC Driver mysql-connector-java-8.0.20</li>
</ul>
</div>
</div>
</div>
<div class="sect1">
<h2 id="_prerequisites">Prerequisites</h2>
<div class="sectionbody">
<div class="ulist">
<ul>
<li>Red Hat JBoss EAP 7.3 installed and running</li>
<li>Red Hat Process Automation Manager 7.10 installed and running on JBoss EAP</li>
<li>Access to a running instance of MySQL Server 5.7</li>
<li>MySQL JDBC Driver mysql-connector-java-8.0.20</li>
</ul>
</div>
<div class="sect1">
<h2 id="_steps">Steps</h2>
<div class="ulist">
<ul>
<li>Configuring the JDBC Driver on JBoss EAP</li>
<li>Configuring MySQL database</li>
<li>Configuring KIE Server to connect to the Database</li>
</ul>
	
</div>

<div class="sectionbody">
<div class="ulist">
<h4 id="_steps">Configuring the JDBC Driver on JBoss EAP</h4>
<div class="ulist">
<p>Follow the steps below to install the JDBC driver for the MySQL database</p>
<ol type="1">
<li>Download the MySQL JDBC driver from the providers website <a href="https://dev.mysql.com/downloads/connector/j/">MySQL connector - java</a>. 
Select Operating System - Platform Independent</li>
<li>Extract the downloaded files to a temporary location</li>
<li>Install the JDBC driver on JBoss EAP, we will be installing the JDBC driver as a core module following steps</li>
<ol type="a">
<li>Start the JBoss EAP server if not already running</li>
<li>Go to the command line and go to the location EAP_HOME/bin</li>
<li>Run the jboss-cli.sh</li>
<li>Connect to the server – type <B>connect</B> or <B>connect host:port</B> to connect to the server instance</li>
<li>Use the command as shown below to add the new core module<br>
<div class="code"><pre>
module add --name=com.mysql --resources=/path/to/mysql-connector-java-8.0.20.jar --dependencies=javax.api,javax.transaction.api</pre>
</div>
</li>
</ol>
<li>Next register the JDBC Driver for it to be referenced by the application datasource
<ol type="a"><li>Run the following command</li></ol>
<div class="code"><pre>
/subsystem=datasources/jdbc-driver=mysql:add(driver-name=mysql,driver-module-name=com.mysql,driver-xa-datasource-class-name=com.mysql.cj.jdbc.MysqlXADataSource, driver-class-name=com.mysql.cj.jdbc.Driver)
</pre></div><br>
</li>
<li>This will make the MySQL JDBC driver available for the application’s (Red Hat Process Automation Manager in our case) to reference the data source.</li>
</ol>
<i>If using KIE servers in a clustered environment the above steps need to be completed on each machine that will run an instance of the KIE Server.</i>
<h4 id="_steps">Configuring MySQL database</h4>
<div class="ulist">
Next step is to configure the MySQL database by running the database scripts for creating the schema and the objects needed by the KIE Server
<ul>
<li>Download the Red Hat Process Automation Manager 7.10.0 Add On zip file (rhpam-7.10.0.add-ons.zip) from the Software Download Page.</li>
<li>Extract the add ons zip to a temporary directory</li>
<li>Navigate to << TEMP DIR >>/rhpam-7.10.0-migration-tool/ddl-scripts and execute the DDL script for your database type, in our case run the mysql5-jbpm-schema.sql scripts for MySQL DB. This will create the necessary schema and object.</li>
</ul>

<h4 id="_steps">Configuring KIE Server to connect to the Database</h4>
<div class="ulist">
Next we will configure the KIE Server to connect to the MySQL database, the same steps can be used for other supported databases.
<ol type="1">
<li>Navigate to the EAP_HOME/standalone/configuration folder</li>
<li>Opens the standalone-full.xml</li>
<li>Add the following properties under the &lt;system-properties> tag <div class="code"><pre>
    &lt;!-- Data source properties. -->
    &lt;property name="org.kie.server.persistence.ds" value="java:/MySqlDSXA"/>
    &lt;property name="org.kie.server.persistence.dialect" value="org.hibernate.dialect.MySQL5Dialect"/></pre></div>
<BR>
<div class="code">
Below is the list of supported dialects
<ul>
<li>DB2: org.hibernate.dialect.DB2Dialect</li>
<li>MSSQL: org.hibernate.dialect.SQLServer2012Dialect</li>
<li>MySQL: org.hibernate.dialect.MySQL5InnoDBDialect</li>
<li>MariaDB: org.hibernate.dialect.MySQL5InnoDBDialect</li>
<li>Oracle: org.hibernate.dialect.Oracle10gDialect</li>
<li>PostgreSQL: org.hibernate.dialect.PostgreSQL82Dialect</li>
<li>PostgreSQL plus: org.hibernate.dialect.PostgresPlusDialect</li>
<li>Sybase: org.hibernate.dialect.SybaseASE157Dialect</li>
</ul></div>
</li>

<li>Next we will configure the datasource and the driver in the standalone-*.xml</li>
<li>Search for the &lt;subsystem xmlns="urn:jboss:domain:datasources:5.0"> section and add a new section for the &lt;datasource> as shown below
<div class="code"><pre>
&lt;datasource jndi-name="java:/MySqlDS" pool-name="MySqlDS" use-java-context="true" enabled="true">
    &lt;datasource-property name="DatabaseName">&lt;&lt;DB Name>>&lt;/datasource-property>
    &lt;datasource-property name="PortNumber">&lt;&lt;DB Port>>&lt;/datasource-property>
    &lt;datasource-property name="ServerName">&lt;&lt;DB host>>&lt;/datasource-property>
    &lt;driver>mysql&lt;/driver>
    &lt;security>
      &lt;user-name>&lt;&lt;db user name>>&lt;/user-name>
      &lt;password>&lt;&lt;db password>>&lt;/password>
    &lt;/security>
&lt;/datasource></pre>
</div><br>
</li>
<li>Next add the entry for the driver as shown below in the same section under the &lt;drivers subsection as shown below
<div class="code"><pre>
&lt;drivers>
.
.
.
&lt;driver name="mysql" module="com.mysql">
     &lt;driver-class>com.mysql.cj.jdbc.Driver</driver-class>
&lt;xa-datasource-class>com.mysql.cj.jdbc.MysqlDataSource</xa-datasource-class>
&lt;/driver>
&lt;/drivers>
</pre>
</div><br>
</li>
<li>Next search for &lt;default-bindings … or datasource="java:jboss/datasources/ExampleDS" and replace the datasource name from java:jboss/datasources/ExampleDS to java:/MySqlDS this is the same as the the jndi-name created above in step 5</li>

<div class="code"><pre>
&lt;default-bindings context-service="java:jboss/ee/concurrency/context/default" datasource="java:jboss/datasources/ExampleDS" </pre>
</div>
<pre>							To</pre>
<div class="code"><pre>
&lt;default-bindings context-service="java:jboss/ee/concurrency/context/default" datasource="java:/MySqlDSXA"</pre>
</div>
<li>Restart the server if everything goes well you should be connected to the database. If all goes well then you should see the database connection enteries in the logs.
	<br><br><img src="https://github.com/rohitralhan/RHPAMDB/blob/main/resources/images/pamcli.png">
	</li>
</ol>

</body>

</html>
