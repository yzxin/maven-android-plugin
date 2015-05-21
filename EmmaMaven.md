# Introduction #

Emma is a tool which measures how many lines of code have been caused during testing.
Better explanation: http://emma.sourceforge.net/

## Important ##

Emma take your compiled java classes and put Emma bytecode.
Why ?
Emma is divided into blocks your code and to each block of code add emma counting methods.
**It is important to NOT submit your code with emma methods to google market.**

_I submitted my code with emma bytecode what are the consequences ?_
Almost none:
**build takes more storage space.**



# Maven and Emma #
Current information are not committed to the repository. **Configuration will start works from 3.0.X version**

## Configuration ##
### The project that we want to measure ###
Add to pom.xml additional emma profile

```
  <build/>

<!-- un commment when you want use sonar to present rules 
<properties>
    <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
    <sonar.emma.reportPath>target/emma/</sonar.emma.reportPath>
    <sonar.surefire.reportsPath >../regalandroid-test/target/surefire-reports</sonar.surefire.reportsPath>
    <sonar.core.codeCoveragePlugin>emma</sonar.core.codeCoveragePlugin>
</properties>
--> 

<profiles>
    <profile>
            <id>emma</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                    <artifactId>android-maven-plugin</artifactId>
                    <configuration>
                      <!-- emma start -->
                        <emma>
                             <enable>true</enable>
                             <classFolders>${project.basedir}/target/classes/</classFolders>
                        <outputMetaFile>${project.basedir}/target/emma/coverage.em</outputMetaFile>
                        </emma>
                        <!-- emma stop -->
                        <dex>
                             <noLocals>true</noLocals> <!--  must be set for emma -->
                        </dex>
                    </configuration>
                    <extensions>true</extensions>
                   </plugin>
                 </plugins>
              </build>
    </profile>
</profiles>

```

emma.enable = true  = run emma instrumentation, modify compiled class by putting emma code to classes.

