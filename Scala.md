# Introduction #

Scala is a very nice alternative to Java for developing Android Applications. The most important  thing to remember: You must use ProGuard. Here the basic configuration you need to add to your master project:

```
…
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>2.9.0-1</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
…
  <repositories>
    <repository>
      <id>scala-tools.org</id>
      <name>Scala-tools Maven2 Repository</name>
      <url>http://www.scala-tools.org/repo-releases</url>
    </repository>
  </repositories>
  <pluginRepositories>
    <pluginRepository>
      <id>scala-tools.org</id>
      <name>Scala-tools Maven2 Repository</name>
      <url>http://www.scala-tools.org/repo-releases</url>
    </pluginRepository>
  </pluginRepositories>
…
  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>com.tiobe.jacobe</groupId>
          <artifactId>maven-jacobe-plugin</artifactId>
          <version>1.0</version>
        </plugin>
        <plugin>
          <groupId>org.scala-tools</groupId>
          <artifactId>maven-scala-plugin</artifactId>
          <version>2.15.2</version>
          <configuration>
            <scalaVersion>${scala.version}</scalaVersion>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
…
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <version>2.15.2</version>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
        </configuration>
      </plugin>
    </plugins>
  </reporting>
…
  <properties>
    <scala.version>2.9.0</scala.version>
  </properties>
…
```

# Using Scala for the Application #

```
…
  <dependencies>
    <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-library</artifactId>
      <type>jar</type>
    </dependency>
  </dependencies>
…
  <build>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <executions>
          <execution>
            <id>scala-compile-first</id>
            <phase>process-resources</phase>
            <goals>
              <goal>add-source</goal>
              <goal>compile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>add-source</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>add-source</goal>
            </goals>
            <configuration>
              <sources>
                <source>src/main/scala</source>
              </sources>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
…
```

# Using Scala for the Instrumentation Test #

```
…
  <dependencies>
    <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-library</artifactId>
      <type>jar</type>
    </dependency>
  </dependencies>
…
  <build>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <executions>
          <execution>
            <id>scala-compile-first</id>
            <phase>process-resources</phase>
            <goals>
              <goal>add-source</goal>
              <goal>compile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>add-source</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>add-source</goal>
            </goals>
            <configuration>
              <sources>
                <source>src/main/scala</source>
              </sources>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  <build>
…
```

# Using Scala for the Both #

Now here things get a little painful because the Scala library must only be included once in the Applications. In the instrumentation test you must set the Scala library to `provided`.

And this leaves us with a little problem: Any Scala class or method only used in test but not in the application will be removed by proguard. The only way to prevent this is to painfully add them to the proguard.cfg.

**Hint:** Use a 2nd proguard.cfg for release / market compiles to keep you apk size down. Which has the nice side effect that you can use proguard.cfg to strip `android.util.Log` calls as well.