[![](https://img.shields.io/badge/tag-authzforce-orange.svg?logo=stackoverflow)](http://stackoverflow.com/questions/tagged/authzforce)
[![Docker badge](https://img.shields.io/docker/pulls/authzforce/restful-pdp.svg)](https://hub.docker.com/r/authzforce/restful-pdp/)
[![Build Status](https://github.com/authzforce/restful-pdp/actions/workflows/maven.yml/badge.svg?branch=develop)](https://github.com/authzforce/restful-pdp/actions/workflows/maven.yml)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fauthzforce%2Frestful-pdp.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fauthzforce%2Frestful-pdp?ref=badge_shield)


# AuthzForce RESTful PDP
RESTful PDP API implementation, compliant with REST Profile of XACML 3.0. This is minimalist compared to [AuthzForce server project](http://github.com/authzforce/server) as it does not provide multi-tenant PDP/PAP but only a single PDP (per instance). Therefore, this is more suitable for microservices, or, more generally, simple applications requiring only one PDP per instance.

In particular, the project provides the following (Maven groupId:artifactId):
* `org.ow2.authzforce:authzforce-ce-restful-pdp-cxf-spring-boot-server`: a fully executable RESTful XACML PDP server (runnable from the command-line), packaged as a [Spring Boot application](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment-install.html) or [Docker image](https://hub.docker.com/repository/docker/authzforce/restful-pdp) (see the [Docker Compose example](docker) for usage).
* `org.ow2.authzforce:authzforce-ce-restful-pdp-jaxrs`: pure JAX-RS implementation of a PDP service, that you can reuse as a library with any JAX-RS framework, especially other than Apache CXF, to provide your own custom RESTful PDP service.

**Go to the [releases](https://github.com/authzforce/restful-pdp/releases) page for
specific release info: downloads (Linux packages), Docker image,
[release notes](CHANGELOG.md)**

## Features
### XACML PDP engine
See [AuthzForce Core features](https://github.com/authzforce/core#features) for the XACML PDP engine's features.

### REST API
* Conformance with [REST Profile of XACML v3.0 Version 1.0](http://docs.oasis-open.org/xacml/xacml-rest/v1.0/xacml-rest-v1.0.html)
* Supported data formats, aka content types: 
	
	* `application/xacml+xml`: XACML 3.0/XML content, as defined by [RFC 7061](https://tools.ietf.org/html/rfc7061), for XACML Request/Response only;
	* `application/xml`: same as `application/xacml+xml`;
	* `application/xacml+json`: XACML 3.0/JSON Request/Response, as defined by [XACML v3.0 - JSON Profile Version 1.0](http://docs.oasis-open.org/xacml/xacml-json-http/v1.0/xacml-json-http-v1.0.html);
	* `application/json`: same as `application/xacml+json`.

## Limitations
See [AuthzForce Core limitations](https://github.com/authzforce/core#limitations).

## System requirements 
Java (JRE) 17 or later.


## Versions
See the [change log](CHANGELOG.md) following the *Keep a CHANGELOG* [conventions](http://keepachangelog.com/).

## License
See the [license file](LICENSE).

## Getting started

Launch the PDP with either Docker or the executable JAR as described in the next sections.

### Using Docker

Git clone this github repository or download the Source code ZIP from the [latest release](https://github.com/authzforce/restful-pdp/releases) and unzip it, then from the git clone / unzipped folder, go to the [`docker`](docker) directory.

If you wish to use a different XACML Policy from the one provided, change the `policyLocation` parameter in the `pdp/conf/pdp.xml` (PDP configuration) file in that directory accordingly.

Then run: `docker compose up -d`, then `docker compose logs` to check the PDP is up and running.

(You can change the logging verbosity by modifying the Logback configuration file `pdp/conf/logback.xml`.)

### Using the executable JAR

Get the [latest executable jar](https://repo1.maven.org/maven2/org/ow2/authzforce/authzforce-ce-restful-pdp-cxf-spring-boot-server/) from Maven Central with groupId/artifactId = `org.ow2.authzforce`/`authzforce-ce-restful-pdp-cxf-spring-boot-server`. The name of the JAR is `authzforce-ce-restful-pdp-cxf-spring-boot-server-M.m.p.jar` (replace `M.m.p` with the latest version).

Make sure it is executable (replace `M.m.p` with the current version):

```sh
chmod u+x authzforce-ce-restful-pdp-cxf-spring-boot-server-M.m.p.jar
```

Copy the content of [that folder](cxf-spring-boot-server/src/test/resources/server) to the same directory.

If you wish to use a different XACML Policy from the one provided, change the `policyLocation` parameter in the `pdp.xml` (PDP configuration) file in that directory accordingly.

Then run the executable from that directory as follows (replace `M.m.p` with the current version):

```sh
$ ./authzforce-ce-restful-pdp-cxf-spring-boot-server-M.m.p.jar
```

If it refuses to start because the TCP listening port is already used (by some other server on the system), you can change that port in file `application.yml` copied previously: uncomment and change `server.port` property value to something else (default is 8080).

You know the embedded server is up and running when you see something like this (if and only if the logger for Spring classes is at least in INFO level, according to Logback configuration file mentioned down below) :
```
... Tomcat started on port(s): 8080 (http)
```

(You can change the logging verbosity by modifying the Logback configuration file `logback.xml` copied previously.)

### Send an XACML Request to the PDP

Once the PDP is up and running, you can make a XACML request from a different terminal, for example using the XACML/JSON request in [that folder](cxf-spring-boot-server/src/test/resources/server/IIA001) (install `curl` tool if you don't have it already on your system):

```sh
$ curl --include --header "Content-Type: application/xacml+json" --data @IIA001/Request.json http://localhost:8080/services/pdp
```
*Add --verbose option for more details.*
You should get a XACML/JSON response such as:

```
{"Response":[{"Decision":"Permit"}]}
```


## Extensions
If you are missing features in AuthzForce, you can extend it with various types of plugins (without changing the existing code), as described on AuthzForce Core's [wiki](https://github.com/authzforce/core/wiki/Extensions).

In order to use them, put the extension JAR(s) into an `extensions` folder in the same directory as the executable jar, already present if you followed the previous *Getting started* section. If the extension(s) use XML configuration (e.g. AttributeProvider), add the schema import into `pdp-ext.xsd` (import namespace only, do not specify schema location) and schema namespace-to-location mapping into `catalog.xml`. Then run the executable as follows (replace `M.m.p` with the current version):

```sh
$ java -Dloader.path=extensions -jar authzforce-ce-restful-pdp-cxf-spring-boot-server-M.m.p.jar
```

### Example with MongoDBPolicyProvider extension
To use the Policy Provider for policies stored in MongoDB, please make sure the JAR with the MongoDB policy provider, i.e. the `authzforce-ce-core-pdp-testutils` module (in the **same version** as `authzforce-ce-core-pdp-engine` that is already included in AuthzForce RESTful PDP) is on the classpath, eg. in the *extensions* folder mentioned above, with *and all its required dependencies*. The main dependencies (looking at the pom of `pdp-testutils` module) in Maven terms are:

```xml
<dependency>
         <groupId>org.jongo</groupId>
         <artifactId>jongo</artifactId>
	 <!-- Set the version to whatever version is specified in authzforce-ce-core-pdp-testutils Maven POM.  -->
         <version>${jongo.version}</version>
</dependency>
<dependency>
         <groupId>org.mongodb</groupId>
         <artifactId>mongodb-driver-legacy</artifactId>
	<!-- Set the version to whatever version is specified in authzforce-ce-core-pdp-testutils Maven POM. -->
         <version>${mongodb-driver-legacy.version}</version>
</dependency>
```

These dependencies have dependencies as well, so make sure to include them all, if not already on the classpath. (There is a way to assemble all jars in a dependency tree automatically with Maven.)

Then do steps 2 to 4 of [Using Policy Providers](https://github.com/authzforce/core/wiki/Policy-Providers#using-policy-providers), that is to say:
1. Add this import to PDP extensions schema (`pdp-ext.xsd`) to allow using the extension(s) from the `authzforce-ce-core-pdp-testutils` module in PDP configuration:
    ```xml
    <xs:import namespace="http://authzforce.github.io/core/xmlns/test/3" />
    ```
1. Add an entry to the XML catalog (`catalog.xml`) to locate the schema corresponding to this namespace:
    ```xml
    <uri name="http://authzforce.github.io/core/xmlns/test/3" uri="classpath:org.ow2.authzforce.core.pdp.testutil.ext.xsd" />
    ```
1. Add the `policyProvider` element to the PDP configuration (`pdp.xml`), using the new namespace above, like in [this example](https://github.com/authzforce/core/blob/master/pdp-testutils/src/test/resources/org/ow2/authzforce/core/pdp/testutil/test/pdp.xml) (follow the link).

[More info](https://github.com/authzforce/core/wiki/Policy-Providers#more-info-on-the-mongodbpolicyprovider).

## Vulnerability reporting
If you want to report a vulnerability, please follow the [GitHub procedure for private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability#privately-reporting-a-security-vulnerability).


## Personal notes
### Build the project
```sh
mvn clean install
```

### Update POM dependencies
```pom
<plugin>
	<!-- Consider combining with Red Hat Victims and OSS Index. More info 
		on Victims vs. Dependency-check: https://bugzilla.redhat.com/show_bug.cgi?id=1388712 -->
	<groupId>org.owasp</groupId>
	<artifactId>dependency-check-maven</artifactId>
	<configuration>
		<!-- The plugin has numerous issues with version matching, which triggers 
			false positives so we need a "suppresion" file for those. More info: https://github.com/jeremylong/DependencyCheck/issues -->
		<nvdApiKey>e236b561-390e-4f58-a24b-a8ac1b10f5bf</nvdApiKey>
		<suppressionFile>owasp-dependency-check-suppression.xml</suppressionFile>
		<failBuildOnAnyVulnerability>true</failBuildOnAnyVulnerability>
	</configuration>
	<executions>
		<execution>
			<goals>
				<goal>check</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```
### The warning message you encountered indicates that the parameter failBuildOnAnyVulnerability is deprecated and you should use the failBuildOnCVSS parameter instead. The failBuildOnCVSS parameter allows you to specify a CVSS (Common Vulnerability Scoring System) score threshold. If any detected vulnerabilities have a CVSS score equal to or higher than this threshold, the build will fail.

Here's how to update your configuration depending on your build tool:

Maven
If you are using the OWASP Dependency-Check Maven plugin, you need to update your pom.xml to replace failBuildOnAnyVulnerability with failBuildOnCVSS.

Before:
```xml
<plugin>
<groupId>org.owasp</groupId>
<artifactId>dependency-check-maven</artifactId>
<version>6.5.0</version>
<configuration>
<failBuildOnAnyVulnerability>true</failBuildOnAnyVulnerability>
<!-- other configurations -->
</configuration>
</plugin>

```
After:
```xml
<plugin>
<groupId>org.owasp</groupId>
<artifactId>dependency-check-maven</artifactId>
<version>6.5.0</version>
<configuration>
<failBuildOnCVSS>6</failBuildOnCVSS>
<!-- other configurations -->
</configuration>
</plugin>
```
By setting failBuildOnCVSS to a value higher than 10 (the maximum CVSS score), the build will not fail due to vulnerabilities.


### API Key is used and a 403 or 404 error occurs
If an API Key is used and you receive a 404 error:

ERROR
io.github.jeremylong.openvulnerability.client.nvd.NvdApiException: NVD Returned Status Code: 404
There is a good chance that the API Key is set incorrectly or is invalid. To check if the API Key works the following curl command should return JSON:

curl -H "Accept: application/json" -H "apiKey: ########-####-####-####-############" -v https://services.nvd.nist.gov/rest/json/cves/2.0\?cpeName\=cpe:2.3:o:microsoft:windows_10:1607:\*:\*:\*:\*:\*:\*:\*
If no JSON is returned and you see a 404 error the API Key is invalid and you should request a new one.

Command to test the API Key:
```sh
curl -H "Accept: application/json" -H "apiKey: e236b561-390e-4f58-a24b-a8ac1b10f5bf" -v https://services.nvd.nist.gov/rest/json/cves/2.0\?cpeName\=cpe:2.3:o:microsoft:windows_10:1607:\*:\*:\*:\*:\*:\*:\*
```

### Running the project
```sh
# Run the project using Docker
cd docker
docker compose up -d
# Check the logs
docker compose logs

# Run the project using the executable JAR
java -jar target/authzforce-ce-restful-pdp-cxf-spring-boot-server-16.0.0.jar
```

## Support
If you are experiencing any issue with this project except for vulnerabilities mentioned previously, please report it on the [GitHub Issue Tracker](https://github.com/authzforce/restful-pdp/issues).
Please include as much information as possible; the more we know, the better the chance of a quicker resolution:

* Software version
* Platform (OS and JDK)
* Stack traces generally really help! If in doubt include the whole thing; often exceptions get wrapped in other exceptions and the exception right near the bottom explains the actual error, not the first few lines at the top. It's very easy for us to skim-read past unnecessary parts of a stack trace.
* Log output can be useful too; sometimes enabling DEBUG logging can help;
* Your code & configuration files are often useful.

If you wish to contact the developers for other reasons, use [AuthzForce contact mailing list](http://scr.im/azteam).

## Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md).
