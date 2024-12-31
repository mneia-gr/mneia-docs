---
title: Install MusicBrainz Server on Ubuntu 24.04
layout: page
---

Follow these steps to install the [MusicBrainz Server](https://github.com/metabrainz/musicbrainz-server/) on a system
running Ubuntu 24.04. Much of the content is copied from the 
[MusicBrainz Server installation instructions](https://github.com/metabrainz/musicbrainz-server/blob/master/INSTALL.md),
adjusted for style, and with the addition of some extra steps required to install the 
[Django MusicBrainz Connector](https://github.com/mneia-gr/django-musicbrainz-connector).

The operating system is Ubuntu 24.04. No other system settings or configuration are required before starting this guide.

1.  Perl 5.38 or newer is required. This is already installed on Ubuntu 24.04:

    ```sh
    $ perl --version

    This is perl 5, version 38, subversion 2 (v5.38.2) [...]
    ```

2.  PostgreSQL version 16 or newer is required, which is the default on Ubuntu 24.04:

    ```sh
    $ apt info postgresql

    Package: postgresql
    Version: 16+257build1.1
    ```

    Installation:

    ```sh
    $ sudo apt install postgresql-16
    $ sudo apt install postgresql-contrib-16
    $ sudo apt install postgresql-server-dev-16
    ```

3.  Install Git:

    ```sh
    $ sudo apt install git
    ```

4.  Install Redis server:

    ```sh
    $ sudo apt install redis-server
    ```

5.  Node version 20.9.0 or newer is required, as well as a modern version of Yarn. Node 20 comes as a snap package on
    Ubuntu 24.04, while Yarn can be installed with `corepack`:

    ```sh
    $ sudo snap install node --classic
    $ sudo npm install --global corepack
    $ sudo corepack enable
    ```

    Confirm:

    ```sh
    $ node --version

    v20.18.1
    ```

6.  Install development tools:

    ```sh
    $ sudo apt install build-essential
    ```

7.  Install script dependencies:

    ```sh
    $ sudo apt install moreutils
    ```

8.  Clone the MusicBrainz Server repository:

    ```sh
    $ git clone --recursive git@github.com:metabrainz/musicbrainz-server.git
    $ cd musicbrainz-server
    ```

9.  To configure the server, copy the sample configuration:

    ```sh
    $ cp lib/DBDefs.pm.sample lib/DBDefs.pm
    ```

    Change the value of `WEB_SERVER` to `localhost:5000` and confirm:

    ```sh
    $ sed --in-place "s/sub WEB_SERVER .*/sub WEB_SERVER { 'localhost:5000' }/" lib/DBDefs.pm
    $ grep --quiet "^sub WEB_SERVER { 'localhost:5000' }" lib/DBDefs.pm || echo "Could not set WEB_SERVER"
    ```

    Change the value of `REPLICATION_TYPE` to `RT_MIRROR` and confirm:

    ```sh
    $ sed --in-place 's/# sub REPLICATION_TYPE { RT_STANDALONE }/sub REPLICATION_TYPE { RT_MIRROR }/' lib/DBDefs.pm
    $ grep --quiet '^sub REPLICATION_TYPE { RT_MIRROR }' lib/DBDefs.pm || echo 'Could not set REPLICATION_TYPE'
    ```

    Change the value of `DB_STAGING_SERVER` to 0 and confirm:

    ```sh
    $ sed --in-place 's/# sub DB_STAGING_SERVER { 1 }/sub DB_STAGING_SERVER { 0 }/' lib/DBDefs.pm
    $ grep --quiet 'sub DB_STAGING_SERVER { 0 }' lib/DBDefs.pm || echo 'Could not set DB_STAGING_SERVER'
    ```

    To keep the database replica up to date, you need a replication access token. Read the instructions at
    [Live Data Feed](https://musicbrainz.org/doc/Live_Data_Feed) for how to obtain a key, and set the value of
    `REPLICATION_ACCESS_TOKEN` in `lib/DBDefs.pm` to that key. For example:

    ```sh
    $ sed --in-place "s/# sub REPLICATION_ACCESS_TOKEN { '' }/sub REPLICATION_ACCESS_TOKEN { '1234' }/" lib/DBDefs.pm
    $ grep --quiet "sub REPLICATION_ACCESS_TOKEN { '1234' }" lib/DBDefs.pm || echo 'Could not set REPLICATION_ACCESS_TOKEN'
    ```

    Overall, this is the `diff` between the sample `lib/DBDefs.pm.sample` and the customised `lib/DBDefs.pm`:

    ```diff
    113c113
    < # sub REPLICATION_TYPE { RT_STANDALONE }
    ---
    > sub REPLICATION_TYPE { RT_MIRROR }
    120c120
    < # sub REPLICATION_ACCESS_TOKEN { '' }
    ---
    > sub REPLICATION_ACCESS_TOKEN { '1234' }
    149c149
    < sub WEB_SERVER                { 'www.musicbrainz.example.com' }
    ---
    > sub WEB_SERVER { 'localhost:5000' }
    190c190
    < # sub DB_STAGING_SERVER { 1 }
    ---
    > sub DB_STAGING_SERVER { 0 }

    ```

10. Install Perl prerequisites:

    ```sh
    $ sudo apt install libdb-dev
    $ sudo apt install libexpat1-dev
    $ sudo apt install libicu-dev
    $ sudo apt install liblocal-lib-perl
    $ sudo apt install libpq-dev
    $ sudo apt install libxml2
    $ sudo apt install libxml2-dev
    $ sudo apt install cpanminus
    $ sudo apt install pkg-config
    ```

11. Set up a local Perl library, so that Perl modules are installed in the user's home directory. These instructions are
    a little different from the ones in the MusicBrainz Server installation steps, in that the Perl modules are
    installed in hidden directory `$HOME/.perl5` instead of `$HOME/perl5`:

    ```sh
    $ mkdir $HOME/.perl5
    $ eval $(perl -I $HOME/.perl5/lib/perl5 -Mlocal::lib=$HOME/.perl5)
    $ echo 'eval $(perl -I $HOME/.perl5/lib/perl5 -Mlocal::lib=$HOME/.perl5)' >> $HOME/.bashrc
    $ echo 'export MANPATH=$HOME/.perl5/man:$MANPATH' >> $HOME/.bashrc
    $ source $HOME/.bashrc
    ```

    These instructions were adapted from the ones in
    [Setting up a local perl library](https://gwcbi.github.io/HPC/perl.html).

12. Install Perl dependencies:

    ```sh
    $ cpanm --installdeps --notest .
    ```

13. Set up translations. This step is out of order, compared with the MusicBrainz installation guide, because of bug
    [MBS-13294](https://tickets.metabrainz.org/browse/MBS-13294) in the MusicBrainz server:

    ```sh
    $ sudo apt install gettext
    $ cd po/
    $ make install
    $ cd ..
    ```

14. Build static resources, and also disable Yarn telemetry:

    ```sh
    $ ./script/compile_resources.sh
    $ yarn config set --home enableTelemetry 0
    ```

15. Configure the database. First, keep a backup copy of the original configuration files:

    ```sh
    $ sudo cp /etc/postgresql/16/main/pg_hba.conf /etc/postgresql/16/main/pg_hba.conf-BAK
    $ sudo cp /etc/postgresql/16/main/pg_ident.conf /etc/postgresql/16/main/pg_ident.conf-BAK
    ```

    Add this line as the **first** line in `pg_hba.conf`. Note that this is not a very secure option because any user
    with access to the system is automatically trusted with access to the database. It would be better to have more
    granular access, but this is good enough for a development setup:

    ```
    local   all    all    trust
    ```

    Add an entry for the MusicBrainz Server for client authentication:

    ```sh
    $ echo 'local musicbrainz_db musicbrainz ident map=mb_map' | sudo tee --append /etc/postgresql/16/main/pg_hba.conf
    ```

    Add a mapping for the user that runs your webserver. The default in Ubuntu is usually `www-user`, I'm using a user
    named `m`:

    ```sh
    $ echo 'mb_map m musicbrainz' | sudo tee --append /etc/postgresql/16/main/pg_ident.conf
    ```

    Finally, restart the PostgreSQL service:

    ```sh
    $ sudo systemctl restart postgresql.service
    ```

16. It's now time to populate the database from the data dumps offered by MusicBrainz. First, get a couple of useful
    tools:

    ```sh
    $ sudo apt install curl bzip2
    ```

    Get the latest data dump version and confirm. Note that the value of `$latest_dump` will be different, and that I'm
    running this in a subdirectory of `$HOME/Downloads`:

    ```sh
    $ mkdir -p $HOME/Downloads/musicbrainz-dump/
    $ cd $HOME/Downloads/musicbrainz-dump/
    $ latest_dump=$(curl https://data.metabrainz.org/pub/musicbrainz/data/fullexport/LATEST)
    $ echo $latest_dump

    20241228-001654
    ```

    Here, I'm creating an array with the names of all the dump files:
    
    ```sh
    $ declare -a dumps=()
    $ dumps+=("mbdump-cdstubs")
    $ dumps+=("mbdump-cover-art-archive")
    $ dumps+=("mbdump-derived")
    $ dumps+=("mbdump-documentation")
    $ dumps+=("mbdump-edit")
    $ dumps+=("mbdump-editor")
    $ dumps+=("mbdump-event-art-archive")
    $ dumps+=("mbdump-stats")
    $ dumps+=("mbdump-wikidocs")
    $ dumps+=("mbdump")
    ```

    Then, downloading them one by one:

    ```sh
    $ for dump in ${dumps[@]}; do wget "https://data.metabrainz.org/pub/musicbrainz/data/fullexport/$latest_dump/$dump.tar.bz2"; done
    ```
    
    At the time of writing this documentation in December 2024, the sizes of the downloaded files are:

    | File                               | Size |
    |------------------------------------|-----:|
    | `mbdump-cdstubs.tar.bz2`           |  64M |
    | `mbdump-cover-art-archive.tar.bz2` | 121M |
    | `mbdump-derived.tar.bz2`           | 401M |
    | `mbdump-documentation.tar.bz2`     |  25K |
    | `mbdump-editor.tar.bz2`            |  95M |
    | `mbdump-edit.tar.bz2`              |  12G |
    | `mbdump-event-art-archive.tar.bz2` | 115K |
    | `mbdump-stats.tar.bz2`             |  99M |
    | `mbdump-wikidocs.tar.bz2`          | 7.2K |
    | `mbdump.tar.bz2`                   | 5.5G |

17. Get the MD5 checksums of the files and verify they match:

    ```sh
    $ wget "https://data.metabrainz.org/pub/musicbrainz/data/fullexport/$latest_dump/MD5SUMS"
    $ md5sum -c MD5SUMS || echo 'MD5 Check Failed'
    ```

18. Change back into the directory where you cloned the `musicbrainz-server` repository:

    ```sh
    $ cd musicbrainz-server
    ```

    Start loading the data from the dumps into the database. Note that this takes hours. The amount of time will differ
    depending on the host's performance, but the last time I ran this it took about 3 hours:

    ```sh
    $ ./admin/InitDb.pl --createdb --import $HOME/Downloads/musicbrainz-dump/mbdump*.tar.bz2 --echo
    ```

    Build materialized tables. This took about 8 minutes:

    ```sh
    $ ./admin/BuildMaterializedTables --database=MAINTENANCE all
    ```

19. You can now start the server:

    ```sh
    plackup -Ilib -r
    ```

    You can visit the website running locally at `http://localhost:5000/`, or whichever location you configured in the
    value of `WEB_SERVER` in `lib/DBDefs.pm`.

20. Optionally, you can install [pgAdmin](https://www.pgadmin.org/) so that you can inspect the database in a GUI. There
    are instructions for its installation on that website, which we won't copy here.

21. Finally, your database is up to date with the MusicBrainz database, up to the point in time at which the data was
    dumped. For example, if the dumps that you downloaded are 2 days old, your local database will be 2 days out of date
    compared to the upstream database of MusicBrainz. To update the local mirror run:

    ```sh
    ./admin/cron/mirror.sh
    ```

    You can add this command to a cron job, to keep your local database up to date. To observe the progress of this
    operation, you can `tail` the `mirror.log` log file.

### Steps for Django MusicBrainz Connector ###

To work with the Django MusicBrainz Connector, you'll need a user with read-only privileges on the MusicBrainz database.
These instructions exist in the [README](https://github.com/mneia-gr/django-musicbrainz-connector/blob/main/README.md)
file of the connector, and copied here for completeness.

1.  Open a PostgreSQL shell with:

    ```sh
    $ sudo su postgres
    $ psql
    ```

2.  Connect to the `musicbrainz_db` database and create a user with read-only privileges:

    ```sql
    \c musicbrainz_db
    CREATE USER django_musicbrainz_connector WITH PASSWORD 'sUp3rSecr3t';
    GRANT CONNECT ON DATABASE musicbrainz_db TO django_musicbrainz_connector;
    GRANT USAGE ON SCHEMA musicbrainz TO django_musicbrainz_connector;
    GRANT SELECT ON ALL TABLES IN SCHEMA musicbrainz TO django_musicbrainz_connector;
    ALTER USER django_musicbrainz_connector SET SEARCH_PATH TO musicbrainz;
    ```

    You can confirm this configuration with something like:

    ```sql
    SELECT grantee, privilege_type
    FROM information_schema.role_table_grants
    WHERE table_name='area_type';
    ```

    The output should include the user you just created:

    ```
              grantee            | privilege_type
    ------------------------------+----------------
    musicbrainz                  | INSERT
    musicbrainz                  | SELECT
    musicbrainz                  | UPDATE
    musicbrainz                  | DELETE
    musicbrainz                  | TRUNCATE
    musicbrainz                  | REFERENCES
    musicbrainz                  | TRIGGER
    django_musicbrainz_connector | SELECT
    ```

    Also, you can test the connection to the database from your shell:

    ```sh
    psql --dbname musicbrainz_db --username django_musicbrainz_connector
    SELECT * FROM musicbrainz.area_type;
    ```