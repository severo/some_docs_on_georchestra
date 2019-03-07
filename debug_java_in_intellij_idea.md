# How to debug Java code in IntelliJ IDEA

Here, we stop at breakpoints in a Java class of the ogc-server-statistics module, when running the "test" target of maven.

## In a terminal

In a terminal, go to the `ogc-server-statistics` directory, and launch:

```bash
mvnDebug -DforkCount=0 -Dgeorchestra.datadir=/etc/georchestra test
```

where:

- `mvnDebug` build the project with convenient options for debug
- `-Dgeorchestra.datadir=/etc/georchestra test` runs the `test` maven goal (providing the `georchestra.datadir` Server property, it's not important here),
- `-DforkCount=0` is necessary for IDEA to stop at breakpoints (to know why, see [Debugging Tests](https://maven.apache.org/surefire/maven-surefire-plugin/examples/debugging.html) in Maven documentation, or a related [answer on StackOverflow](https://stackoverflow.com/a/6452204/7351594) - both are somewhat old, see [Fork Options and Parallel Test Execution](https://maven.apache.org/surefire/maven-failsafe-plugin/examples/fork-options-and-parallel-execution.html) on the replacement of `forkMode` by `forkCount`)

An equivalent alternative is:

```bash
MAVEN_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=localhost:8000" mvn -DforkCount=0 -Dgeorchestra.datadir=/etc/georchestra test
```

where:

- `MAVEN_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=localhost:8000"` launches a server, listening on port 5005 of localhost, and waiting for a connection before running.

The console will then show it's waiting for a client to connect:

```
Preparing to execute Maven in debug mode
Listening for transport dt_socket at address: 8000
```

## In IntelliJ IDEA

Import a maven project from the directory `ogc-server-statistics`, and in "Run" / "Edit configurations", create a new configuration ("+"), with the following options:

- Name: Remote-debugger
- Debugger mode: Attach to remote JVM
- Host: localhost
- Port: 8000
- Use module classpath: ogc-server-statistics

Then, setup breakpoints on the lines that you want to inspect.

Finally select the new configuration and launch "Debug Remote-debugger". You should see in the IntelliJ IDEA console:

```
Connected to the target VM, address: 'localhost:8000', transport: 'socket'
```

and then the execution should suspend at the first breakpoint. In the terminal, you should be able to see the Maven execution. 

## In Eclipse

Exactly the same way, you can debug from inside Eclipse.

First setup the project from `ogc-server-statistics` (you can create the Eclipse files from the `ogc-server-statistics` with the command: `mvn eclipse:eclipse -DdownloadSources=true`).

Then create a Debug configuration ("Run", "Debug Configurations"), select "Remote Java Application" and create a "New launch configuration" with the following options:

- Name: Remote-debugger
- Project: ogc-server-statistics
- Connection type: Standard (Socket Attach)
- Host: localhost
- Port: 8000

Then, setup breakpoints on the lines that you want to inspect.

Finally launch the new Debug configuration. The execution should suspend at the first breakpoint. In the terminal, you should be able to see the Maven execution.

## Alternative in IntelliJ IDEA

In IntelliJ IDEA, you can alternatively create a new configuration ("Run", "Edit Configuration") of type "Maven", with the following parameters:

- in the "Parameters" tab
  - Name: ogc-server-statistics [test]
  - Working directory: ...../georchestra/ogc-server-statistics
  - Command line: test
- in the "Runner" tab
  - Disable "Use project settings"
  - VM Options: -DforkCount=0 -Dgeorchestra.datadir=/etc/georchestra
  - JRE: Use Project JDK

Now: setup the breakpoints, and select the new configuration and launch the Debug. Much simpler, to run locally.
