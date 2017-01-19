
## VERTX - MAVEN - DOCKER


### VERTX

Vertx es una kit de herramientas para construir aplicaciones reactivas en el JVM

Vertx genera un servidor http en el puerto 8080.
Vamos a realizar una aplicacion que nos devuelva un hello world.

#### REQUERIMIENTOS

Se requiere tener los siguientes prerequisitos para poder realizar el ejercicio:

- Java Jdk 1.8
- Maven
- Docker



#### INSTALACIÓN

- Crear directorio hello world

```bash
mkdir vertx_hello_world
cd vertx_hello_world
```

- crear estructura asi:

```
.
├── pom.xml
├── src
│   ├── main
       └── java


```
- Ejecutamos el siguiente comando:

```bash
touch pom.xml

mkdir -p src/main/java    

```

- Crear el archivo pom.xml con lo siguiente:

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>io.vertx.blog</groupId>
  <artifactId>my-first-app</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>io.vertx</groupId>
      <artifactId>vertx-core</artifactId>
      <version>3.0.0</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.3</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>

```

- Estamos declarando la dependencia a vertx-core
- Y estamos poniendo el compilador que use Java 8.
- Crear el siguiente directorio 

```bash
cd src/main/java

mkdir -p io/vertx/blog/first

cd io/vertx/blog/first

touch MyFirstVerticle.java

```
- Editar el archivo y colocar la siguiente información:


```java
package io.vertx.blog.first;

import io.vertx.core.AbstractVerticle;
import io.vertx.core.Future;

public class MyFirstVerticle extends AbstractVerticle {

  @Override
  public void start(Future<Void> fut) {
    vertx
        .createHttpServer()
        .requestHandler(r -> {
          r.response().end("<h1>Hello from my first " +
              "Vert.x 3 application at EcuadorJUG</h1>");
        })
        .listen(8080, result -> {
          if (result.succeeded()) {
            fut.complete();
          } else {
            fut.fail(result.cause());
          }
        });
  }
}
```
- Listo tenemos la configuracion necesaria para la vertical de vertx Con un hello world.

- Antes vamos a generar un fat jar --- jar ejecutable. Por lo que usaremos el siguiente plugin de maven
	añadimos lo siguiente al pom.xml en la seccion de plugins.

```xml
	<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>2.3</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <transformers>
          <transformer
            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <manifestEntries>
              <Main-Class>io.vertx.core.Starter</Main-Class>
              <Main-Verticle>io.vertx.blog.first.MyFirstVerticle</Main-Verticle>
            </manifestEntries>
          </transformer>
        </transformers>
        <artifactSet/>
        <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-fat.jar</outputFile>
      </configuration>
    </execution>
  </executions>
</plugin>
```


- Nos regresamos hasta el directorio donde se encuentra el pom.xml y ejecutamos el siguiente comando de maven:

```bash
mvn clean install
```

- Ya nos genero el jar que lo ejecutamos con lo siguiente:

```bash
java -jar target/my-first-app-1.0-SNAPSHOT-fat.jar
```

- Abrimos un browser en la direccion:  

```
http://localhost:8080/

```
ctrl-c -> bajamos la aplicacion.



### DOCKER

- Vamos a realizar la misma aplicacion con docker usando maven.



- Abrimos el pom.xml 
- añadimos el siguiente dependencia de fabric8

```xml
 <plugin>
        <groupId>io.fabric8</groupId>
        <artifactId>fabric8-maven-plugin</artifactId>
        <version>2.2.140</version>
      </plugin>
      
```
  
- Añadimos el siguiente plugin justo debajo del tag build que se encarga de generar automáticamente la imagen de docker configuramos el puerto 8989 al 8080.
- Le damos el nombre de la vertical que queremos iniciar.
- Damos permisos de lectura escritura al directorio interno de la imagen.

```xml
 <pluginManagement>
      <plugins>
        <plugin>
          <groupId>io.fabric8</groupId>
          <artifactId>docker-maven-plugin</artifactId>
          <version>0.15.9</version>
          <configuration>
            <images>
              <image>
                <name>${docker.image}</name>
                <build>
                  <from>vertx/vertx3</from>
                  <tags>
                    <tag>${project.version}</tag>
                  </tags>
                  <ports>
                  <port>8080</port>
                  </ports>
                  <cmd>
                    <exec>
                      <arg>vertx</arg>
                      <arg>run</arg>
                      <arg>${verticle.name}</arg>
                      <arg>-cp</arg>
                      <arg>/usr/verticles/${project.artifactId}-${project.version}.jar</arg>
                    </exec>
                  </cmd>
                  <runCmds>
                    <runcmd>chmod -R 777 /usr/verticles</runcmd>
                    <runcmd>chmod -R 777 /usr/verticles/*</runcmd>
                  </runCmds>
                  <assembly>
                    <basedir>/</basedir>
                    <inline>
                      <files>
                        <file>
                          <source>${project.build.directory}/${project.artifactId}-${project.version}.jar</source>
                          <outputDirectory>/usr/verticles/</outputDirectory>
                          <fileMode>0755</fileMode>
                        </file>
                      </files>
                    </inline>
                  </assembly>
                </build>
                <run> 
         <ports> 
           <port>8989:8080</port>
         </ports>
       </run>
              </image>
            </images>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>

```

- copiamos los siguientes properties de maven:

```xml
<properties>
    <verticle.name>io.vertx.blog.first.MyFirstVerticle</verticle.name>
    <docker.image>vertx/vertx3-example-fabric8</docker.image>
  </properties>
```

- guardamos y corremos el siguiente comando para generar la imagen:

```bash
mvn docker:build
```

- luego corremos para correr la imagen de docker:

```bash
mvn docker:start
```

- Corremos el comando para verificar que la imagen esta arriba y con el mapeo de los puertos.
```bash
docker ps
```

- Corremos en el explorador en la siguiente url:

```
http://localhost:8989/

```
- Si necesitamos parar el contenedor de docker corremos el siguiente comando. Esto nos borra el contenedor luego de parar la ejecución del mismo.

```bash
mvn docker:stop
```


### LINKS

Para referencias y mas información:

<http://vertx.io/blog/my-first-vert-x-3-application/>
<http://vertx.io/docs/vertx-docker/>
<https://dmp.fabric8.io>
