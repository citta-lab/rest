RESTFul Service [ juggling pebbles ]
--------------------------

1. Packaging ( sun vs galssfish )
----
Jersey packaging has been changed from version 2.0 to glassfish from sun. So prior to version 2.0 jersey packaging has been found in `org.sun.jersey` and since 2.0 the packaging has been done in `org.glassfish.jersey`. Content suppose to be same however i have faced problem handling CORSFilter between glassfish and sun.
Example: [CORSFilter](https://github.com/citta-lab/rest/tree/master/CORSFilter)

2. Maven
-----
Maven build command will generate the skeleton of the jersey starter app which we can leverage to develop the web service.
```javascript
mvn archetype:generate -DarchetypeGroupId=org.glassfish.jersey.archetypes -DarchetypeArtifactId=jersey-quickstart-webapp -DarchetypeVersion=2.25.1
```
* web.xml: defines the bootstrap for jersey servlet container during server load, and it looks for `param-value`’s value and the defined path in `url-pattern`
* Basic Building Blocks: @Path ( path to the service ), @GET ( the request ) and @Produces ( response type )
* JSON requires: `jersey-media-moxy` needs to be added to `pom.xml` ( so JAXB will convert pojo to xml, and then xml to JSON). if we don’t have this we will face MediaType error.
* Finding and grabbing specific parameter from url: @Path(“{activityId}”) and in method we can pass this activityId as parameter by mentioning function_name( `@PathParam (“activityId”) String activityId`)
* Gathering Form: Gathering form params can be done by `MultivaluedMap<String, String>` formParam Extracting form data => `formParam.getFirst(“description”)`
* GenericType: To handle list of particular class type we can use GenericType object wrapper provided by jersey to let the code understand the class type.

3. Eclipse ( Maven Trouble )
----
* Maven directory in mac: `~/.m2`
* `No Class Exception`: Missing dependency in pom.ml and m2 folder.
* Build maven:  `mvn package -P dev`
* Eclipse pom.xml error: right click on project > Maven > Update Project Configuration
  [StackOverflow](https://stackoverflow.com/questions/6111408/maven2-missing-artifact-but-jars-are-in-place )
* Maven Error while running maven clean on eclipse:
  Description: This typically happens when maven project is ran on the eclipse whenever pom.xml doesn’t read the defined jar and add it to the dependencies.
  Error: `Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/maven/cli/MavenCli`
  Fix: Right click on project > Build Path > Configure Build Path > double Click on JRE System Library ( it will be 1.6 or less ) > workspace default > apply.
