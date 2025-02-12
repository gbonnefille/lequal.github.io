# CAT - User guide
This guide will explain how to install and start Docker-CAT. Docker-CAT may work better on linux systems.

## Quick setup
For a quick setup, you can read the
[readme file](https://github.com/lequal/docker-cat) from the official repo.

## Complete setup

### Pre-requisite
Before using Docker-CAT please check that docker is installed and running.

- [Docker documentation](https://docs.docker.com/install/)
- Start docker daemon: `sudo service docker start`

### Getting docker image

#### From the docker hub
The easiest way to get Docker-CAT is to clone Docker image from docker hub with `docker pull lequal/docker-cat`

#### Build the image yourself
If needed, you can build the image yourself. If you edit the Docker-CAT image feel free to create a pull request !

To build your own image, just clone the repo and use `docker build`.
```
git clone https://github.com/lequal/docker-cat
cd docker-cat
docker build -t <tag> .
```

### Configure & start Docker-CAT
Before starting Docker-CAT you must identify the shared folder and files that should be accessible by SonarQube.

#### Choose the shared folder
The docker have root access to the shared folder. It is recommended to use a dedicated folder.

#### Autorize SonarQube to access files inside the shared folder
The `ALLOWED_GROUPS` variable let you define a list of GID (Group ID) associated with the `sonarqube` user. All files
inside the shared folder should be accessible by at least one of this group to run code analysis.

To find GID in your case, you can follow these steps:
- Use `ls -l` to get group associated with each file.
- For every groups, you can get the GID with `cat /etc/groups` or
`getent <group_name> | cut -d : -f3`

Once you know all GID, add `-e ALLOWED_GROUPS="<GID1>;<GID2>;<...>"` in the `docker start` command.

#### Start Docker-CAT
Now, you just have to use this command to get started ! Just replace `<shared_folder>` and `<GID>`.
```
docker run -v <shared_folder>:/media/Sf_Shared:rw -p 9000:9000 -p 9001:9001 -e ALLOWED_GROUPS="<GID>" lequal/docker-cat:latest
```
If everything is ok, you can test Docker-CAT at
[http://localhost:9000](http://localhost:9000/)


## Using Docker-CAT
To use interact with Docker-CAT, you have to use the SonarQube web interface. Docker-CAT includes some plugins in
SonarQube.

**WARNING**: if you want persistent data, be sure to link a database (see the corresponding section).

**TIPS**: After an analisys, docker-CAT generate a report. If you lose it you can generate report again
by selecting "More => CNES Report" in the menu. Then, you can save your report in a safe place.

To run an analysis, copy your code in shared folder then:

- Open [http://localhost:9000](http://localhost:9000/) and log in.
    - Default login: User = `admin` / Password = `admin`
    - If you use Docker-CAT on a server, you should change the password !
- Click on *More -> CNES Analisys*.
- Fill the form:
    - **Project key**: Key for sonar, must be unique (one per project)
    - **Workspace**: the path to your project (relative to shared folder)
        - E.g.: your project is saved on `/home/user/docker/organization/project`, shared folder is `/home/user/docker`,
        you must write `organization/project`
    - **Sources**: path to the source, relative to project root.
    - **sonar-project.properties**: For advanced usage you can edit this file. For java project you can uncomment some
    lines in the default file. (Guideline are directly visible in the form)
    - **Quality Profile**: It defines the rules you want to test. If the project contains multiple languages, you can choose more than one profile. **You can only use one profile per language**. See the next section to learn more about availables profiles.
- Start analysis !
- Check results on the Sonar web interface, on the report generated in shared folders or by using *More -> CNES Report*
link in the menu.

#### Choose a profile
With docker-CAT you have a lot of profile. For most of the languages there is a profile called
"Sonar way". This is a default profile created by SonarQube developers, this general profile
can fit most of the projects.

You can also find CNES profiles. They are marked with A, B, C or D (for example: CNES_CPP_D).
This profile are created based on CNES standards (RNC) and the letters represent a criticity
level. If you are doing a project with CNES, you can contact the Quality Assurance departement
to get the best profile for your project. You can also check rules by using the "Rules" tab in
the SonarQube interface.

If you want to check all the rules you can use profiles prefixed with "ALL_". It's an easy
way to do a very complete analysis but with a large project you may find a lot of issues in
your code.

You can also find some profile from FindBugs (JAVA). For more information about FindBugs
you can refer to the [FindBugs repo](https://github.com/spotbugs/sonar-findbugs/).

#### Link docker CAT to a database
If you already have a database you can just set these environment variables on docker:

- `SONARQUBE_JDBC_USERNAME`
- `SONARQUBE_JDBC_PASSWORD`
- `SONARQUBE_JDBC_URL`

In your `docker start` command you need to add `-e SONARQUBE_JDBC_USERNAME=value -e SONARQUBE_JDBC_PASSWORD=value -e SONARQUBE_JDBC_URL=value`


If you did not have database, you can use docker to create your database. Just download the
`docker-compose.yml` from [the repo](https://github.com/lequal/docker-cat), install docker-compose with your
package manager (apt, yum, ...) and use `docker-compose up` in your terminal. It will start Docker-CAT and PgSQL



#### Add plugin
There is 2 ways to add plugin:
- Add plugin in the docker file and build (see "Build the image yourself" above)
- Copy plugin in docker and restart:
    - `docker cp <plugin>.jar <docker_image_name>:/opt/sonarqube/extensions/plugins/`
    - `docker restart <docker_image_name>`
