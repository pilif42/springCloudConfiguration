# springCloudConfiguration
Spring Cloud Config repo for my personal apps


Configuration files for components should be maintained in the following format in environment-specific repositories and on the master branch only,
		- {application-name}-{profile}.properties 
		- For example, the configuration file of the User API for the ITE environment will be cspuserapi-ite.properties.

		
All password/confidential information in configuration files should be encrypted. Encrypted values can be generated using Spring CLI, Information to install and generate encrypted values is available at 
http://projects.spring.io/spring-cloud/spring-cloud.html#_spring_boot_cloud_cli Properties will be encrypted using a public key and the Spring config server will be using a private key to decrypt property values and pass to clients.
 You can find the public key for the relevant environment at \springCloudConfiguration\keys\springconfigserver-public.pem.
		- spring encrypt <property value> --key springconfigserver-public.pem where the spring command is part of spring-cli.
Generated encrypted values should be appended with {cipher} and replaced in configuration files. Example: fixture.auth.password={cipher}903954da6804095d7e0003e3d9b9a70af575be5f73a1d02fc7151a5a078fa674
Going forward, each environment should have its own repository like dev/ITE/Prod and maintaining different encryption Keys. The environment owning person should be responsible for generating encrypted values.


Spring cloud server is bundled as a docker image and pushed to docker hub and will be running as AWS ECS services as other Spring-Boot Application.
Expected configuration for docker are as follows which will be injected as environment variable,
		export spring_application_name=configserver
		export encrypt_failOnError=true
		export server_port=8888
		export management_context-path=/admin
		export spring_cloud_config_server_git_uri=git@github.com:pilif42/springCloudConfiguration.git
		export security_user_password=<basic auth password used by Spring boot client applications>
		export encrypt_key=<private key to decrypt values>
		
	  
To activate Spring-config-client, Following configuration should be added in the client project,
	1. In build.gradle, the following dependency should be added,
		  compile 'org.springframework.cloud:spring-cloud-config-client:1.0.2.RELEASE'

	2. Spring boot launcher should be annotated with @Configuration.

	3. As all spring boot applications are bundled as docker images and configurations will be injected as environment variables, Following environment variables need to be appended in existing configuration files.
	   	export spring_cloud_config_uri=<environment specific route 53 Spring-config-server alias>
		export spring_cloud_config_username=user
		export spring_cloud_config_password=<basic auth password configued cloud server>
		export spring_cloud_config_failfast=true
		export spring_application_name=<api name with which file stored on github>
		export spring_profiles_active=<environment suffix with which configuration file stored on github>


# Client Application Command Line
When running from the command line, the same information that is normally specified as part of the `bootstrap.yml` can also be specified as program arguments. The following is taken from the `csp-appstore-api` project. 	
java -jar \
--spring.cloud.config.uri=http://ec2-54-171-191-160.eu-west-1.compute.amazonaws.com:8080 \
--spring.cloud.config.failfast=true \
--spring.application.name=cspappstoreapi \
--spring.profiles.active=dev
```
