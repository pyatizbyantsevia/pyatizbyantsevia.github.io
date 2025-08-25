---
title: "Final-keyword doesn't work properly in Jenkins pipeline"
date: 2025-08-24 12:00:00 +0300
# last_modified_at: 2025-08-24 15:30:00 +0300
tags: [jenkins, pipeline, groovy]
description: >-
  Unexpected, but that’s the reality of the Pipeline
---

## Self-sufficient example


```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                script {
                    final String str = "someVal"
                    str = "yetAnotherVal" // Exception expected
                    echo str
                }
            }
        }
    }
}
```

As a result of executing this Pipeline, we get the following output in stdout:

```console
yetAnotherVal
```

Now let’s just run the same code outside of the Pipeline DSL:

```groovy
final String str = "someVal"
str = "yetAnotherVal" // Exception expected
println str
```

> Main-Pipeline script can execute simple Groovy code
{: .prompt-tip }

But the result is the same.

## What about Shared library

Variables with local scope can still be overwritten, both in class methods and in scripts from the `vars/`

However, **properties** are protected. Attempting to change a `final-property` results in an exception:

```console
hudson.remoting.ProxyException: groovy.lang.ReadOnlyPropertyException:
Cannot set readonly property: myProperty for class: mainpackage.MyClass
```

## Why?

I asked this question[^1] in a discussion on [community.jenkins.io](https://community.jenkins.io) and received the following answer from one of the Jenkins developers:

> Pipeline script is run through a Jenkins-specific transformer which winds up essentially running an independent interpreter, ... nobody bothered to check for the final modifier in this context...


## What does it mean?

Until this issue[^2] is resolved, you should **not rely on the `final` keyword**. For example, when creating a global variable in a Pipeline, the interpreter does not enforce immutability:

```groovy
final SOME_GLOBAL_VAR
pipeline {
    agent any

    stages {
        stage('Prepare') {
            steps {
                script {
                    SOME_GLOBAL_VAR = calculate()
                }
            }
        }
        stage('Global var may be overriden here') {
            steps {
                script {
                    ...
                }
            }
        }
    }
}

def calculate() { ... }
```


[^1]: [https://community.jenkins.io/t/final-keyword-doesnt-work-properly-in-pipeline/33172](https://community.jenkins.io/t/final-keyword-doesnt-work-properly-in-pipeline/33172)
[^2]: [https://issues.jenkins.io/browse/JENKINS-76013](https://issues.jenkins.io/browse/JENKINS-76013)



