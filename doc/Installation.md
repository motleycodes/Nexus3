Supported Versions

Sonatype fully supports versions of repository manager for one year after the release date. Older releases are supported on a best effort basis and the release dates are listed in our download archives. The terms of support are explained in section 3 of the End User License Agreement.
Host Operating System

Any Windows, Linux or Macintosh operating system that can run a supported Java version will work. Other operating systems may work, but they are not tested by Sonatype.

The most widely used operating system for Nexus Repository Manager (NXRM) is Linux and therefore customers should consider it the best tested platform.
Dedicated Operating System User Account

Unless you are just testing the repository manager or running it only for personal use, a dedicated operating system user account is strongly recommended to run each unique process on a given host.

The NXRM process user is typically named 'nexus' and must be able to create a valid shell.

Important

As a security precaution, do not run Nexus Repository Manager 3 as the root user.
Adequate File Handle Limits

NXRM3 will most likely want to consume more file handles than the per user default value allowed by your Linux or OSX operating system.

Running out of file descriptors can be disastrous and will most probably lead to data loss. Make sure to increase the limit on the number of open files descriptors for the user running Nexus Repository Manager permanently to 65,536 or higher prior to starting.

See https://issues.sonatype.org/browse/NEXUS-12041 for additional background.
Linux

On most Linux systems, persistent limits can be set for a particular user by editing the /etc/security/limits.conf file. To set the maximum number of open files for both soft and hard limits for the nexus user to 65536, add the following line to the /etc/security/limits.conf file, where "nexus" should be replaced with the user ID that is being used to run the repository manager:

nexus - nofile 65536

This change will only take effect the next time the nexus process user opens a new session. Which essentially means that you will need to restart NXRM.

On Ubuntu systems there is a caveat: Ubuntu ignores the /etc/security/limits.conf file for processes started by init.d.

