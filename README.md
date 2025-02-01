# Demo Project: Dynamically Incrementing Application Version in a Jenkins Pipeline

 This README documents my journey of creating a project focused on dynamically incrementing an application's version within a Jenkins CI/CD pipeline. I leveraged a mix of technologies, including Jenkins, Docker, Github, Git, Java, and Maven, to achieve this.

## What I Accomplished

Here's a breakdown of what I did and the end result of each stage:


1. **Incrementing Version for the app:**

Below is the Maven command I used in the `increment version` stage to increment the apps patch version in the `pom.xml` file by 1.

```groovy

stages {
        stage("increment version")
            steps {
                script {
                    echo "incrementing app version..."
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                }
            }
```

I use `build-helper:parse-version` to read my current version (like "1.1.0")

With `versions:set`, I prepare to set my new version

In `-DnewVersion`, I specify my version pattern:
- I keep my major version (first number) using `${parsedVersion.majorVersion}`
- I keep my minor version (second number) using `${parsedVersion.minorVersion}`
- I increment my patch version (last number) using `${parsedVersion.nextIncrementalVersion}`

Finally, I make my changes permanent with `versions:commit`



    