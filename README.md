<!-- markdownlint-disable MD033 -->
# Java Caller for Node.js

[![Version](https://img.shields.io/npm/v/java-caller.svg)](https://npmjs.org/package/node-java-caller)
[![Downloads/week](https://img.shields.io/npm/dw/java-caller.svg)](https://npmjs.org/package/java-caller)
[![Downloads/total](https://img.shields.io/npm/dt/java-caller.svg)](https://npmjs.org/package/java-caller)
[![CircleCI](https://circleci.com/gh/nvuillam/node-java-caller/tree/master.svg?style=shield)](https://circleci.com/gh/nvuillam/node-java-caller/tree/master)
[![codecov](https://codecov.io/gh/nvuillam/node-java-caller/branch/master/graph/badge.svg)](https://codecov.io/gh/nvuillam/node-java-caller)
[![GitHub contributors](https://img.shields.io/github/contributors/nvuillam/node-java-caller.svg)](https://gitHub.com/nvuillam/node-java-caller/graphs/contributors/)
[![GitHub stars](https://img.shields.io/github/stars/nvuillam/node-java-caller?label=Star&maxAge=2592000)](https://GitHub.com/nvuillam/node-java-caller/stargazers/)
[![License](https://img.shields.io/npm/l/java-caller.svg)](https://github.com/nvuillam/node-java-caller/blob/master/LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

Lightweight cross-platform javascript module to easily call java commands from Node.js sources.

- Automatically installs java (currently 1.8) if not present on the system
- Uses node [spawn](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options) method to perform the call

## Installation

```shell
npm install java-caller --save
```

## Usage

```javascript
const JavaCaller = require('java-caller');
const java = new JavaCaller(JAVA_CALLER_OPTIONS);
const {status, stdout, stderr} = java.run(JAVA_ARGUMENTS,JAVA_CALLER_RUN_OPTIONS);
```

### JAVA_CALLER_OPTIONS

| Parameter          | Description                                                                                                                                                           | Default value        | Example                                  |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|------------------------------------------|
| jar                | Path to executable jar file                                                                                                                                           |                      | `"myfolder/myjar.jar"`                   |
| classPath          | If jar parameter is not set, classpath to use<br/>Use `:` as separator (it will be converted if runned on Windows)                                                    | `.` (current folder) | `"java/myJar.jar:java/myOtherJar.jar"`   |
| mainClass          | If classPath set, main class to call                                                                                                                                  |                      | `"com.example.MyClass"`                  |
| rootPath           | If classPath elements are not relative to the current folder, you can define a root path. <br/> You may use `__dirname` if you classes / jars are in your module folder    | `.` (current folder) | `"/home/my/folder/containing/jars"`      |
| javaExecutable     | You can force to use a defined java executable, instead of letting java-caller find/install one                                                                       |                      | `"/home/some-java-version/bin/java.exe"` |
| minimumJavaVersion | Minimum java version to be used to call java command.<br/> If the java version found on machine is lower, java-caller will try to install and use the appropriate one | `1.8`                | `11`                                     |
| maximumJavaVersion | Maximum java version to be used to call java command.<br/> If the java version found on machine is upper, java-caller will try to install and use the appropriate one |                      | `10`                                     |
| additionalJavaArgs | Additional parameters for JVM                                                                                                                                         |                      | `["-Xms256m","-Xmx2048m"]`                |

### JAVA_ARGUMENTS

The list of arguments can contain both arguments types together:

- Java arguments (**-X*** , **-D***). ex: `"-Xms256m"`, `"-Xmx2048m"`
- Main class arguments (sent to `public static void main method`). ex: `"--someflag"` , `"--someflagwithvalue myVal"` , `"-c"`

Example: `["-Xms256m", "--someflagwithvalue myVal", "-c"]`

### JAVA_CALLER_RUN_OPTIONS

| Parameter      | Description                                                                                                                                                                      | Default         | Example                 |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|-------------------------|
| [detached](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options)   | If set to true, node will node wait for the java command to be completed.<br/>In that case, `childJavaProcess` property will be returned, but `stdout` and `stderr` may be empty, except if an error is triggered at command execution | `false`         | `true`                  |
| waitForErrorMs | If detached is true, number of milliseconds to wait to detect an error before exiting JavaCaller run                                                                             | `500`           | `2000`                  |
| [cwd](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options)        | You can override cwd of spawn called by JavaCaller runner                                                                                                                        | `process.cwd()` | `some/other/cwd/folder` |

## Examples

Call a class located in classpath

```javascript
const java = new JavaCaller({
    classPath: 'test/java/dist',
    mainClass: 'com.nvuillam.javacaller.JavaCallerTester'
});
const { status, stdout, stderr } = await java.run();
```

Call a class located in classpath with java and custom arguments

```javascript
const java = new JavaCaller({
    classPath: 'test/java/dist',
    mainClass: 'com.nvuillam.javacaller.JavaCallerTester'
});
const { status, stdout, stderr } = await java.run(['-Xms256m', '-Xmx2048m', '--customarg nico']);
```

Call a class in jar located in classpath

```javascript
const java = new JavaCaller({
    classPath: 'test/java/jar/JavaCallerTester.jar',
    mainClass: 'com.nvuillam.javacaller.JavaCallerTester'
});
const { status, stdout, stderr } = await java.run();
```

Call a runnable jar

```javascript
const java = new JavaCaller({
    jar: 'test/java/jar/JavaCallerTesterRunnable.jar',
});
const { status, stdout, stderr } = await java.run();
```

Call a detached java process

```javascript
const java = new JavaCaller({
    classPath: 'test/java/dist',
    mainClass: 'com.nvuillam.javacaller.JavaCallerTester'
});
const { status, stdout, stderr, childJavaProcess } = await java.run(['--sleep'], { detached: true });

// Kill later the java process if necessary
childJavaProcess.kill('SIGINT');
```

You can see **more examples in** [**test methods**](https://github.com/nvuillam/node-java-caller/blob/master/test/java-caller.test.js)

## TROUBLESHOOTING

Set environment variable `DEBUG=java-caller` before calling your code using java-caller module, and you will see the java commands executed.

Example debug log:

```shell
java-caller Found Java version 1.80131 +1s
java-caller Java command: java -Xms256m -Xmx2048m -cp C:\Work\gitPerso\node-java-caller\test\java\dist com.nvuillam.javacaller.JavaCallerTester -customarg nico +1ms
```

## CONTRIBUTE

Contributions are very welcome !

Please follow [Contribution instructions](https://github.com/nvuillam/node-java-caller/blob/master/CONTRIBUTING.md)

## RELEASE NOTES

## [1.1.0] 2020-08-10

- Return `javaChildProcess` when `detached` is true, so it can be used to be killed later

## [1.0.0] 2020-08-10

- Initial version

____

See complete [CHANGELOG](https://github.com/nvuillam/node-java-caller/blob/master/CHANGELOG.md)