So if NXRM is started using init.d there, edit /etc/pam.d/common-session and uncomment the following line ( remove the hash # and space at the beginning of the line):

# session    required   pam_limits.so

For more information refer to your specific operating system documentation.

If you're using systemd to launch the server the above won't work. Instead, modify the configuration file to add a LimitNOFILE line:

[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target

Mac OSX

The method to modify the file descriptor limits on OSX has changed a few times over the years. Please note your OS X version and follow the appropriate instructions.

For OS X Yosemite (10.10) and newer

    Create the file: /Library/LaunchDaemons/limit.maxfiles.plist

    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
      <plist version="1.0">
        <dict>
          <key>Label</key>
            <string>limit.maxfiles</string>
          <key>ProgramArguments</key>
            <array>
              <string>launchctl</string>
              <string>limit</string>
              <string>maxfiles</string>
              <string>65536</string>
              <string>65536</string>
            </array>
          <key>RunAtLoad</key>
            <true/>
          <key>ServiceIPC</key>
            <false/>
        </dict>
      </plist>

    If this file already exists, then ensure the value is at least 65536 as shown.

    The file must be owned by root:wheel and have permissions -rw-r--r--. 

    sudo chmod 644 /Library/LaunchDaemons/limit.maxfiles.plist
    sudo chown root:wheel /Library/LaunchDaemons/limit.maxfiles.plist

    Reboot the operating system to activate the change.

    Add a new line to  $install-dir/bin/nexus.vmoptions  containing:

    -XX:-MaxFDLimit

    Restart NXRM to activate the change.

For OS X Lion (10.7) up to OS X Mavericks (10.9)

    Create and edit the system file /etc/launchd.conf using this command:

    sudo sh -c 'echo "limit maxfiles 65536 65536" >> /etc/launchd.conf'

    Reboot the operating system to activate the change.

    Add a new line to $install-dir/bin/nexus.vmoptions containing:

    -XX:-MaxFDLimit

    Restart NXRM to activate the change.

Windows

Windows operating systems do not need file handle limit adjustments.
Docker

The Nexus Repository Docker images are configured with adequate file limits. Some container platforms such as Amazon ECS will override the default limits. On these platforms it is recommended that the Docker image be run with the following flags:

--ulimit nofile=65536:65536


Java

Nexus Repository Manager requires a Java 8 Runtime Environment (JRE). The distributions for OSX and Windows include suitable runtime environments for the specific operating system. The distributions for Unix do not include the runtime environment. If you prefer to use an external runtime or use a Unix operating system, you can choose to install the full JDK or the JRE only. You can confirm the installed Java version with the java -version  command, for example:

$ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)

When multiple JDK or JRE versions are installed, you need to ensure the correct version is configured by running the above command as the operating system user that is used to run the repository manager.

In the event you have a non-standard location you need to update the configuration to specify a specific JDK or JRE installation path. To set the path for a specific Java location open the bin/nexus script and locate the line INSTALL4J_JAVA_HOME_OVERRIDE. Remove the hash and specify the location of your JDK/JRE:

INSTALL4J_JAVA_HOME_OVERRIDE=/usr/lib/jvm/openjdk-8

The startup script verifies the runtime environment by checking for the existence of the nested bin/java command  as well as major and minor version of the runtime to be the required 1.8. If the configured runtime is not suitable, it will proceed with a best effort to locate a suitable runtime configured on the path or via the JAVA_HOME environment variable. If successful, it will start up the repository manager with this JVM. This allows you to have a dedicated runtime environment for the repository manager installed that is not on the path and not used by other installed applications. Further, you can separate upgrades of the Java runtime used by the repository manager from upgrades of the runtime used by other applications.
CPU

Performance is primarily bounded by IO (disk and network) rather than CPU. Available CPUs will impact longer running operations and also the thread allocation algorithms of the web container.

Minimum CPUs: 4

Recommended CPUs: 8+
Memory
Configurable Memory Types

Visit the Configuring the Runtime Enviroment page to learn how to change the default memory settings.
JVM Heap Memory

Heap memory stores runtime application objects. A min ( -Xms ) and max ( -Xmx ) value must be specified and the values should be identical.

Increasing the heap memory larger than recommendations or setting the min and max values to be different is not recommended. This will create performance issues causing the operating system to thrash needlessly.

Unless you have evidence that a max heap of 4GB is consistently utilized or there are frequent lengthy garbage collection pauses that cannot be explained by software bugs, then do not set max heap size larger than 4GB.
JVM Direct Memory

Direct memory is allocated outside of and distinct from heap memory. A max value must be configured.
Host Physical Memory

The total memory allocated to the entire operating system or virtual hardware, commonly referred to as RAM.
Memory Requirements

The requirements assume there are no other significant memory hungry processes running on the same host.

	JVM Heap	JVM Direct	Host Physical/RAM
Minimum ( default ) 	2703MB	2703MB	8GB
Maximum	4GB	(host physical/RAM * 2/3) - JVM max heap	no limit
General Memory Guidelines

    minimum physical/RAM memory on the host 8GB
    minimum heap ( -Xms ) must equal set maximum heap ( -Xmx )
    minimum heap size 2703MB
    maximum heap size <= 4GB
    minimum direct memory ( -XX:MaxDirectMemorySize ) size 2703MB
    minimum unallocated host physical/RAM memory should be no less than 1/3 of total physical RAM to allow for virtual memory swap
    max heap + max direct memory <= host physical/RAM * 2/3

Instance Memory Sizing Profiles

These profiles help gauge the typical physical memory requirements needed for a dedicated server host running repository manager. Due to the inherent complexities of use cases, one size does not fit all and this should only be interpreted as a guideline.
Profile Use Case 	Physical/RAM Memory

small, personal

repositories < 20
total blobstore size < 20GB
single repository format type
	 8GB minimum

medium, team

repositories < 50
total blobstore size < 200GB
a few repository formats
	 16GB

large, enterprise

repositories > 50
total blobstore size > 200GB
diverse set of repository formats 
	 32GB+
Example Maximum Memory Configurations 
Physical/RAM Memory
	
Example Maximum Memory Configuration
8GB	

-Xms2703M
-Xmx2703M
-XX:MaxDirectMemorySize=2703M

12GB	

-Xms4G
-Xmx4G
-XX:MaxDirectMemorySize=4014M

16GB	

-Xms4G
-Xmx4G
-XX:MaxDirectMemorySize=6717M

32GB	

-Xms6G
-Xmx6G
-XX:MaxDirectMemorySize=15530M

64GB	

-Xms8G
-Xmx8G
-XX:+UseG1GC
-XX:MaxDirectMemorySize=35158M

Advanced Database Memory Tuning

Refer to another article which outlines additional memory tuning procedures.
Temporary Directory

The temporary directory at $data-dir/tmp must not be mounted with noexec or repository manager startup will fail with java.lang.UnsatisfiedLinkError  message of  failed to map segment from shared object: Operation not permitted .
Disk Space

Application Directory - The size of this directory varies slightly each release. It currently around 330 MB. It is normal to have multiple application directories installed on the same host over time as repository manager is upgraded.

Data Directory - On first start, repository manager creates the base files needed to operate. The bulk of disk space will be held by your deployed and proxied artifacts, as well as any search indexes. This is highly installation specific, and will be dependent on the repository formats used, the number of artifacts stored, the size of your teams and projects, etc.  It's best to plan for a lot though, formats like Docker and Maven can use very large amounts of storage (500Gb easily).  When available disk space drops below 4GB the database will switch to read-only mode.
File Systems

Nexus Repository stores multiple kinds of data, with two primary storage requirements:

    Embedded data (H2, OrientDB, Elastic Search) requires very responsive, fast storage, ideally local disk
    Blob storage (component binaries), which requires moderately responsive, high-capacity storage

File system selection should be made bearing both of these in mind.
File System	Embedded data	Blob Stores	Comment
XFS	Supported	Supported	This is a commonly used file system for locally attached storage.
NFS v4	Not Recommended**	Supported	Most common protocol for network attached storage among Nexus Repository deployments.
Amazon EBS	Supported	Supported	EBS is a viable choice for both embedded data and binary storage.
Amazon EFS	Unsupported	Not Recommended	EFS isn't sufficiently responsive for embedded data, and in our testing handles too few requests per second for binary storage.
Amazon S3	N/A	Supported	S3 semantics aren't applicable for embedded data, but S3 is popular for binary storage.
SMB, CIFS	Unsupported	Supported	Problems are common with SMB or CIFS-mounted devices for embedded data.
Azure Blob Storage	N/A	Supported	Available for blob storage from Nexus 3.30.0 Pro. For performance reasons, the Azure blob store should be in the same Azure region as the Nexus Repo installation.
Azure Files	Unsupported	Supported	Issues with file handles have been observed when accessing embedded data over SMB.
S3-Compatible	Unsupported	Some S3-compatible object stores do not support all the features required by Nexus Repository or have subtle compatibiity issues with the AWS Java SDK that NXRM uses. This includes providers such as Ceph S3.
NFS v3	Unsupported	Numerous customers have experienced inadequate performance with NFS v3.
GlusterFS	Unsupported	Split-brain problems and slow performance are common.
FUSE	Unsupported	FUSE based user-space filesystems are known to be unreliable for Nexus Repository.

** NFSv4.1 or higher can be used for the work directory in small lightly loaded installations, but we have found that it does not provide sufficient performance for anything larger.  In general it should be avoided for the work directory..
File System Optimization

We also have some optimization suggestions to use at your discretion.  Also consider the noatime option for your Nexus Repository work directory mounts and limit the symbolic links used as this will cause increased overhead whenever paths need to be resolved to an absolute file path.
Web Browser

Our general policy is to support the most recent modern browser version for your supported OS at time of NXRM release date.
Vendor	
Browser
	
Versions
Google	Chrome	latest at NXRM release
Mozilla	Firefox	latest and ESR at NXRM release
Apple	Safari	latest at NXRM release
Microsoft	Edge	latest at NXRM release
Microsoft	Internet Explorer	No longer supported