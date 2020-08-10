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

## Set up storage

Here we'll set up storage so that some of the data from the container can be persisted on the NAS itself. This will allow us to update the image and not have to worry about having to re-create pictures or other files.

1. If you already have a user dedicated to allowing containers access to files on the NAS, skip this section. Otherwise, create a folder on your NAS where you'll store the files from Monica.
1. Using Synology's **Control Panel**, create a user and a group. These will be used exclusively so that the container can access files on the NAS so the only permissions they need are read/write on the folder you just created. Go ahead and set up the permissions.
1. Log into the NAS using SSH (Do this with your regular user, not the dedicated one). Then type `id docker_user` replacing *"docker_user"* with the user you just created; make a note of the `uid` and `gid` (You'll need them later).

## Set up Monica container

1. Open the **Docker** package in **Package Center**.
1. In the **Registry** tab, search for *Monica* and download the image `monicahq/monicahq`.
1. Next, you'll either need create the container. You can achieve this by importing the [provided JSON file](files/monica.json) or by adding one from the **Image** tab in docker.
1. Before launching the container, you'll need to edit the values of some of the environmental variables. In the **Container** tab right click the *Monica* container and *Edit*, then head to the **Environment** tab. Alternatively edit the JSON file before importing.
   * **Variables starting with `MAIL_`:** If you want to receive e-mail reminders, change those values to suit your e-mail provider.
   * **`HASH_SALT`:** Change this to a random string of your choice.
   * **`APP_ENV`:** This value must be `local`, otherwise you won't be able to connect.
   * **Variables starting with `DB_`:** Change these to the values you used earlier for the DB setup.
   * **`APP_DEBUG`:** This is set to `false` in the provided file but might be worth flipping to `true` if you're doing some debugging.
   * **`PUID` and `PGID`:** Match these to the corresponding ones of the Synology user you're using to manage storage for the container.
1. You may also want to modify the port settings, so that the an unused port on the NAS is mapped to the container's port 80.
1. Make sure to edit the Volume settings to reflect your own. Select the folder you created to store Monica's files and set `/var/www/monica/storage` as the mount path.
1. Start the container, then right click on it and go into *Details*. Go to the terminal tab. You should see the terminal output while Monica sets up the DB. This step will take a while (In my case it took almost 20 mins). The step that took the longest was:
   ```
   ✓ Performing migrations                                                        
   '/usr/bin/php7' 'artisan' migrate --force
   ```
1. After set up is done, open Monica on your browser.

#### Debugging
> :beetle: **To test your e-mail setup, open up a `sh` terminal on the container and run** `php artisan monica:test-email`
