# Monica on Synology

A guide to install [Monica](https://github.com/monicahq/monica) on a Synology NAS.

## Install MariaDB on Synology

1. Go to the **Synology DiskStation Manager (DSM)**. By default it's at `https://<NAS_ip>:5000/`.
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
1. Create the DB and user that Monica will use. You'll need *DB name*, *DB user* and *password* later on.
    ```sql
    CREATE DATABASE monica_db;
    GRANT ALL PRIVILEGES ON monica_db.* TO 'monica_user'@'%' IDENTIFIED BY 'newpassword';
    FLUSH PRIVILEGES;
    ```
1. If you want restrict which IP that user can connect from, replace `%` with the desired IP.

#### Debugging
> :beetle: **If you need to debug MariaDB issues, the error log is available at** `/var/packages/MariaDB10/target/mysql/io.err`

## Set up Monica container

1. Open the **Docker** package in **Package Center**.
1. In the **Registry** tab, search for *Monica* and download the image `monicahq/monicahq`.
1. Next, you'll either need create the container. You can achieve this by importing the [provided JSON file](files/monica.json) or by adding one from the **Image** tab in docker.
1. Before launching the container, you'll need to edit the values of some of the environmental variables. In the **Container** tab right click the *Monica* container and *Edit*, then head to the **Environment** tab. Alternatively edit the JSON file before importing.
   * **Variables starting with `MAIL_`:** If you want to receive e-mail reminders, change those values to suit your e-mail provider.
   * **`HASH_SALT`:** Change this to a random string of your choice.
   * **`APP_ENV`:** This value must be `local`, otherwise you won't be able to connect.
   * **Variables starting with `DB_`:** Change these to the values you used earlier for the DB setup.
1. You may also want to modify the port settings, so that the an unused port on the NAS is mapped to the container's port 80.
1. Start the container, then right click on it and go into *Details*. Go to the terminal tab. You should see the terminal output while Monica sets up the DB. This step will take a while (In my case it took almost 20 mins). The step that took the longest was:
   ```
   ✓ Performing migrations                                                        
   '/usr/bin/php7' 'artisan' migrate --force
   ```
1. After set up is done, open Monica on your browser.

#### Debugging
> :beetle: **To test your e-mail setup, open up a `sh` terminal on the container and run** `php artisan monica:test-email`
