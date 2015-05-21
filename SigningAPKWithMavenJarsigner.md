# Introduction #

This article describes how to use the Maven jarsigner plugin to sign an android application with a proper key store, rather then the debug one. This is the recommended way to sign your Android application when using Maven to build it.

A fully working application of signing, zipaligning and proguarding is available in the MorseFlash application that is part of the Android Maven Samples project.

# Details #

To enable signing you need to provide a key store path (in the keystore node below), a key alias (in the alias node below), and passwords for both the keystore and the key (in the storepass and keypass nodes respectively).

Include the following in the pom file. If you use a hierarchal project structure then the below should be located in the parent pom in order to sign all child poms with the same key.

```
<profiles>
        <profile>
            <id>sign</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-jarsigner-plugin</artifactId>
                        <version>1.2</version>
                        <executions>
                            <execution>
                                <id>signing</id>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                                <phase>package</phase>
                                <inherited>true</inherited>
                                <configuration>
                                    <archiveDirectory></archiveDirectory>
                                    <includes>
                                        <include>target/*.apk</include>
                                    </includes>
                                    <keystore>path/to/keystore</keystore>
                                    <storepass>storepasword</storepass>
                                    <keypass>keypassword</keypass>
                                    <alias>key-alias</alias>
                                    <arguments>
                                      <argument>-sigalg</argument><argument>MD5withRSA</argument>
                                      <argument>-digestalg</argument><argument>SHA1</argument>
                                    </arguments>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                        <artifactId>android-maven-plugin</artifactId>
                        <inherited>true</inherited>
                        <configuration>
                            <sign>
                                <debug>false</debug>
                            </sign>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
```

To trigger the profile use the following command:
```
mvn install -Psign
```

The configuration will bind the jarsigner plugin to the package phase of the execution when enabled with the -Psign switch in the build. Rather then storing the passwords in clear text in your pom you can pass them to maven as arguments when you execute the command. Go to the <a href='http://maven.apache.org/plugins/maven-jarsigner-plugin/index.html'>jarsigner plugins documentation page</a> for information on how it works and how it can be configured.