emma.classFolders = folder where are compiled classes.
emma.outputMetaFile = coverage.em - emma meta data file ( need for generating rapport - because of emma4it plugin the best place folder for this file is target/emma/ folder.

Important:
dex.noLocals - this need to be set for emma.

To run emma instrumentation:
mvn clean install -Pemma
from parent project.
This command will compile code with emma instrumentation.

### The project where tests are ###
add to pom.xml

```
   <build/>
    <profiles>
        <profile>
       <id>emma</id>
            <dependencies>

               <dependency>
                    <groupId>emma</groupId>
                    <artifactId>emma</artifactId>
                    <type>jar</type>
                    <scope>compile</scope>
                    <version>2.1.5320</version>
               </dependency> 
             </dependencies>
             
            <build>
            <plugins>
               <plugin>
                  <groupId>org.codehaus.mojo</groupId>
                  <artifactId>properties-maven-plugin</artifactId>
                  <version>1.0-alpha-1</version>
                  <executions>
                     <execution>
                        <phase>initialize</phase>
                        <goals>
                           <goal>read-project-properties</goal>
                        </goals>
                        <configuration>
                           <files>
                              <!-- read project properties ( can be build.properties or default.properties 
Most important property is tested.project.dir - should be path to project which we want measure coverage
--> 
                              <file>project.properties</file>
                           </files>
                        </configuration>
                     </execution>
                  </executions>
               </plugin>

               <plugin>
                  <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                  <artifactId>android-maven-plugin</artifactId>
                  <configuration>
                     <test>
                        <!-- Run test with flag "-w coverage true" this is need for generate coverage.ec file, result file--> 
                        <coverage>true</coverage>
                        <createReport>true</createReport>
                     </test>
                  </configuration>
                  <extensions>true</extensions>
                  <!-- need for pull coverage.ec file and move to tested project-->
                  <executions>
                     <execution>
                        <id>pull-coverage</id>
                        <phase>post-integration-test</phase>
                        <goals>
                           <goal>pull</goal>
                        </goals>
                        <configuration>
                           <pullSource>/data/data/com.example.android.apis/files/coverage.ec</pullSource>
                           <pullDestination>${tested.project.dir}/target/emma/coverage.ec</pullDestination>
                        </configuration>
                     </execution>
                  </executions>
                  
               </plugin>
            </plugins>
         </build>
         <reporting>
            <plugins>
               <plugin>
                  <!-- Plugin for generate report - if you want use sonar you could skip this raport plugin --> 
                  <groupId>org.sonatype.maven.plugin</groupId>
                  <artifactId>emma4it-maven-plugin</artifactId>
                  <version>1.3</version>
                  <configuration>
                     <metadatas>${tested.project.dir}/target/emma/coverage.em,${tested.project.dir}/src/</metadatas>
                     <instrumentations>${tested.project.dir}/target/emma/coverage.ec</instrumentations>
                     <reportDirectory>${tested.project.dir}/target/emma/</reportDirectory>
                     <baseDirectory>${tested.project.dir}/target/</baseDirectory>
                     <formats>xml,html</formats>
                  </configuration>
               </plugin>
            </plugins>
         </reporting>
      </profile>
   </profiles>   

```

To run emma instrumentation and measure test coverage:
```
mvn clean install -Pemma  
```
from parent project.
This command will compile code with emma instrumentation and run integration tests with flag "-w coverage true"
and generate coverage.ec file
and pull coverage.ec file to ${tested.project.dir}/target/emma/

when all tests are successful finished we should have in folder:
> ${tested.project.dir}/target/emma/
coverage.em file ( around more then 1MB)
coverage.ec file ( more then 37B ) 37B = empty file

When we have this two files we can generate report (sonar or report plugin)

## How to present emma coverage result ##
We have more then one:
### Sonar (_thanks for Anthony Dahanne (anthonydahanne) for help configure sonnar_) ###
We cam enable emma plugin in sonar by adding properties to our project:
```

<properties>
    <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
    <sonar.emma.reportPath>target/emma/</sonar.emma.reportPath>
    <sonar.surefire.reportsPath >../<<TestProjectFolder>>/target/surefire-reports</sonar.surefire.reportsPath>
    <sonar.core.codeCoveragePlugin>emma</sonar.core.codeCoveragePlugin>
</properties>

```
This properties are need to force use coverage.ec and coverage.em files and not run sonar re instrument code by emma plugin in Sonar.

### Report plugin ###
```
mvn org.sonatype.maven.plugin:emma4it-maven-plugin:1.3:report 
```
from parent project.
Report will be generated:
> ${tested.project.dir}/target/site/emma/index.html

Because plugin emma4it has some bugs ( or I can't configure emma4it correctly) only base report will be generated.
To generate full report ( each line will be marked if is covered or not ) please use shell:
```
java -cp ${emmaDirectory}/emma.jar emma report -r html -in coverage.em,coverage.ec -sp ${project.source.folder} 
```

# Emma from shell #
theoretically you can run emma from shell:

(C1):
```
java -cp ${emmaDirectory}/emma.jar emma instr -m overwrite -cp ${jar_file/folder_with_compiled_sources}
```
example:
(C2):
```
java -cp /androidsources/external/emma/lib/emma.jar emma instr -m overwrite -cp project-1.0-SNAPSHOT.jar
```

This will generate **coverage.em** file - metadata file with information about your code and this command mixing emma bytecode with your bytecode.

Next you must compile generated bytecode by dex, then package to apk.
Install apk to emulator/device.
Then run test with coverage flag.Example:
(C3):
```
adb shell am instrument *-e coverage true* -w com.arnav.test/pl.polidea.instrumentation.PolideaInstrumentationTestRunne
```

Command C3 - will generate coverage.ec file on device in /data/data/${project}/files/coverage.ec

coverage.ec file has information about which block was called during tests.

When we have this two files: coverage.ec, coverage.em we can generate nice report:
(C4):
```
java -cp ${emmaDirectory}/emma.jar emma report -r html -in coverage.em,coverage.ec -sp ${project.source.folder}
```
In command C4, option -sp is optional - when you use this option, report will be more detailed - because each line of code will be marked.

## Emma compile process ##

When we use maven-android-plugin without emma profile:
```
mvn clean compile install
```
Then:
  1. maven clean target folder and temporary folder
  1. maven use javac to compile sources -> to target/classes/ folder
  1. maven use dex command to convert java byte code to dex format -> targer/classes.dex
  1. classees.dex and resources are package to targer/.apk
  1. (1-4) for integration test.
  1. apk file are install to device
  1. integration test are started.

When we use maven-android-plugin with emma profile:
```
mvn clean compile install -Pemma
```
Then:
  1. maven clean target folder and temporary folder
  1. maven use javac to compile sources -> to target/classes/ folder
  1. **emma.jar is run - and we inject emma code to java byte code. This command generate coverage.em file - meta-data file which collect name of all class and function in our project**
  1. maven use dex command to convert java byte code to dex format -> targer/classes.dex
  1. classees.dex and resources are package to targer/.apk
  1. (1-5) for integration test.
  1. apk file are install to device
  1. integration test are started with flag **coverage true** and after this command on device in folder **/data/data/${project}/files/coverage.ec** is generated