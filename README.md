# [beanshell-examples](https://chabala.github.io/beanshell-examples/)

BeanShell example code, in the context of enhancing a Maven build.

<div style="text-align: right">
<a href="https://github.com/chabala/beanshell-examples/">GitHub repository for this page</a> / 
<a href="https://chabala.github.io/">Up to the root page</a>
</div>

## What is BeanShell?

[BeanShell][1] is one of several [JSR-223][2] compliant scripting languages. JSR-223 compliance is a standard
for scripting languages to interoperate on and with the Java platform. The main JSR-223 language is Javascript;
Oracle included a Javascript engine in the JRE, while other language engines must be added explicitly to be
used. BeanShell as a language is essentially plain Java, with some extra syntax added to make scripting easier.

[1]: http://www.beanshell.org/
[2]: https://docs.oracle.com/javase/8/docs/technotes/guides/scripting/prog_guide/api.html

## Why would one use BeanShell?

BeanShell can be used for all the use cases of any JSR-223 language, e.g. allowing more dynamic interaction in
a program by interpreting a script, such as adding flexibility by allowing configuration-via-script or
user-supplied scripts, delegating functions to scripts in other languages, etc.

I'm collecting examples here of a specific use case: using BeanShell as an _escape hatch_ in Maven builds for
custom logic. The advantage of using BeanShell over JavaScript in that case is that one already has access
to some useful Java helper methods from Maven, and can easily add library dependencies to the script runner
if needed, while JavaScript would be a different ecosystem entirely.

## Why use BeanShell in Maven?

[Maven][3] is a declarative build tool for Java. There is an XML document, [the POM][4], which defines Java
plugins that do the work of the build. Most of the time, all the tasks one could want in a build are covered
by the first-party plugins of Maven, or a popular third-party plugin. But if one has a strong desire to do
something outside of those plugins, there are various _escape hatches_ before writing a custom Maven plugin is
required.

[3]: https://maven.apache.org/
[4]: https://maven.apache.org/pom.html

The first is the [`maven-antrun-plugin`][5]. This will call any [`Ant`][6] tasks defined for it. For a simple
task like copying/moving/deleting a file, this is a good choice. In theory, one can do anything that would be
possible in Ant, but you'll also be using Ant, and a complex task will become unpleasant quickly.

[5]: https://maven.apache.org/plugins/maven-antrun-plugin/
[6]: https://ant.apache.org/

The other alternative to writing a custom plugin, is using a script running plugin, and here I'm going to focus
on two that allow running BeanShell:

### beanshell-maven-plugin

