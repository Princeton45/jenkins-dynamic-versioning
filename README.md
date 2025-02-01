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




    