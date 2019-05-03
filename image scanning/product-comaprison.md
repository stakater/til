# Image Scanning
## Popular Open Source tools:

1. Anchore
2. SonarQube
3. Clair
4. Aqua Micro Scanner

## Anchore
The Anchore engine is an open source project that inspects, analyzes, and certifies Docker images. Anchore is available as a Docker image that can be run standalone or with orchestration platforms such as Kubernetes.
1. Fetches security data from Anchore’s hosted cloud service.
2. Jenkins plugin available.
3. Helm chart available
4. Anchore engine can be accessed through a RESTful API or via the Anchore CLI.
5. In our workflow, Scanning step can be added after the image is built and pused to the image repo. If the scan passes, the pipeline passes fails otherwise.

## Clair
Clair is an open source project for the static analysis of vulnerabilities in application containers (currently including appc and docker).
1. Jenkins Plugin not available
2. Helm Chart available
3. Clair Scan-step can be added to a YAML file.
4. Updates vulnerability database from configured sources
5. Clair API indexes container images creating a list of features present in the image and stores them in the database.
6. API can be queried for vulnerabilities of a particular image; correlating vulnerabilities and features is done for each request, avoiding the need to rescan images.
7. Difference clients can be integrated as per requirement: [clients](https://github.com/coreos/clair/blob/master/Documentation/integrations.md)

## SonarQube
SonarQube is an automatic code review tool to detect bugs, vulnerabilities and code smells in the code. It can integrate with existing workflow to enable continuous code inspection across the project branches and pull requests.
1. Most used for code quality but also has features for vulnerability detection/scanning
2. Jenkins Plugin Available
3. Helm Chart available
4. Also checks for Code Coverage, Code smells, Block duplication etc.
5. SonarQube can be inserted as a gateway step after the image is built to ensure code quality/vulnerabilities.


## Aqua Micro Scanner
Aqua Security's MicroScanner checks container images for vulnerabilities. If the scanned image has any known high-severity issue, MicroScanner can fail the image build, making it easy to include as a step in your CI/CD pipeline.
1. Jenkins Plugin Available
2. Displays results in an HTML page
3. List down detailed Vulnerabilities
4. Defines Policy/Gateway to pass or fail a pipeline
5. Needs to edit Dockerfile
6. Needs to register and take a token from AquaSecurity
7. Only OpenSource package scanning


Anchore and Clair are widely used. SonarQube can be an option if code quality checks are also in consideration. MicroScanner is very basic and does not integrates well (Need to change every Dockerfile).