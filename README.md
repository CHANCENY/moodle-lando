# How to Set Up Moodle on Lando and Configure `config.php`

Moodle is a powerful learning management system, and Lando provides an excellent local development environment for it. 
In this guide, we will walk through the steps to set up Moodle on Lando, 
configure its `config.php` file, and ensure that the `wwwroot`, `dataroot`, and database settings are properly configured.

---

## **1. Install Lando**
If you haven't installed Lando yet, download and install it from [Lando's official website](https://lando.dev/).

---

## **2. Set Up Lando for Moodle**
Create a project directory and navigate to it:

```bash
mkdir moodle-project && cd moodle-project
```

Then, create a `.lando.yml` file with the following content:

```yaml
name: moodle-project
recipe: lamp
config:
  webroot: moodle
  php: 8.4

services:
  database:
    type: mariadb:10.6
    portforward: true

  phpmyadmin:
    type: phpmyadmin
    hosts:
      - database
    portforward: true

  appserver:
    run_as_root:
      # Install required dependencies
      - apt-get update && apt-get install -y cron nano
      - service cron start

    run:
      # Set up Moodle cron job
      - (crontab -l 2>/dev/null | grep -q "admin/cli/cron.php" || (crontab -l 2>/dev/null; echo "* * * * * /usr/local/bin/php /app/moodle/admin/cli/cron.php >> /app/moodle/cron.log 2>&1") | crontab -)
```

---

## **3. Download and Install Moodle**
Clone the Moodle repository inside the `moodle-project` directory:

```bash
git clone https://github.com/moodle/moodle.git moodle
```
Or

Download the latest stable moodle [Moodle official site](https://download.moodle.org/releases/latest/)

Start Lando:

```bash
lando start
```

Once Lando starts, enter the Lando app server:

```bash
lando ssh
```

Inside the container, move to the Moodle directory and install dependencies:

```bash
cd /app/moodle
composer install
```

---

## **4. Configure Moodle's `config.php` File**
Moodle requires a `config.php` file for setup. Copy the sample config:

```bash
cp moodle/config-dist.php moodle/config.php
```

Then, edit the `config.php` file:

```php
<?php
// Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'database'; // Lando database service name
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodleuser';
$CFG->dbpass    = 'moodlepassword';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
    'dbpersist' => 0,
    'dbport' => '',
    'dbsocket' => '',
    'dbcollation' => 'utf8mb4_unicode_ci',
);

$CFG->wwwroot   = 'https://moodle-project.lndo.site/'; // Lando proxy URL
$CFG->dataroot  = '/app/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 02777;
require_once(__DIR__ . '/lib/setup.php');
```

---

## **5. Create Moodle Data Directory**
Moodle requires a `moodledata` directory for file storage. Create it and set permissions:

```bash
mkdir moodledata
chmod -R 777 moodledata
```
---

## **6. Install Moodle**
Now, access Moodle in your browser:

```bash
https://moodle-project.lndo.site/
```

Follow the installation process, ensuring:
- Database settings match those in `config.php`
- Web address is `https://moodle-project.lndo.site/`
- Data directory is `/app/moodledata`

After installation, log in with your admin credentials.

---

## **7. Running Moodle Cron Jobs**
Ensure the cron job is running by checking logs:

```bash
cat moodle/cron.log
```

You can manually run it:

```bash
lando ssh
/usr/local/bin/php /app/moodle/admin/cli/cron.php
```

---

## **Conclusion**
Congratulations! 🎉 You now have Moodle running on Lando with a properly configured `config.php`. 
You’ve also set up automatic cron jobs to keep Moodle running smoothly. Enjoy your development environment! 🚀

