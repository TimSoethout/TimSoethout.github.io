---
layout: post
category:
tagline:
tags: [scala, java, programming, qa, quality, sonar, maven, multi-module]
title: Mixing Java & Scala with Sonar with correct code coverage
---
{% include JB/setup %}

Recently we added Scala to a Java Maven project. This works perfectly fine, until we looked at the Sonar report. It turns out that having nice automated code checks for a combined Java/Scala project is quite hard.
Last week it was solved. This post is to write down the lessons learned, for it might help others.

Using a single language is fine:

- Java + Sonar works perfectly fine, using FindBugs, PMD, any code coverage framework you would like to use (Cobertura, Jacoco etc).

- Scala + Sonar also works fine, using the [sonar-scala plugin](https://github.com/1and1/sonar-scala), ScalaStyle and [Scoverage](https://github.com/RadoBuransky/sonar-scoverage-plugin) plugin.

The combination of combined source code makes it hard.

# Code coverage methods for Java and Scala

Cobertura uses instrumentation on the bytecode to scan the coverage. This also works on Scala code, but since the Scala compiler generated a lot more bytecode for case classes, traits etc. The coverage will be 20% lower than it really is.

Another option is Jacoco, which uses an agent by default to instrument the code when it is loaded by the Class loader. This seems to work fine but does not take the Scala code in account properly.

For dedicated Scala coverage Scoverage is a good candidate. Scoverage does statement coverage, which means that it has a more fine-grained knowledge, since it measures coverage per statement and not per line as the most Java coverage tools (such as Cobertura and Jacoco) do. This is more ideal for Scala, since one often writes more expressive code, which means more information and semantics are embedded in a single line of code.
Scoverage only scans Scala source files.

Jacoco turned out not to be an option for us because it interprets the bytecode too strict. In certain cases the Java compiler generates more byte code than you normally cover. For example an expression such as `if(!someBool())` results in 4 branches being generated, possibly by first evaluating `someBool()` and assigning it to a variable and than doing the actual check. Not all those branches are correctly covered by unit tests, which results in lower coverage. On the github issue tracker it is promised in [a two year old issue](https://github.com/jacoco/jacoco/issues/15) these kind of issues will be solved by filters for the report generation, but it is not available yet.

For us this was not acceptable since we want to automatically check the coverage levels, and if the level is tens of percentages lower depending on which kind of statements we use, it would not help us very much.

# The Solution


The approach became: Cobertura for Java source files, Scoverage for Scala source files.
Also we needed Sonar version of 4.5 or higher (we picked 5.1), because it supports mixed projects with multiple languages.

This resulted us to two challenges:

1. Both the Java CoberturaSensor and Scala CoberturaSensor kicked in when `mvn sonar:sonar` was run, which resulted in duplicate coverage information for the same file, failing the Sonar run.
2. Both Cobertura's coverage file and Scoverage's coverage file contain information for the Scala source files.

## 1. Multiple CoberturaSensor's

After a lot of debugging I found that it was not possible to fix this. Both CoberturaSensors use the same `reportPath` property to find the coverage file. I wanted to disable the Scala Cobertura scanner since I did not want to use Cobertura for the Scala sources. In the end I had to change the Scoverage Sonar plugin code to make this happen by introducing a different property `sonar.scala.cobertura.reportPath` which is used if specified. I did this to point the `ScalaCoberturaSensor` to a non-existing file so it was skipped and only the Java `CoberturaSensor` would actually be run and submit to Sonar.
See my [fork](https://github.com/TimSoethout/sonar-scala) and the [pull request](https://github.com/1and1/sonar-scala/pull/1) for the change.

## 2. Duplicate coverage for Scala source files

Now the problem became coverage information being inserted twice by the now only (Java) Cobertura run, which still contained the Scala source coverage information and also by the Scoverage report.

Since I only wanted the qualitatively higher coverage of Scoverage I decided to delete the scala coverage information from Cobertura's `coverage.xml`. Fortunately this turned out to be quite easy by removing everything which is matched by this xpath: `//class[contains(@filename,'.scala')]`.

From what it seems Sonar interprets the coverage information itself again, so the calculated and now incorrect averages and totals which remain in the filtered coverage xml will be ignored by Sonar.

The next step was removing this as part of our automated build. I converted this command to a Ruby one-liner, but ran into issues the `maven-exec-plugin` trying to run this, which had to do with ruby dependencies which were somehow not available during the build.

Another approach was using `xml-maven-plugin` and an XSLT template to convert the cobertura XML. This was nice, but I wanted it also to work on a multi module maven project, where I could configure this in the parent pom.xml. Unfortunately the reference to the XSLT file was relative to the project being build, which would mean I had to configure this again for every module in the project or move the XSLT to a absolute location. Both are not acceptable.

I decided to create a maven plugin which could do this for me. This way I know for sure it will work correctly for multimodule maven projects and also on all platforms including the build server.

I release the (minimal viable) plugin on maven central, which was a nice experience in itself.
The source can be found [here](https://github.com/TimSoethout/transform-xml-maven-plugin) and it can be include in your project like this:

{% highlight xml %}
<build>
    <plugins>
        <plugin>
            <groupId>nl.timmybankers.maven</groupId>
            <artifactId>transform-xml-maven-plugin</artifactId>
            <version>1.0.0</version>
            <executions>
                <execution>
                    <phase>prepare-package</phase>
                </execution>
            </executions>
            <goals>
                <goal>transform-xml</goal>
            </goals>
            <configuration>
                <inputXmlPath>${project.build.directory}/site/cobertura/coverage.xml</inputXmlPath>
                <outputXmlPath>${sonar.build.directory}/${sonar.cobertura.reportPath}</outputXmlPath>
                <xpath>//class[contains(@filename,'.scala')]</xpath>
                <action>DELETE</action>
                <skipOnFileErrors>true</skipOnFileErrors>
            </configuration>
        </plugin>
    </plugins>
</build>
{% endhighlight %}

For now it only support the `DELETE` action for the usecase as described above.
Now I can run my build including publish to Sonar using this one liner:
`mvn clean cobertura:cobertura scoverage:report prepare-package sonar:sonar`
Make sure that `cobertura.report.format` is set to `xml` which will result in the coverage information being available in `target/site/cobertura/coverage.xml`.

My sonar properties in maven:

{% highlight xml %}
<sonar-maven-plugin.version>2.6</sonar-maven-plugin.version>
<sonar.jdbc.driver>org.postgresql.Driver</sonar.jdbc.driver>
<sonar.jdbc.url>jdbc:...</sonar.jdbc.url>
<sonar.host.url>http://...:9000/</sonar.host.url>
<sonar.core.codeCoveragePlugin>scoverage</sonar.core.codeCoveragePlugin>
<sonar.java.coveragePlugin>cobertura</sonar.java.coveragePlugin>
<sonar.junit.reportsPath>target/surefire-reports</sonar.junit.reportsPath>
<sonar.scoverage.reportPath>target/scoverage.xml</sonar.scoverage.reportPath>
<sonar.cobertura.reportPath>target/cobertura-without-scala.xml</sonar.cobertura.reportPath>
<sonar.scala.cobertura.reportPath>/target/nonexisting.xml</sonar.scala.cobertura.reportPath>
<sonar.sources>src</sonar.sources>
<sonar.exclusions>src/test/**</sonar.exclusions>
<sonar.sourceEncoding>UTF-8</sonar.sourceEncoding>
{% endhighlight %}

# Conclusion

It was quite an effort to get this working, but in the end I am happy with the result. I hope others will also benefit from this.

Long story short: Cobertura gives the best coverage information for the Java code, Scoverage for the Scala code. To make sure no duplicate coverage information is send to Sonar, I changed the sonar-scoverage-plugin to ignore the report for the `ScalaCoberturaSensor` and removed the coverage information for the Scala source from Cobertura's coverage xml using a maven plugin. Using this method I can run the Sonar scan directly using maven for any mixed Java/Scala project.
