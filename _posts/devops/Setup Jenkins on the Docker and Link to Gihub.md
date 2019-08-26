# Setup Jenkins on the Docker and Link to Gihub

## 1. Pull image and run Jenkins on the docker

- `docker run -d -p 1500:8080 --name jenkins jenkins/jenkins:lts`
  - I set the host port to 1500 to avoid conflictions because 8080 is used a lot in other applications
  - Put :lts to use `Long Term Support` version

## 2. Setup Jenkins

- To login as admin it requires initial admin password
  - Enter the docker image `docker exec -it jenkins /bin/bash`
  - Check the initial admin password inside the image using tail 
    `tail /var/jenkins_home/secrets/initialAdminPassword`

## 3. SSH Key for Github

- Generate SSH Key
  `ssh-keygen -t rsa -f github-jenkins`

- Copy private key
  `pbcopy < github-jenkins`

- Copy public key 

  `pbcopy < github-jenkins.pub`

## 4. Connect Jenkins to Sonarqube

```bash
docker run -d -p 1500:8080 --name jenkins --link sonarqube jenkins
```

## 5. Create a jenkins job to use SonarQube

```bash
clean install org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.host.url=http://sonarqube:9000
```



