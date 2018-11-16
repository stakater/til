# Maven how to use multiple repositories?

- Here is a working example of how to use multiple repositories to download maven dependencies for `Maven 3.5`
- It's impossible to specify a dedicated repository to look up an artifact. Maven will look up all configured repositories one by one until the artifact is found. Just add the central mirror and internal repositories to the `settings.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd
http://maven.apache.org/SETTINGS/1.1.0 "
          xmlns="http://maven.apache.org/SETTINGS/1.1.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <mirrors>

        <!-- 
        When you use the advanced syntax and configure multiple mirrors, keep in mind that their declaration order matters. When Maven looks for a mirror of 
        some repository, it first checks for a mirror whose <mirrorOf> exactly matches the repository identifier. If no direct match is found, Maven picks the 
        first mirror declaration that matches according to the rules above (if any). Hence, you may influence match order by changing the order of the 
        definitions in the settings.xml

        Examples:

        * = everything
        external:* = everything not on the localhost and not file based.
        repo,repo1 = repo or repo1
        *,!repo1 = everything except repo1        
        -->
        <mirror>
            <id>Nexus1</id>
            <name>Nexus based maven repository manager for Company X.</name>
            <!-- The base URL of this mirror. The build system will use this URL to connect to a repository rather than the original repository URL. -->
            <url>https://nexus.tools.nexus1.com/repository/public/</url>
            <!--  everything not on the localhost and not file based. -->
            <!-- The id of the repository that this is a mirror of. For example, to point to a mirror of the Maven central repository 
            (https://repo.maven.apache.org/maven2/), set this element to central. More advanced mappings like repo1,repo2 or *,!inhouse are also possible. 
            This must not match the mirror id. -->
            <!-- mirrorOf>external:*</mirrorOf -->
            <mirrorOf>stakater</mirrorOf>
        </mirror>

        <mirror>
            <id>Nexus2</id>
            <name>Nexus based maven repository manager for Company Y.</name>
            <!-- The base URL of this mirror. The build system will use this URL to connect to a repository rather than the original repository URL. -->
            <url>https://nexus.tools.nexus2.com/repository/public/</url>
            <!--  everything not on the localhost and not file based. -->
            <!-- The id of the repository that this is a mirror of. For example, to point to a mirror of the Maven central repository 
            (https://repo.maven.apache.org/maven2/), set this element to central. More advanced mappings like repo1,repo2 or *,!inhouse are also possible. 
            This must not match the mirror id. -->
            <!-- mirrorOf>external:*</mirrorOf -->
            <mirrorOf>dealer</mirrorOf>
        </mirror>

       <mirror>
            <id>MavenCentral</id>       
            <url>https://repo.maven.apache.org/maven2</url>
            <mirrorOf>central</mirrorOf>
       </mirror>

    </mirrors>

    <servers>
        <!-- This can be ignored if your repositories are public (although thats a bad idea) -->
        
        <!-- The repositories for download and deployment are defined by the repositories and distributionManagement elements of the POM. However, certain 
        settings such as username and password should not be distributed along with the pom.xml. This type of information should exist on the build server in 
        the settings.xml. -->

        <server>
            <!-- This is the ID of the server (not of the user to login as) that matches the id element of the repository/mirror that Maven tries to connect to. -->
            <id>Nexus2</id>
            <!-- These elements appear as a pair denoting the login and password required to authenticate to this server. -->
            <username>un-changeme</username>
            <password>pwd-changeme</password>
        </server>

        <server>
            <!-- This is the ID of the server (not of the user to login as) that matches the id element of the repository/mirror that Maven tries to connect to. -->
            <id>Nexus1</id>
            <!-- These elements appear as a pair denoting the login and password required to authenticate to this server. -->
            <username>un-changeme</username>
            <password>pwd-changeme</password>
        </server>

    </servers>

    <profiles>
        <profile>
            <id>nexus</id>
            <repositories>
                <repository>
                    <id>Nexus1</id>
                    <name>company1 repository</name>
                    <url>https://nexus.tools.nexus1.com/repository/public/</url>
                </repository>

                <repository>
                    <id>Nexus2</id>
                    <name>company2 repository</name>
                    <url>https://nexus.tools.nexus2.com/repository/public/</url>
                </repository>            

                <repository>
                    <id>MavenCentral</id>
                    <url>https://repo.maven.apache.org/maven2/</url>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <!--  If you specify repositories in profiles you must remember to activate that
        particular profile! As you can see above we do this by registering a profile
        to be active in the <<<activeProfiles>>> element. -->
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>

</settings>
```

## Maven Dependency Search Sequence

When we execute Maven build commands, Maven starts looking for dependency libraries in the following sequence

- Step 1: Search dependency in local repository, if not found, move to step 2 else perform the further processing.
- Step 2: Search dependency in central repository, if not found and remote repository/repositories is/are mentioned then move to step 4. Else it is downloaded to local repository for future reference.
- Step 3: If a remote repository has not been mentioned, Maven simply stops the processing and throws error (Unable to find dependency).
- Step 4: Search dependency in remote repository or repositories, if found then it is downloaded to local repository for future reference. Otherwise, Maven stops processing and throws error (Unable to find dependency).

## Types of Maven Repositories

Maven repository are of three types. 

- local: Maven local repository is a folder location on your machine. It gets created when you run any maven command for the first time.
- central: Maven central repository is repository provided by Maven community. It contains a large number of commonly used libraries.
- remote: Sometimes, Maven does not find a mentioned dependency in central repository as well. It then stops the build process and output error message to console. To prevent such situation, Maven provides concept of Remote Repository, which is developer's own custom repository containing required libraries or other project jars.

## Keywords

- Maven settings for multiple repositories
- `settings.xml` for multiple maven repositories
