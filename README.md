# Dynamically Incrementing Application Version in a Jenkins Pipeline

 This README documents my journey of creating a project focused on dynamically incrementing an application's version within a Jenkins CI/CD pipeline. I leveraged a mix of technologies, including Jenkins, Docker, Github, Git, Java, and Maven, to achieve this.

## What I Accomplished

Here's a breakdown of what I did and the end result of each stage:


1. **Incrementing Version for the app:**

Below is the Maven command I used in the `increment version` stage to increment the apps patch version in the `pom.xml` file by 1.

```groovy
stages {
        stage("increment version") {
            steps {
                script {
                    echo "incrementing app version..."
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
```


2. **Incrementing the Docker image version:**
```groovy
stage("build image") {
            steps {
                script {
                    echo "building the docker image"
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t prince450/demo-app:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push prince450/demo-app:${IMAGE_NAME}"
                    }

                }
            }
        }
```

3. **Replace new version in Dockerfile:**

I also made sure that the `COPY` and `ENTRYPOINT` commands in the Dockerfile were using the proper incremented version name of the `JAR` application artifact 

Instead of `ENTRYPOINT` in the Dockerfile, I used `CMD` so we can match the method used in the `COPY` command.

```yaml
FROM amazoncorretto:8-alpine3.17-jre

EXPOSE 8080

COPY ./target/java-maven-app-*.jar /usr/app/
WORKDIR /usr/app

CMD java -jar java-maven-app-*.jar
```

4.  **Commit version update of Jenkins back to Git Repository:**

This step is crucial because without it, Jenkins will only increment the app version in the `pom.xml` file locally on its runner and not in the git repository. 

If we run the pipeline twice without this step, it will use the old version because the `pom.xml` in the Git repository was not updated with the new incremeneted version.

So now, when other developers want to commit something to the `main` branch for example, they first need to run a `git pull` or `git fetch` to get the changes to the `pom.xml` that Jenkins commited then continue working from there.

```groovy
 stage ("commit version update") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh "git status"
                        sh "git branch"
                        sh "git config --list"
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/Princeton45/jenkins-dynamic-versioning.git"
                        sh "git add ."
                        sh 'git commit -m "ci: version bump"'
                        sh "git push origin HEAD:main"
                    }
                }
            }
```

5.  **Prevent pipeline loops from CI version update commits**

If you have a webhook configured to automatically trigger the Jenkins pipeline when a push is made to the branch while having the `Commit version update of Jenkins back to Git Repository` step configured, this will cause a commit loop.

It creates a loop because:

1. You push code → webhook triggers Jenkins pipeline
2. Jenkins builds and commits version update → this is a new push
3. That new push triggers webhook again → Jenkins runs again
4. Repeat forever

To fix this loop, I needed to make sure the workflow detects that a commit was made from Jenkins (not from a human) and to ignore the trigger when the commit is from Jenkins.

I needed to install the `Ignore Committer Strategy` plugin in Jenkins

![ignore](https://github.com/Princeton45/jenkins-dynamic-versioning/blob/main/images/ignore.png)

Then I configured the `Ignore Committer Strategy` plugin to ignore commits that come from the author `jenkins@example.com` (I set the author for the `commit version update` step in the `Jenkinsfile`)

![ignore](https://github.com/Princeton45/jenkins-dynamic-versioning/blob/main/images/ignore2.png)


Full Jenkins Pipeline:

```groovy
def gv

pipeline {   
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("increment version") {
            steps {
                script {
                    echo "incrementing app version..."
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage("build app") {
            steps {
                script {
                    echo "building the application"
                    sh "mvn clean package"
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    echo "building the docker image"
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t prince450/demo-app:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push prince450/demo-app:${IMAGE_NAME}"
                    }

                }
            }
        }


        stage("deploy") {
            steps {
                script {
                    echo "deploying docker image"
                }
            }
        }

        stage ("commit version update") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh "git status"
                        sh "git branch"
                        sh "git config --list"
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/Princeton45/jenkins-dynamic-versioning.git"
                        sh "git add ."
                        sh 'git commit -m "ci: version bump"'
                        sh "git push origin HEAD:main"
                    }
                }
            }
        }               
    }
} 
```
    