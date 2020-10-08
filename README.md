# wildfly-galleon-postgresql

This repository contains a complete example on how to create a WildFly Bootable Jar which connects to an external 
postgresql database;

## Start a POSTGRESQL database

```shell script

podman login registry.redhat.io
podman pull registry.redhat.io/rhel8/postgresql-10
podman run --rm -d --name postgresql_database \
    -e POSTGRESQL_USER=user -e POSTGRESQL_PASSWORD=pass -e POSTGRESQL_DATABASE=db \
    -p 5432:5432 rhel8/postgresql-10

```

## Run the Bootable Jar

```shell script

export POSTGRESQL_DATASOURCE=PostgreSQLDS
export POSTGRESQL_USER=user
export POSTGRESQL_PASSWORD=pass
export POSTGRESQL_DATABASE=db
export POSTGRESQL_SERVICE_HOST=localhost
export POSTGRESQL_SERVICE_PORT=5432

java -jar galleon-postgresql-web/target/galleon-postgresql-web-bootable.jar

curl http://localhost:8080/

```

## custom-postgresql-datasource layer

This layer is the ultimate layer used by our web application and is included in the
`postgresql-datasources-galleon-pack` feature-pack;

The `wildfly-jar-maven-plugin` brings together our web app and the feature-pack into
the Bootable Jar: 

```xml
            <plugin>
                <groupId>org.wildfly.plugins</groupId>
                <artifactId>wildfly-jar-maven-plugin</artifactId>
                <version>2.0.0.Beta9</version>
                <configuration>
                    <feature-packs>
                        <feature-pack>
                            <groupId>org.wildfly</groupId>
                            <artifactId>wildfly-galleon-pack</artifactId>
                            <version>21.0.0.Beta1</version>
                        </feature-pack>
                        <feature-pack>
                            <artifactId>postgresql-datasources-galleon-pack</artifactId>
                            <groupId>org.example</groupId>
                            <version>1.0-SNAPSHOT</version>
                        </feature-pack>
                    </feature-packs>
                    <layers>
                        <layer>cloud-profile</layer>
                        <layer>custom-postgresql-datasource</layer>
                    </layers>
                </configuration>
                ...
            </plugin>

```

## postgresql-datasources-galleon-pack feature-pack

It's created by the homonym maven module using the `wildfly-galleon-maven-plugin` maven plugin:

```xml
            <plugin>
                <groupId>org.wildfly.galleon-plugins</groupId>
                <artifactId>wildfly-galleon-maven-plugin</artifactId>
                <version>${version.org.wildfly.galleon-plugins}</version>
                <executions>
                    <execution>
                        <id>postgresql-datasources-galleon-pack-build</id>
                        <goals>
                            <goal>build-user-feature-pack</goal>
                        </goals>
                        <phase>compile</phase>
                    </execution>
                </executions>
            </plugin>
```

The `wildfly-galleon-maven-plugin` maven plugin looks under `src/main/resources` for `modules`, 
`layers`, `feature_groups`, etc. which make up the feature-pack;
