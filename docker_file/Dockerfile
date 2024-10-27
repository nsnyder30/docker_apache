FROM almalinux:latest
COPY build_files/launcher.sh /usr/bin/launcher.sh
COPY build_files/connections.ini /etc/connections.ini
COPY build_files/odbc.ini /etc/odbc.ini
COPY build_files/tnsnames.ora /etc/tnsnames.ora
WORKDIR /app

# gcc-c++ make autoconf automake libtool: Requred to update to PHP 8.1
# PHP 8.1 required to install sqlsrv and pdo_sqlsrv packages, which are needed to connect to SQL Server data sources in Alma Linux
RUN curl https://packages.microsoft.com/config/rhel/9/prod.repo -o /etc/yum.repos.d/mssql-release.repo || echo "ERROR: Could not download Microsfot SQL Server ODBC Drivers. skipping" \
    && curl -O --output-dir /tmp https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient21/x86_64/getPackage/oracle-instantclient-basic-21.3.0.0.0-1.x86_64.rpm || echo "ERROR: Could not download Oracle Instantclient Basic Repo. Skipping" \
    && curl -O --output-dir /tmp https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient21/x86_64/getPackage/oracle-instantclient-odbc-21.3.0.0.0-1.x86_64.rpm || echo "ERROR: Could not download Oracle Instantclient ODBC Repo. Skipping" \
    && curl -O --output-dir /tmp https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient21/x86_64/getPackage/oracle-instantclient-sqlplus-21.3.0.0.0-1.x86_64.rpm || echo "ERROR: Could not download Oracle Instantclient SQL Plus Repo. Skipping" \
    && curl -O --output-dir /tmp https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient21/x86_64/getPackage/oracle-instantclient-devel-21.3.0.0.0-1.x86_64.rpm || echo "ERROR: Could not download Oracle Instantclient SQL Plus Repo. Skipping" \
    && curl -O --output-dir /tmp https://www.php.net/distributions/php-8.1.0.tar.gz || echo "ERROR: Could not download PHP source code. Skipping"

RUN dnf clean all && dnf makecache
RUN tar -xvzf /tmp/php-8.1.0.tar.gz -C /tmp/. || echo "ERROR: failed to extract php source code. skipping" \
    && dnf install epel-release -y || echo "ERROR: failed to install epel-release. skipping" \
    && dnf install wget -y || echo "ERROR: failed to install wget. skipping" \
    && dnf install php -y || echo "ERROR: failed to install php. skipping" \
    && dnf install java-11-openjdk-devel -y || "ERROR: failed to install java virtual machien. skipping" \
    && dnf install glibc libxcrypt-compat -y || echo "ERROR: failed to install glibc and libxcrypt prereqs for informix client SDK installation. skipping" \
    && dnf install gcc make autoconf automake libtool -y || echo "ERROR: failed to install development tools needed to upgrade to PHP 8.1. skipping" \
    && dnf install php-devel php-pear https://rpms.remirepo.net/enterprise/remi-release-9.rpm -y || echo "ERROR: failed to install PHP development tools. skipping" \
    && dnf install php php-cli php-common php-odbc php-devel libaio-devel -y || echo "ERROR: failed to install php components needed to upgrade to PHP 8.1 skipping" \
    && dnf module reset php -y || echo "ERROR: failed to reset PHP module. skipping" \
    && dnf module enable php:8.1 -y || echo "ERROR: failed to upgrade to PHP 8.1. skipping" \
    && ACCEPT_EULA=Y dnf install msodbcsql17 -y || echo "ERROR: failed to install Microsoft SQL Server ODBC Drivers. skipping" \
    && dnf install openssh-server openssh-clients -y || echo "ERROR: failed to install openssh. skipping" \
    && dnf install httpd -y || echo "ERROR: failed to install httpd. skipping" \
    && dnf install vim -y || echo "ERROR: failed to install vim. skipping" \
    && dnf install git -y || echo "ERROR: failed to install git. skipping" \
    && dnf install sudo -y || echo "ERROR: failed to install sudo. skipping" \
    && dnf install net-tools -y || echo "ERROR: failed to install net-tools. skipping" \
    && dnf install unixODBC -y || echo "ERROR: failed to install unixODBC. skipping" \
    && dnf --enablerepo crb install unixODBC-devel -y || echo "ERROR: failed to install unixODBC-devel. skipping" \
    && ACCEPT_EULA=Y dnf install /tmp/oracle-instantclient-basic-21.3.0.0.0-1.x86_64.rpm -y || echo "ERROR: failed to install Oracle Instantclient Basic. skipping" \
    && ACCEPT_EULA=Y dnf install /tmp/oracle-instantclient-odbc-21.3.0.0.0-1.x86_64.rpm -y || echo "ERROR: failed to install Instantclient Oracle ODBC. skipping" \
    && ACCEPT_EULA=Y dnf install /tmp/oracle-instantclient-sqlplus-21.3.0.0.0-1.x86_64.rpm -y || echo "ERROR: failed to install Oracle Instantclient SQL Plus. skipping" \
    && ACCEPT_EULA=Y dnf install /tmp/oracle-instantclient-devel-21.3.0.0.0-1.x86_64.rpm -y || echo "ERROR: failed to install Oracle Instantclient SQL Plus. skipping" \
    && dnf install php-odbc -y || echo "ERROR: failed to install php-odbc. skipping" \
    && dnf install php-mysqlnd -y || echo "ERROR: failed to install php-mysqlnd -y" \
    && dnf install mysql-connector-odbc -y || echo "EREROR: failed to install mysql-connector-odbc. skipping" \
    && dnf install mariadb-connector-odbc -y || echo "ERROR: failed to install mariadb-connector-odbc. skipping" \
    && dnf install initscripts -y || echo "ERROR: failed to install initscripts. skipping" \ 
    && dnf module enable php:8.1 -y || echo "ERROR: failed to enable PHP 8.1. skipping"
    
