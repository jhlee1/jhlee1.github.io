# Setup Sonarqube With Docker



## I. Install Docker

- [Check this out](https://docs.docker.com/install/)

## II. Install & Run SonarQube container with docker

- `docker pull sonarqube`
  - Get the lastest image of sonarqube image
- `docker run -d --name sonarqube -p 9000:9000 sonarqube`
  - Run docker sonarqube image with 
    - detach mode
    - port 9000
    - name sonarqube
- `docker ps`
  - Check if the image is currently running

## III. Add maven plugin in your project

- In your pom.xml
  - So far, 3.6.0.1398 is the newest version for sonar scanner. (2019. 8. 14)

```xml
<project>
...
	<build>
		<plugins>
            ...
            
			<plugin>
				<groupId>org.sonarsource.scanner.maven</groupId>
				<artifactId>sonar-maven-plugin</artifactId>
				<version>3.6.0.1398</version>
  		</plugin>
  	</plugins>
	</build>
</project>
```

## IV. Run with sonar command

- Run with maven plugin button on your intellij 
- Or use Command  `mvn clean install org.sonarsource.scanner.maven:sonar-maven-plugin:${sonarVersion}:sonar`
  - You should make sure the sonarqube is running
  - Change the `${sonarVersion}` version with your version
- After you check the `build success` message checkout the sonarqube server (http://localhost:9000)

## V. Connect SonarLint with SonarQube in Intellij

- Add SonarQube Server Preferences > SonarQube General Settings
  - The admin account user name is `admin` with password `admin`
- Add the server you connected above in SonarQube Project Settings

## VI. Setup DB for SonarQube (Postgres)

- Pull and start postgres

  ```
  docker run --name postgres -e POSTGRES_PASSWORD=password1234 -d postgres
  ```

  - I guess `POSTGRES_PASSWORD` field is not necessary

- Connect Sonar Container to postgres

  ```
  docker run -d -p 10000:9000 --name sonarqube --link postgres:sonardb -e sonar.jdbc.username=sonar -e sonar.jdbc.password=sonar -e sonar.jdbc.url=jdbc:postgresql://sonardb/sonar sonarqube
  ```

  - I am setting the db user name as `sonar` and password as `sonar`
  - Linking postgres container as sonardb
  - Using environment variables for sonarqube db setup is not a recommended way according to the sonarqube guide in the docker hub
  - I guess it will be better to change the property file in

  ```
  /opt/sonarqube/conf/sonar.properties
  ```

- enter postgres container

- ```
  docker exec -it postgres /bin/bash
  ```

- Enter psql as username = postgres
  `psql --username=postgres`

  - Check user roles

    - `\du`
    - the user sonar would not exist

  - create user `sonar` having password `sonar`

  - ```sql
    CREATE ROLE sonar LOGIN CREATEDB PASSWORD 'sonar';
    ```

  - create table sonar to save the data from sonarqube. It's owner user is `sonar`, which I created just above

    ```sql
    CREATE DATABASE sonar OWNER sonar;
    ```

    



## VII. Resources

- [About sonarqube image on docker hub](https://hub.docker.com/_/sonarqube)
- [SonarQube with Jenkins in Korean](https://cubenuri.tistory.com/222)
- [PostgreSQL image on docker hub](https://hub.docker.com/_/postgres)

