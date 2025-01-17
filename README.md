# Liferay DXP with JBoss EAP Docker Image

First clone this repository: `git clone https://github.com/balazsletai/liferay-jboss-docker.git`

Download the zip file of JBoss EAP 7.4.0 (`jboss-eap-7.4.0.zip`) from [this link](https://files.liferay.com/private/apps/redhat/jbosseap/7/) and place it in the `liferay-jboss-docker/build/files/jboss` folder.

In the `compose.yaml` file, you can set the Liferay version by changing the value of `LR_IMAGE_TAG`.

To run the environment from the `liferay-jboss-docker` folder, run the following command:

    docker compose up --build

The Liferay Docker image is created from the Dockerfile in the `liferay-jboss-docker/build` folder.

***


## Configuration in the Dockerfile

### Default versions

- **Liferay DXP Version:** 2024.Q4.2 (This can be overridden by the LR\_IMAGE\_TAG set in the `compose.yaml` file)

- **Base Image:** Red Hat UBI 7.9

- **Java Version:** Java 21 (Azul Zulu JDK)


### **To change the Java version used by the Docker image:**

1. **Modify the JDK Version:**

   - Locate the term `zulu21-jdk` in the Dockerfile.

   - Replace `21` with the desired version. For example:

     - For Java 17, use `zulu17-jdk`.

2. **Adjust Source and Target VM Versions:**

   - Search for the strings `source-vm=` and `target-vm=` in the Dockerfile.

   - Update their values to match the desired Java version. For example:

     - For Java 17, set:

           source-vm="17"
           target-vm="17"


### Java Runtime Compatibility

- **Java JDK 8:**

  - Removed in 2024.Q2 and later releases.

- **Java JDK 11:**

  - Deprecated in 2024.Q1 and removed in 2024.Q3 and later releases.

- **Java JDK 17 and JDK 21:**

  - Certified for 2024.Q2 or later releases.

  - **JDK 21** is the recommended runtime for Liferay DXP.

***


## Additional Resources

For more details about compatibility and quarterly releases, visit the official Liferay documentation:\
[Quarterly Releases Compatibility Matrix](https://help.liferay.com/hc/en-us/articles/4411310034829-Liferay-DXP-Quarterly-Releases-Compatibility-Matrix)

***

Feel free to submit issues or pull requests to improve this repository!
