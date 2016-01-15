

# Introduction #

[LittleS3](http://code.google.com/p/littles3/) is distributed as a WAR file designed to be installed and run from a Java-based application server. This documentation uses [Tomcat](http://tomcat.apache.org/) as an example. The goal is to install littleS3 and be able to manually store objects within the application.

# Installation #

## Tomcat ##

To start, download and install [Tomcat](http://tomcat.apache.org/) as appropriate for your environment.

## Configuration files ##

[LittleS3](http://code.google.com/p/littles3/) uses two configuration files by default: `StorageEngine.properties` and `StorageEngine-servlet.xml`. These configuration files are loaded automatically from the classpath.

## Custom classpath ##

You can modify the Tomcat start up script to modify the default. For example...

In the tomcat home directory, `C:\apps\apache-tomcat-5.5.23` for instance, create a new directory named "`settings`".

Edit the `catalina.bat` file, found in the "`tomcat home directory/bin`" directory, adding:

```
rem Extra classpath environment stuff
set CLASSPATH=%CLASSPATH%;C:\apps\apache-tomcat-5.5.23\settings
```

to this section:

```
:noJsse
set CLASSPATH=%CLASSPATH%;%CATALINA_HOME%\bin\bootstrap.jar

rem Extra classpath environment stuff
set CLASSPATH=%CLASSPATH%;C:\apps\apache-tomcat-5.5.23\settings

if not "%CATALINA_BASE%" == "" goto gotBase
set CATALINA_BASE=%CATALINA_HOME%
:gotBase
```

The two configuration files `StorageEngine.properties` and `StorageEngine-servlet.xml` can now be placed in the new directory "`settings`".

## Deploy war ##

The littleS3 application is [distributed](http://code.google.com/p/littles3/downloads/list) as a WAR file. The WAR packaging follows the pattern

`littleS3-[version].war`: `littleS3-2.1.0.war` for instance.

A WAR file can be deployed to a default Tomcat instance by placing the WAR file in the "`tomcat home directory/webapps`" directory. Tomcat will automatically unpack and deploy the WAR. This will make the littleS3 application available at the URL:

http://localhost:8080/littleS3-2.1.0/

Where "`localhost:8080`" is the host and default port that Tomcat uses.

There are more advanced Tomcat deployment options that allow you to control the "`context path`", the "`/littleS3-2.1.0`" portion of the URL. You could deploy to a context path of "`/littleS3`", or even to the Tomcat root, so the context path would be "`/`". The context path is important when crafting the URL to access littlS3, as the [S3 protocol](http://docs.amazonwebservices.com/AmazonS3/2006-03-01/) uses URL components to create buckets.

# Configuration #

There are two configuration files for littleS3: [StorageEngine-servlet.xml](http://littles3.googlecode.com/files/StorageEngine-servlet.xml) and [StorageEngine.properties](http://littles3.googlecode.com/files/StorageEngine.properties).

  * `StorageEngine-servlet.xml` is a Spring Framework bean wiring configuration file. This controls the component makeup of the application. You typically don't have to change the configuration values in this file.
  * `StorageEngine.properties` is a configuration file that is consumed by components configured in the `StorageEngine-servlet.xml` configuration file.

## Property Names ##

<dl>
<dt>host</dt>
<dd>May include a "token" of <code>$resolvedLocalHost$</code> to have the system "resolve" the local host value. For instance, <code>$resolvedLocalHost$:8080</code>. This would use the Java methods <a href='http://java.sun.com/j2se/1.5.0/docs/api/java/net/InetAddress.html#getCanonicalHostName()'>InetAddress.getLocalHost().getCanonicalHostName()</a> to determine what the local value of the hostname is. Used by <code>com.jpeterson.littles3.StorageEngine</code>. Example: <code>www.myserver.com:8080</code></dd>

<dt>storageLocation</dt>
<dd>Identifies the local file system storage location that will subsequently be used for objects and metadata. This directory should already exist. Used by <code>com.jpeterson.littles3.dao.je.JeCentral</code>, <code>com.jpeterson.littles3.dao.filesystem.FileBucketDao</code>, <code>com.jpeterson.littles3.dao.filesystem.FileS3ObjectDao</code>. Example: <code>C:/temp/StorageEngine</code></dd>

<dt>dir.buckets</dt>
<dd>Identifies the directory under <code>storageLocation/dir.meta</code> that will be used to store bucket information. Used by <code>com.jpeterson.littles3.dao.filesystem.FileBucketDao</code>. Default value: <code>buckets</code>. Example: <code>buckets</code></dd>

<dt>dir.db</dt>
<dd>Directory under <code>storageLocation</code> that will be used for to store the <a href='http://www.oracle.com/database/berkeley-db/je/index.html'>Oracle Berkeley DB Java Edition (JE)</a> database based data. Used by <code>com.jpeterson.littles3.dao.je.JeCentral</code>. Example: <code>db</code></dd>

<dt>dir.meta</dt>
<dd>Identifies the directory under <code>storageLocation</code> that will be used to store meta information. Used by <code>com.jpeterson.littles3.dao.filesystem.FileBucketDao</code>, <code>com.jpeterson.littles3.dao.filesystem.FileS3ObjectDao</code>. Default value: <code>meta</code>. Example: <code>meta</code></dd>

<dt>dir.objects</dt>
<dd>Identifies the directory under <code>storageLocation/dir.meta</code> that will be used to store object information. Used by <code>com.jpeterson.littles3.dao.filesystem.FileS3ObjectDao</code>. Default value: <code>objects</code>. Example: <code>objects</code></dd>

<dt>db.object</dt>
<dd>Directory under <code>storageLocation/dir.db</code> where the object information is stored in an <a href='http://www.oracle.com/database/berkeley-db/je/index.html'>Oracle Berkeley DB Java Edition (JE)</a> database. Used by <code>com.jpeterson.littles3.dao.je.JeCentral</code>. Example: <code>objectDatabase</code></dd>

<dt>db.bucket</dt>
<dd>Directory under <code>storageLocation/dir.db</code> where the bucket information is stored in an <a href='http://www.oracle.com/database/berkeley-db/je/index.html'>Oracle Berkeley DB Java Edition (JE)</a> database. Used by <code>com.jpeterson.littles3.dao.je.JeCentral</code>. Example: <code>bucketDatabase</code></dd>

<dt>user.file</dt>
<dd>Sample configuration file: <a href='http://littles3.googlecode.com/files/users.config'>users.config</a></dd>
</dl>

# Source #

The source is divided into 4 different components: `api`, `webapp`, `filesystem` module, and `je` module. The `api` component provides the common objects used by the system. The `webapp` is the collection of the other components into a runnable system. The `filesystem` and `je` modules provide different persistent storage options. The `filesystem` module stores the persistent data using plain files in the file system. The `je` module store stores the persistent data using the [Oracle Berkeley DB Java Edition (JE)](http://www.oracle.com/database/berkeley-db/je/index.html) database.

# Basic Usage #

In these samples, I have used a base URL of `http://localhost:8080/littleS3-2.1.0/`. The `StorageEngine.properties` configuration parameter `host` is set to the value `localhost:8080`:

```
host=localhost:8080
```

This will allow the littleS3 system to properly parse the URL request and determine what function to perform.

## Create a bucket ##

The following REST request will create the bucket `firstBucket`.

```
curl --request PUT "http://localhost:8080/littleS3-2.1.0/firstBucket"
```

Based on the other configuration parameters and using the `filesystem` module, this command will:

  * Create the directory `C:\temp\StorageEngine`.
  * Create the directory `C:\temp\StorageEngine\buckets`.
  * Create the directory `C:\temp\StorageEngine\buckets\firstBucket`.
  * Create the directory `C:\temp\StorageEngine\meta`.
  * Create the directory `C:\temp\StorageEngine\meta\buckets`.
  * Create the directory `C:\temp\StorageEngine\meta\buckets\firstBucket`.
  * Create the file `C:\temp\StorageEngine\meta\buckets\firstBucket\firstBucket.ser`.

## Add an object ##

The following REST request will create the object `foo.html`.

```
curl --data "@foo.html" --request PUT --header "Content-Type: text/html" "http://localhost:8080/littleS3-2.1.0/firstBucket/foo.html"
```

This command will:

  * Create this directory `C:\temp\StorageEngine\buckets\firstBucket\9b`. You will most likely have a different directory name; it will be two characters, but two different characters.
  * Create the file `C:\temp\StorageEngine\buckets\firstBucket\9b\9b8295ad7ad382ef47becac31f548902`. This file contains the actual object data.
  * Create the directory `C:\temp\StorageEngine\meta\objects`.
  * Create the directory `C:\temp\StorageEngine\meta\objects\firstBucket`.
  * Create the file `C:\temp\StorageEngine\meta\objects\firstBucket\keys.ser`.
  * Create the directory `C:\temp\StorageEngine\meta\objects\firstBucket\ad`. You will most likely have a different directory name; it will be two characters, but two different characters.
  * Create the file `C:\temp\StorageEngine\meta\objects\firstBucket\ad\ad9f205559bcb9eb394e457d5b003b2a.ser`. This file stores metadata about the object.

## Retrieve an object ##

For this, you can use your browser. Open the following URL in your browser:

```
http://localhost:8080/littleS3-2.1.0/firstBucket/foo.html
```

## Delete an object ##

The following REST request will delete the object `foo.html`.

```
curl --request DELETE "http://localhost:8080/littleS3-2.1.0/firstBucket/foo.html"
```

  * Delete the directory `C:\temp\StorageEngine\buckets\firstBucket\9b` and any files in the directory. The object contents are deleted. Since this was the only object, all of the empty directories are also removed.
  * Delete the directory `C:\temp\StorageEngine\meta\objects` and any child folders and files. Since the object contents were deleted, the object metadata is removed and all of the empty directories are also removed.

## Delete a bucket ##

The following REST request will delete the object `firstBucket`.

```
curl --request DELETE "http://localhost:8080/littleS3-2.1.0/firstBucket"
```

  * Delete the directory `C:\temp\StorageEngine\buckets\firstBucket`.
  * Delete the directory `C:\temp\StorageEngine\meta\buckets`. The `firstBucket` directory is no longer needed and being the only bucket, the `buckets` directory is removed too.

## List buckets ##

For this, you can use your browser. Open the following URL in your browser:

```
http://localhost:8080/littleS3-2.1.0/
```