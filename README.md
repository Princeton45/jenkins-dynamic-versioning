# Demo Project: Dynamically Incrementing Application Version in a Jenkins Pipeline

 This README documents my journey of creating a project focused on dynamically incrementing an application's version within a Jenkins CI/CD pipeline. I leveraged a mix of technologies, including Jenkins, Docker, Github, Git, Java, and Maven, to achieve this.

## What I Accomplished

Here's a breakdown of what I did and the end result of each stage:


1. **Incrementing Version for the app:**

Below is the Maven command I used in the `increment version` stage to increment the apps patch version in the `pom.xml` file by 1.

```groovy

```

<have claude explain the code>


2. **Incrementing the Docker image version:**


3. **Replace new version in Dockerfile:**

I also made sure that the `COPY` and `ENTRYPOINT` commands in the Dockerfile were using the proper incremented version name of the `JAR` application artifact 

Instead of `ENTRYPOINT` in the Dockerfile, I used `CMD` so we can match the method used in the `COPY` command.

4.  **Commit version update of Jenkins back to Git Repository:**

This step is crucial because without it, Jenkins will only increment the app version in the `pom.xml` file locally on its runner and not in the git repository. 

If we run the pipeline twice without this step, it will use the old version because the pom.xml in the Git repository was not updated with the new incremeneted version.

So now, when other developers want to commit something to the `main` branch for example, they first need to run a `git pull` or `git fetch` to get the changes to the `pom.xml` that Jenkins commited then continue working from there.




    