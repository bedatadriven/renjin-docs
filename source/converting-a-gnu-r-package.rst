.. highlight:: none

Converting a GNU R Package
==========================

As mentioned earlier, there are some differences between the directory layout of
a GNU R package and a Renjin package. The biggest problem however, is the use of
C, C++ and Fortran code in GNU R packages. There is no simple way to convert such code
into java or but fortunately the gcc-bridge, which is part of Renjin, can covert such code into byte code.

The gcc-bridge uses gcc 4.7 to compile c, c++ and fortran code into an intermediate form (gimple)
which is then translates into JVM bytecode.

The Renjin maven plugin supports such compilation but since gcc 4.7 is somewhat difficult to install,
the recommended way is to run it in a virtual machine e.g. with the help of `vagrant <https://www.vagrantup.com>`_
where you can run ubuntu 16.04 on which gcc 4.7 and other required tools are easy to install.

Note that many of the common GNU R packages have already been converted and are available from the
`bedatadriven nexus <https://nexus.bedatadriven.com/>`_

The process to convert a GNU R package to a Renjin package is as follows:

1. Download and unpack the GNU R package

2. Create a pom.xml file

3. Create a Vagrant file

4. Run ``mvn install`` from inside the vagrant vm

5. Use the package

2. Creating a pom.xml file
--------------------------

Use the pom.xml below as a template. You just need to replace the section

.. code-block:: xml

    <groupId>com.acme</groupId>
    <artifactId>my-package</artifactId>
    <version>1.0.0</version>

...with whatever is appropriate.

.. parsed-literal::

    <?xml version="1.0" encoding="UTF-8"?>
    <project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
       <!-- This pom file must have access to gcc 4.7 e.g. run inside a vagrant container -->
       <modelVersion>4.0.0</modelVersion>
       <groupId>com.acme</groupId>
       <artifactId>my-package</artifactId>
       <version>1.0.0</version>
       <properties>
          <renjin.version>\ |release|\ </renjin.version>
       </properties>
       <dependencies>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>methods</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>datasets</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>stats</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>grDevices</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>stats4</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>tools</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>utils</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>graphics</artifactId>
             <version>${renjin.version}</version>
          </dependency>
          <dependency>
             <groupId>org.renjin</groupId>
             <artifactId>compiler</artifactId>
             <version>${renjin.version}</version>
             <scope>provided</scope>
          </dependency>
       </dependencies>
       <repositories>
          <repository>
             <id>bedatadriven</id>
             <url>https://nexus.bedatadriven.com/content/groups/public/</url>
          </repository>
       </repositories>
       <pluginRepositories>
          <pluginRepository>
             <id>bedatadriven</id>
             <url>https://nexus.bedatadriven.com/content/groups/public/</url>
          </pluginRepository>
       </pluginRepositories>
       <build>
          <plugins>
             <plugin>
                <groupId>org.renjin</groupId>
                <artifactId>renjin-maven-plugin</artifactId>
                <version>${renjin.version}</version>
                <executions>
                   <execution>
                      <id>renjin-compile</id>
                      <phase>process-classes</phase>
                      <goals>
                         <goal>namespace-compile</goal>
                      </goals>
                      <configuration>
                         <sourceDirectory>${basedir}/R</sourceDirectory>
                         <dataDirectory>${basedir}/data</dataDirectory>
                         <defaultPackages>
                            <package>methods</package>
                            <package>stats</package>
                            <package>utils</package>
                            <package>grDevices</package>
                            <package>graphics</package>
                            <package>datasets</package>
                         </defaultPackages>
                      </configuration>
                   </execution>
                   <execution>
                      <id>renjin-test</id>
                      <phase>test</phase>
                      <goals>
                         <goal>test</goal>
                      </goals>
                      <configuration>
                         <timeoutInSeconds>30</timeoutInSeconds>
                         <testSourceDirectory>${basedir}/tests</testSourceDirectory>
                         <defaultPackages>
                            <package>methods</package>
                            <package>stats</package>
                            <package>utils</package>
                            <package>grDevices</package>
                            <package>graphics</package>
                            <package>datasets</package>
                         </defaultPackages>
                      </configuration>
                   </execution>
                   <execution>
                      <id>gnur-compile</id>
                      <phase>compile</phase>
                      <goals>
                         <goal>gnur-compile</goal>
                      </goals>
                   </execution>
                </executions>
             </plugin>
            <plugin>
              <!-- the gcc-bridge compiles into the src dir so we need to extend the
                clean target to get the compiled files clean out from there as well -->
              <artifactId>maven-clean-plugin</artifactId>
              <version>3.1.0</version>
              <configuration>
                <filesets>
                  <fileset>
                    <directory>src</directory>
                    <includes>
                      <include>\*\*/\*.so</include>
                      <include>\*\*/\*.a</include>
                      <include>\*\*/\*.gimple</include>
                      <include>\*\*/\*.o</include>
                    </includes>
                    <followSymlinks>false</followSymlinks>
                  </fileset>
                </filesets>
              </configuration>
            </plugin>
          </plugins>
       </build>
    </project>

3. Create a the vagrant config file (Vagrantfile)
-------------------------------------------------

Below is an example Vagrantfile that you can use without modification:

.. code-block:: ruby

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # Vagrantfile API/syntax version
    VAGRANTFILE_API_VERSION = "2"

    # Override host locale variable
    ENV["LC_ALL"] = "en_US.UTF-8"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

      config.vm.box = "ubuntu/xenial64"
      config.vm.provision :shell, inline: "apt-get update && apt-get install openjdk-8-jdk maven make gcc-4.7 gcc-4.7-plugin-dev gfortran-4.7 g++-4.7 gcc-4.7.multilib g++-4.7-multilib unzip libz-dev -y"
      config.vm.synced_folder ".", "/home/ubuntu/renjin"
      config.vm.synced_folder "~/.m2", "/home/vagrant/.m2"

      config.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 2
      end
    end


In configuration above, Vagrant configures a shared directory on the VirtualBox guest machine
that includes the Renjin repository, as well as mapping the .m2 dir to your ~/.m2 so that you can run ``mvn install``
inside vagrant and then use the package outside of the vagrant container.

4. Build the project
--------------------

.. code-block:: sh

    vagrant up
    vagrant ssh -c "cd /home/ubuntu/renjin && mvn -f pom.xml clean install"

5. Use the package
------------------

Add the following dependency to your pom.xml

.. code-block:: xml

    <dependency>
      <groupId>com.acme</groupId>
      <artifactId>my-package</artifactId>
      <version>1.0.0</version>
    </dependency>

and then in your R code you use it as you would use the GNU R package e.g.

.. code-block:: r

    library('com.acme:my-package')

    # call any functions from the package as usual

Examples
--------
`ctsmr <https://github.com/perNyfelt/renjinSamplesAndTests/tree/master/ctsmr>`_