# Need to create symoblic link to libltdl library to enable installing sqlsrv and pdo_sqlsrv packages
RUN ln -s /usr/lib64/libltdl.so.7 /usr/lib64/libltdl.so \
    && pecl install sqlsrv \
    && pecl install pdo_sqlsrv

WORKDIR /tmp/php-8.1.0/ext/pdo_oci
RUN phpize || echo "ERROR: phpize failed for PDO_OCI. skipping" \
    && ./configure --with-pdo-oci=instantclient,/usr/lib/oracle/21/client64/lib,21.3 || echo "ERROR: configure failed for PDO_OCI. skipping" \
    && make || echo "ERROR: make failed for PDO_OCI. skipping" \
    && make install || echo "ERROR: make install failed for PDO_OCI. skipping"

RUN mkdir /run/php-fpm
RUN sed -i 's/display_errors = Off/display_errors = On/g' /etc/php.ini
# Alternative method for granting PHP access to system environment variables.
# This passes all system environment variables to the $_ENV superglobal at startup.
# 
# RUN sed -i 's/variables_order =.*/variables_order = "EGPCS"/g' /etc/php.ini
RUN sed -i 's/;clear_env = no/clear_env = no/g' /etc/php-fpm.d/www.conf \
    && sed -i 's/max_execution_time =.*/max_execution_time = 30000/g' /etc/php.ini \
    && sed -i 's/memory_limit =.*/memory_limit = 512M/g' /etc/php.ini \
    && echo 'extension=pdo.so' >> /etc/php.ini \
    && echo 'extension=sqlsrv' >> /etc/php.ini \
    && echo 'extension=pdo_sqlsrv' >> /etc/php.ini \ 
    && echo 'extension=pdo_oci.so' >> /etc/php.ini || echo "ERROR: failed to update php.ini" \
    && echo 'PassEnv MY_ENV_VAR' >> /etc/httpd/conf/httpd.conf || echo "ERROR: failed to modify httpd.conf" \
    && rm -rf /tmp/* || echo "ERROR: Failed to empty tmp directory. skipping"

RUN chmod +x /usr/bin/launcher.sh
