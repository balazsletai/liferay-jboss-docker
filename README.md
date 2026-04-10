# Liferay DXP with JBoss EAP 8 Docker Image

First clone this repository: `git clone https://github.com`

Download the zip file of JBoss EAP 8.1.0 from Redhat official site or [from my drive](https://drive.google.com/file/d/19-Gys-FYdJrNS8I5OuYogZKr98Gqeoob/view?usp=sharing) and place the zip file in the `liferay-jboss-docker/build/files/jboss` folder.

In the `compose.yaml` file, you can set the Liferay version by changing the value of `LR_IMAGE_TAG`.

Copy your license into `liferay-jboss-docker/override_files/liferay/deploy` folder.

To run the environment from the `liferay-jboss-docker` folder, run the following command:

    docker compose up --build

The Liferay Docker image is created from the Dockerfile in the `liferay-jboss-docker/build` folder.

***

## Configuration in the Dockerfile

### Default versions

- **Liferay DXP Version:** 2026.Q1.2-LTS (This can be overridden by the LR_IMAGE_TAG set in the `compose.yaml` file)

- **Application Server:** JBoss EAP 8.1.0 (Jakarta EE 10)

- **Database:** PostgreSQL 17

- **Elasticsearch:** 8.19.0 (Built from the `./elastic` folder)

- **Base Image:** Red Hat UBI 9.4

- **Java Version:** Java 21 (Azul Zulu JDK)

- **Patching tool:** 4.0.10

### Java Runtime Compatibility

- **Java JDK 21:**

  - **JDK 21** is the required and recommended runtime for Liferay DXP 2026.Q1 and JBoss EAP 8.1.0.

- **Note on Jakarta EE 10:**

  - Liferay 2026.Q1 and JBoss 8 utilize the `jakarta.*` namespace. Ensure all custom modules and configurations have been migrated from `javax.*`.

### **To change the Java version used by the Docker image:**

1. **Modify the JDK Version:**

   - Locate the term `zulu21-jdk` in the Dockerfile.

   - Replace `21` with the desired version.

2. **Adjust Source and Target VM Versions:**

   - Search for the strings `source-vm=` and `target-vm=` in the Dockerfile.

   - Update their values to match the desired Java version. For example:

           source-vm="21"
           target-vm="21"

***

## Patching Liferay

You can place the hotfix in the `override_files/liferay/patching` folder before starting up the environment.

In case you need to upgrade the patching tool, download the latest from [Downloads page](https://liferay.com), then unzip it and replace the patching-tool in the files folder.

Note: make sure the `default.properties` will remain in the patching-tool folder.

## License for Liferay

Place it in /override_files/liferay/deploy folder.

## Additional Resources

For more details about compatibility and quarterly releases, visit the official Liferay documentation:
[Quarterly Releases Compatibility Matrix](https://liferay.com)

***

Feel free to submit issues or pull requests to improve this repository!
