# Demo Project: Dynamically Incrementing Application Version in a Jenkins Pipeline

 This README documents my journey of creating a project focused on dynamically incrementing an application's version within a Jenkins CI/CD pipeline. I leveraged a mix of technologies, including Jenkins, Docker, Github, Git, Java, and Maven, to achieve this.

## What I Accomplished

Here's a breakdown of what I did and the end result of each stage:

1. **Jenkins and GitLab Integration:**
    *   I installed the GitLab Plugin in Jenkins, enabling communication between the two platforms.
    *   I configured a GitLab access token and established a connection within the GitLab project settings, allowing Jenkins to interact with my GitLab repository.
    *   I set up Jenkins to automatically trigger the CI pipeline whenever a code change is pushed to GitLab. This ensures that the version increment and build process kick off without manual initiation.
    *   **Picture Suggestion:** A screenshot showing the GitLab Plugin configuration in Jenkins.

2. **CI Pipeline Configuration:**
    *   **Increment Patch Version:** I created a CI step that automatically increments the patch version of my application each time the pipeline runs.
    *   **Picture Suggestion:** A screenshot of the Jenkins pipeline stage responsible for incrementing the version.
    *   **Build Java Application:** I configured a step to build the Java application using Maven. This step also cleans up any old artifacts to ensure a fresh build every time.
    *   **Picture Suggestion:** A screenshot of the Jenkins console output showing the successful Maven build.
    *   **Build Docker Image with Dynamic Tag:** I set up a step to build a Docker image of the application. The key here is that the image tag is dynamically generated based on the incremented version number.
    *   **Picture Suggestion:** A screenshot of the Dockerfile or the pipeline script that handles the image build and dynamic tagging.
    *   **Push Image to Docker Hub:** I added a step to push the newly built Docker image to my private Docker Hub repository. This makes the latest version of the application readily available for deployment.
    *   **Picture Suggestion:** A screenshot of the Docker Hub repository showing the pushed image with the dynamic tag.
    *   **Commit Version Update to Git:** I configured a step to commit the updated version number back to the Git repository. This keeps the repository in sync with the application's current version.
    *   **Picture Suggestion:** A screenshot of the GitLab commit history showing the version update commit made by Jenkins.

3. **Preventing Commit Loops:**
    *   I configured the Jenkins pipeline to not trigger automatically on commits made by the CI build itself. This prevents an infinite loop where the version update commit triggers another build, which then updates the version again, and so on.
    *   **Picture Suggestion:** A screenshot of the Jenkins pipeline configuration showing the setting to prevent triggering on CI build commits.

