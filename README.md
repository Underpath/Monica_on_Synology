# Monica on Synology

A guide to instal [Monica](https://github.com/monicahq/monica) on a Synology NAS.

## Install MariaDB on Synology

1. Go to the **Synology DiskStation Manager (DSM)**. By default it's at `https://<NAS ip>:5000/`.
1. Open the **Package Center**.
1. At the top in the search bar type *MariaDB*.
1. Hit install on the **MariaDB** package. You'll be required to provide a password for the root user of the DB.
1. After installation is complete, open the **MariaDB** package in the **Package Center**.
1. Make sure the `Enable TCP/IP connection` option is enabled. Change the port if you want. Hit apply.

## Set up DB

1. Open up an SSH session to the NAS.
1. Connect to the Database locally using the password you provided earlier:
    ```sh
    /usr/local/mariadb10/bin/mysql -u root -p
    ```
1. Create the DB and user that Monica will use. You'll need DB name, DB user and password later on.
    ```sql
    CREATE DATABASE monica_db;
    GRANT ALL PRIVILEGES ON monica_db.* TO 'monica_user'@'%' IDENTIFIED BY 'newpassword';
    FLUSH PRIVILEGES;
    ```
1. If you want to restrict logins to from a certain IP replace `%` with the desired IP.

#### Debugging
> :beetle: **If you need to debug MariaDB issues, the error log is available at** `/var/packages/MariaDB10/target/mysql/io.err`

## Set up Monica container

1. Open the **Docker** package in the **Package Center**.
1. In the **Registry** tab, search for Monica and download the image `monicahq/monicahq`.