[`beanshell-maven-plugin`](https://genthaler.github.io/beanshell-maven-plugin/) is my first choice, because
the name of the plugin makes it clear it's for running BeanShell, and that's all it does.

### script-maven-plugin

[`script-maven-plugin`](https://github.com/alexec/script-maven-plugin) is another option for running BeanShell,
along with other JSR-223 languages. It is more flexible in that way, and since BeanShell is the default
language, configuration is about the same as `beanshell-maven-plugin`.

Both have useful documentation, and are worth poking around if you want to try and figure out who is still using
BeanShell, for what purposes, or what other languages exist for JSR-233 scripting. For instance, [Appendix A][7]
on the `script-maven-plugin` README has a nice list of other JSR-223 script languages and their engines, and
the underlying [Apache BSF library][8] appears to [support even more][9]. Neither plugin has seen any significant
development in 7+ years, so they must be stable and feature complete, right?

[7]: https://github.com/alexec/script-maven-plugin/blob/7dac99b2b8529a067ec7d7dd495c86f7ecc541bd/README.md#appendix-a---languages-table
[8]: https://commons.apache.org/proper/commons-bsf/
[9]: https://github.com/apache/commons-bsf/blob/master/src/main/resources/org/apache/bsf/Languages.properties

BeanShell has [documentation on its own website](http://beanshell.org/manual/bshmanual.html) as well, but it is
from the viewpoint of a general scripting language with no special information about using it in a Maven
context; only Ant is mentioned as a build tool integration. Unfortunately for BeanShell, for general
Java-as-a-scripting-language usage, [JBang](https://www.jbang.dev/) would always be a better choice, unless
you were stuck in a pre-Java-8 environment.

## Documentation examples

First let's look at the examples from the plugin documentation.

From: [https://genthaler.github.io/beanshell-maven-plugin/usage.html](
https://genthaler.github.io/beanshell-maven-plugin/usage.html)

```xml
<plugin>
    <groupId>com.github.genthaler</groupId>
    <artifactId>beanshell-maven-plugin</artifactId>
    <version>1.4</version>
    <executions>
        <execution>
            <phase>test</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <script><![CDATA[
              import org.codehaus.plexus.util.FileUtils;
              FileUtils.fileWrite( "touched.txt", "This is a Beanshell Maven Plugin POM test" );
          ]]></script>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Nothing too exciting here, just writing a file. The most interesting detail here is that
`org.codehaus.plexus.util.FileUtils` is available for import without declaring any dependencies, which implies
that all of [Plexus-Utils](https://codehaus-plexus.github.io/plexus-utils/) is in scope, likely due to being core
to Maven. As we shall see, plain BeanShell is very limiting, so having a utility library available is useful.

We are further invited to peruse the integration tests for further usage ideas, but they don't do anything more
interesting than writing a file as above.

___

`script-maven-plugin` is following the 'README is my only documentation' pattern, [which contains this][10]:

[10]: https://github.com/alexec/script-maven-plugin/blob/7dac99b2b8529a067ec7d7dd495c86f7ecc541bd/README.md#example-1---beanshell

```xml
<plugin>
    <groupId>com.alexecollins.maven.plugin</groupId>
    <artifactId>script-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals><goal>execute</goal></goals>
            <configuration>
                <!-- beanshell is the default language -->
                <script>
                    System.out.println(project.getName());
                </script>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.apache-extras.beanshell</groupId>
            <artifactId>bsh</artifactId>
            <version>2.0b6</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</plugin>
```

Actually, I had to merge a couple fragments to get a coherent example, but nothing too exciting here either. The
takeaway from this example is the `project.getName()`: Maven properties are in scope and can be referenced by your
scripts.

## Real world examples

Luckily, there's a large corpus of real code that can be searched to see how people are using these plugins to run
BeanShell, GitHub. I searched specifically for pom.xml files that contained 'beanshell', and found these.

### JaCoCo

From: [https://github.com/jacoco/jacoco/blob/25594b2c01d5b76a4f76cf2de967aa70258c84c0/org.jacoco.build/pom.xml#L671-L717](
https://github.com/jacoco/jacoco/blob/25594b2c01d5b76a4f76cf2de967aa70258c84c0/org.jacoco.build/pom.xml#L671-L717)

```xml
<plugin>
    <groupId>com.github.genthaler</groupId>
    <artifactId>beanshell-maven-plugin</artifactId>
    <executions>
      <execution>
        <id>parse-version</id>
        <phase>validate</phase>
        <goals>
          <goal>run</goal>
        </goals>
        <configuration>
          <quiet>true</quiet>
          <script><![CDATA[
            major = project.getProperties().get("parsedVersion.majorVersion");
            minor = project.getProperties().get("parsedVersion.minorVersion");
            incremental = project.getProperties().get("parsedVersion.incrementalVersion");
            unqualifiedVersion = major + "." + minor + "." + incremental;
            project.getProperties().setProperty("unqualifiedVersion", unqualifiedVersion);

            qualifier = "${maven.build.timestamp}";
            project.getProperties().setProperty("buildQualifier", qualifier);

            qualifiedVersion = unqualifiedVersion + "." + qualifier;
            project.getProperties().setProperty("qualified.bundle.version", qualifiedVersion);

            buildDate = qualifier.substring(0, 4) + "/" + qualifier.substring(4, 6) + "/" + qualifier.substring(6, 8);
            project.getProperties().setProperty("build.date", buildDate);

            commitId = project.getProperties().get("build.commitId");
            pkgName = commitId.substring(commitId.length() - 7, commitId.length());
            project.getProperties().setProperty("jacoco.runtime.package.name", "org.jacoco.agent.rt.internal_" + pkgName);

            void loadLicense(String libraryId) {
                version = project.getProperties().get(libraryId + ".version");
                path = project.getBasedir().toPath().resolve("../org.jacoco.build/licenses/" + libraryId + "-" + version + ".html");
                license = new String(java.nio.file.Files.readAllBytes(path), "UTF-8");
                project.getProperties().setProperty(libraryId + ".license", license);
            }
            loadLicense("args4j");
            loadLicense("asm");
            loadLicense("googlecodeprettify");
          ]]>
          </script>
        </configuration>
      </execution>
    </executions>
</plugin>
```

JaCoCo is licensed under [Eclipse Public License 2.0](https://www.eclipse.org/legal/epl-2.0/).

This example is making use of some features we've seen earlier, like referencing the Maven `project` object that is in
scope during the build. It's also referencing `maven.build.timestamp` with some `${}` syntax. Finally, it defines a
method in the script to load a file from a calculated path into a String, and add that as a build property.

In summary, this example is reading and writing properties, but in a more complex way than [properties-maven-plugin][11]
could handle, and presumably using them later on in the build.

[11]: https://www.mojohaus.org/properties-maven-plugin/

### MariaDB4j

From: [https://github.com/MariaDB4j/MariaDB4j/blob/a77298386c309276674f207f9dced010c73eb35c/mariaDB4j-pom-lite/pom.xml#L97-L153](
https://github.com/MariaDB4j/MariaDB4j/blob/a77298386c309276674f207f9dced010c73eb35c/mariaDB4j-pom-lite/pom.xml#L97-L153)

```xml
<plugin>
    <groupId>com.alexecollins.maven.plugin</groupId>
    <artifactId>script-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
       <execution>
         <phase>prepare-package</phase>
         <goals>
           <goal>execute</goal>
         </goals>
         <configuration>
           <script>
             // BeanShell is 2005-ish and thus doesn't support generics, varargs, try-with-resources or lambdas, so:
             // (If we do this kind of inline code in pom.xml more often, we shold have a new simple module in
             //  odl-parent, which has *.java that we compile, and then just depend on it here and call one-line
             //  static class methods only - it will be MUCH easier to write!)
             void copy(File root, String glob, File target) {
                 java.nio.file.DirectoryStream dirStream = java.nio.file.Files.newDirectoryStream(root.toPath(), glob);
                 Iterator dirStreamIterator = dirStream.iterator();
                 while (dirStreamIterator.hasNext()) {
                     java.nio.file.Path path = dirStreamIterator.next();
                     java.nio.file.Files.copy(path, new File(target, path.toFile().getName()).toPath(),
                         new java.nio.file.CopyOption[] {
                             java.nio.file.StandardCopyOption.REPLACE_EXISTING,
                             java.nio.file.StandardCopyOption.COPY_ATTRIBUTES
                         }
                     );
                 }
                 dirStream.close();
             }

             File gitRepoRootDir = project.basedir;
             while (!new File(gitRepoRootDir, ".git").exists() &amp;&amp; gitRepoRootDir.getParentFile() != null) {
                 gitRepoRootDir = gitRepoRootDir.getParentFile();
             }

             File target = new File(project.build.outputDirectory);
             target.mkdirs();
             copy(gitRepoRootDir, "README*", target);
             copy(gitRepoRootDir, "CONTRIBUTING*", target);
             copy(gitRepoRootDir, "CHANGES*", target);
             copy(gitRepoRootDir, "LEGAL*", target);
             copy(gitRepoRootDir, "CONTRIBUTORS*", target);
             copy(gitRepoRootDir, "LICENSE*", target);
             copy(gitRepoRootDir, "NOTICE*", target);
           </script>
         </configuration>
       </execution>
     </executions>
     <dependencies>
       <dependency>
         <groupId>org.apache-extras.beanshell</groupId>
         <artifactId>bsh</artifactId>
         <version>2.0b6</version>
       </dependency>
     </dependencies>
  </plugin>
```

MariaDB4j is licensed under [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0).

Firstly, the comment is great. Can confirm, varargs and try-with-resources do not work in BeanShell. Given the list
of language features BeanShell doesn't support, it might be slightly more useful to describe it as pre-Java-5 syntax.

There's also a hint that if the inline code gets too complex, it makes sense to compile it as part of an artifact
one can then depend upon. That gets around the lack of language feature support: BeanShell can call into
precompiled bytecode without issue. But if one is making an artifact of helper methods for a BeanShell script, it
might be time to start thinking about making a dedicated Maven plugin.

As far as this specific script, a method is defined for copying using some `java.nio.file` classes, and it also uses
an `Iterator`, presumably to work around a lack of enhanced for loop support, or to ensure the `DirectoryStream`
is closed in the absence of try-with-resources support. The Maven property `project.build.outputDirectory` is
referenced directly as the input to the `File` constructor, without any need of `${}` syntax.

Those two example comprise the entirety of what I discovered in real world usage on GitHub, though there were many
copies of them simply due to forks of the containing projects and slight variations therein.

### Personal examples

The next two examples are my own work, and you are free to use them as you see fit.

From: [https://github.com/chabala/brick-control-lab/pull/26/commits/a3d940914e08e1056ca07eee463406ec07683376](
https://github.com/chabala/brick-control-lab/pull/26/commits/a3d940914e08e1056ca07eee463406ec07683376)

```xml
<plugin>
    <groupId>com.github.genthaler</groupId>
    <artifactId>beanshell-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>add-redirect-pages</id>
            <phase>pre-site</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <quiet>true</quiet>
                <script><![CDATA[
        // Generate redirect pages for historic site URLs (MPIR-323)
        String contents(String target) {
            String url = project.url + target;
            String template = "<!DOCTYPE html PUBLIC \"-//W3C//DTD XHTML 1.0 Transitional//EN\" " +
                    "\"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd\">\n" +
                    "<html xmlns=\"http://www.w3.org/1999/xhtml\" xml:lang=\"en\" lang=\"en\">\n" +
                    "<head>\n" +
                    "    <meta http-equiv=\"Content-Type\" content=\"text/html; charset=UTF-8\" />\n" +
                    "    <link rel=\"canonical\" href=\"" + url + "\" />\n" +
                    "    <meta http-equiv=\"refresh\" content=\"0;url=" + target + "\" />\n" +
                    "</head>\n<body>\n<h1>\n" +
                    "    This page has been moved to <a href=\"" + target + "\">" + url + "</a>\n" +
                    "    <script type='text/javascript'> window.location.replace(\"" + url + "\"); </script>\n" +
                    "</h1>\n</body>\n</html>\n";
            return template;
        }
        File site = new File(project.reporting.outputDirectory);
        site.mkdirs();
        String[][] filenameArray = new String[][] {
                {"integration.html", "ci-management.html"},
                {"issue-tracking.html", "issue-management.html"},
                {"license.html", "licenses.html"},
                {"project-summary.html", "summary.html"},
                {"source-repository.html", "scm.html"},
                {"team-list.html", "team.html"}};
        for (int i=0; i<filenameArray.length; i++) {
            File file = new File(site, filenameArray[i][0]);
            org.codehaus.plexus.util.FileUtils.fileWrite(file, "UTF-8", contents(filenameArray[i][1]));
        }
        ]]>
                </script>
            </configuration>
        </execution>
    </executions>
</plugin>
```

This is an attempt to smooth over a slight change introduced in [MPIR-323](
https://issues.apache.org/jira/browse/MPIR-323). The URLs of several standard reports were changed to better match
the names of the reports and the Maven goals that produce them. For a new project, one wouldn't care, but for an
existing site, once a version is published using the version of the site plugin that includes this change, the new URLs
mean that existing inbound links will break, and if the menu definition was customized, it or other internal links may
be broken as well.

Fixing broken links in the site documentation is trivial, but I wanted to have redirect pages at the old URL locations
in case any organic search traffic was pointing at them. I could do this without BeanShell, just make six basic HTML
pages in my site files and they would be dutifully copied into the site. But, then I have six new files in my code
forever, that are almost identical except for which URL they redirect to, not very [DRY](
https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

I spent time looking at template tools and plugins to see if I could define my redirect page once and them make six
copies with slight variations, but didn't find anything. That is what I ended up creating here: there's a method that
defines the template for the page content, and then I iterate over the array of files to be created, creating them
with the Plexus `FileUtils` we saw used earlier.

This feels like it hits the sweet spot of being too niche to warrant having its own plugin, and yet too complex to try
to cobble together from Ant primitives in `maven-antrun-plugin`. The last example is a different story.

From: [https://github.com/chabala/brick-control-lab/pull/26/commits/94072024c98e1b591efc789699c3180cbe45f389](
https://github.com/chabala/brick-control-lab/pull/26/commits/94072024c98e1b591efc789699c3180cbe45f389)

```xml
<plugin>
    <groupId>com.github.genthaler</groupId>
    <artifactId>beanshell-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>late-site-add-canonical-urls</id>
            <phase>site</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <quiet>true</quiet>
                <script><![CDATA[
// Add canonical URLs to any site HTML pages that are missing them
String buildCanonicalUrl(String absoluteFilePath, String baseDirectory, String baseUrl) {
    String urlPath = (
        org.codehaus.plexus.util.FileUtils.basename(absoluteFilePath).equalsIgnoreCase("index.")
            ? org.codehaus.plexus.util.FileUtils.dirname(absoluteFilePath) + File.separator
            : absoluteFilePath
        ).substring(baseDirectory.length());
    return baseUrl + (File.separator.equals("/") ? urlPath : urlPath.replace(File.separator, "/"));
}
void insert(String filename, long offset, String content) throws IOException {
    File tempFile = File.createTempFile(org.codehaus.plexus.util.FileUtils.filename(filename), null);
    try {
        RandomAccessFile r = new RandomAccessFile(new File(filename), "rw");
        try {
            RandomAccessFile rtemp = new RandomAccessFile(tempFile, "rw");
            try {
                final long fileSize = r.length();
                java.nio.channels.FileChannel sourceChannel = r.getChannel();
                try {
                    java.nio.channels.FileChannel targetChannel = rtemp.getChannel();
                    try {
                        //move origin file contents from offset to end-of-file to temp file
                        sourceChannel.transferTo(offset, (fileSize - offset), targetChannel);
                        //clear origin file after offset
                        sourceChannel.truncate(offset);
                        r.seek(offset);        //move to new end-of-file
                        r.writeBytes(content); //write new content
                        long newOffset = r.getFilePointer(); //obtain offset for new end-of-file
                        targetChannel.position(0L); //set cursor in temp file to beginning for read
                        //move saved content from temp file back to end of origin file
                        sourceChannel.transferFrom(targetChannel, newOffset, (fileSize - offset));
                    } finally {
                        targetChannel.close();
                    }
                } finally {
                    sourceChannel.close();
                }
            } finally {
                rtemp.close();
            }
        } finally {
            r.close();
        }
    } finally {
        org.codehaus.plexus.util.FileUtils.forceDelete(tempFile);
    }
}
int countCanonicalLinks(File htmlFile, String projectUrl) throws IOException {
    //jsoup object scope
    org.jsoup.nodes.Document document = org.jsoup.Jsoup.parse(htmlFile, "UTF-8", projectUrl);
    return document.head().selectXpath("//link[@rel='canonical']").size();
}
void ensureCanonicalLink(String absoluteFilePath, String outputDirectory, String projectUrl) throws IOException {
    if (countCanonicalLinks(new File(absoluteFilePath), projectUrl) == 0) {
        //build canonical link tag
        String canonicalLinkTag = "<link rel=\"canonical\" href=\"" +
            buildCanonicalUrl(absoluteFilePath, outputDirectory, projectUrl.substring(0, projectUrl.length() - 1)) + "\" />\n";
        //find </head>
        int offset = org.codehaus.plexus.util.FileUtils.fileRead(absoluteFilePath, "UTF-8").indexOf("</head>");
        //insert link tag and linebreak
        insert(absoluteFilePath, offset, canonicalLinkTag);
    }
}
files = org.codehaus.plexus.util.FileUtils.getFilesFromExtension(
    project.reporting.outputDirectory, new String[] { "htm", "html" });
for (int i=0; i<files.length; i++) {
    ensureCanonicalLink(files[i], project.reporting.outputDirectory, project.url);
}
                            ]]>
                </script>
            </configuration>
        </execution>
    </executions>
</plugin>
```

I don't feel great about this one. I've been trying to improve the SEO of my project by following some best practices,
like [having a sitemap.xml](https://www.simplify4u.org/sitemapxml-maven-plugin/) listing all the pages, and having
pages specify their [canonical URL][12] via a link tag in the header.

[12]: https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls#use-rel=canonical-link-annotations

The problem is that almost every page in the typical Maven site is generated by a different tool. The main pages have
a theme, or rather a skin, that is applied uniformly, but that doesn't support adding canonical URLs out of the box.
Even if the skin could be enhanced to do so, that wouldn't cover all the other pages that don't participate in the
skin, like javadoc, test reports, etc. So what I end up needing is a post-site-generation fix-up phase, where each
generated HTML page is checked, and if it doesn't have a canonical URL specified, one is chosen and added.

That is what is happening in this BeanShell script. I again leverage Plexus `FileUtils` to collect all the HTML files,
then parse each one using [jsoup](https://jsoup.org/). If a canonical URL link tag is not found, it is generated and
added. I would have used jsoup to add the tag, but after some testing I found jsoup could not avoid rewriting the
whole file just to insert a tag, and because it didn't fully agree with the syntax of Maven's HTML output, there were
a lot of changes that looked like they might break pages.

So I use jsoup just for the parsing, and handle inserting missing canonical URL link tags myself. This taught me
there's no easy way to insert text in the middle of a file; one either builds a copy of the file with the text
inserted and then replaces the original with the copy, or move some of the original file into a temp file while
the new text is added, and then re-append the text from the temp file back into the original file, which is the path
I took.

This script ends up being too big in my opinion, and if I want to do this in other projects, I don't want
to copy this monstrosity around, so it's likely I'll create a custom Maven plugin for this at some point and
replace this script. The lack of try-with-resources support really sticks out here, and causes some deep nesting.

## Techniques for developing BeanShell scripts for Maven

Something that becomes clear quickly when developing a BeanShell script for Maven, is that writing it in the pom.xml
and running the build to test is not a productive development pattern. Instead, I recommend adding BeanShell (and
Plexus utils) temporarily as test dependencies, and making a JUnit test to exercise your logic:

```xml
<dependency>
    <groupId>org.beanshell</groupId>
    <artifactId>bsh</artifactId>
    <version>2.0b5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.codehaus.plexus</groupId>
    <artifactId>plexus-utils</artifactId>
    <version>3.5.1</version>
    <scope>test</scope>
</dependency>
```

Start out with the logic of your script. Any method of the script can just become a public method of the test class,
and the entrypoint of the script is the body of the test method. If you need Maven properties, you can mock them as
strings:

```java
@Test
public void testRun() throws IOException {
    String outputDirectory = "/home/chabala/code/brick-control-lab/target/site/";
    String projectUrl = "https://chabala.github.io/brick-control-lab/";
    String[] files = FileUtils.getFilesFromExtension(outputDirectory, new String[] {"htm", "html"});

    for (int i=0; i<files.length; i++) {
        ensureCanonicalLink(files[i], outputDirectory, projectUrl);
    }
}
```

When your logic is working, you can test through the BeanShell interpreter. Here I'm again mocking Maven properties
by added them into the interpreter context:

```java
@Test
public void testRunBeanshell() throws EvalError {
    Interpreter i = new Interpreter();
    i.set("project.reporting.outputDirectory", "/home/chabala/code/brick-control-lab/target/site/");
    i.set("project.url", "https://chabala.github.io/brick-control-lab/");
    i.eval("print(\"post-site beanshell\");\n" +
            "files = org.codehaus.plexus.util.FileUtils.getFilesFromExtension(\n" +
            "project.reporting.outputDirectory, new String[] { \"htm\", \"html\" });\n" +
            "print( files ); ");
}
```

This is the time when missing Java 5+ language features may start to appear, but you can easily modify the Java code
and then re-paste it into the BeanShell interpreter test. A good IDE like IntelliJ IDEA will help with converting a
clipboard full of Java code into a properly escaped string version without a lot of tedium. Likewise with copying
the code into a CDATA block in the pom.xml.

Copyright © 2024 [Greg Chabala](https://github.com/chabala)
