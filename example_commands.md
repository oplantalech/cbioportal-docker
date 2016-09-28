### Importing data ###

Use this command to validate a dataset in the folder `./study-dir`, connecting
to the web API of the container `cbioportal-container`, and import it into the
database configured in the image, saving an html report of the validation to
`~/Desktop/report.html`.  Note that the paths given to the `-v` option must be
absolute paths.

```shell
docker run --rm --net cbio-net \
    -v "$PWD"/study-dir:/study:ro \
    -v "$HOME"/Desktop:/outdir \
    cbioportal-image \
    metaImport.py --jar_path=/usr/local/tomcat/webapps/cbioportal/WEB-INF/lib/core-1.2.5.jar -u http://cbioportal-container:8080/cbioportal -s /study --html=/outdir/report.html
```

### Running cBioPortal code from a local folder ###

TODO: document building images based on a different online git
branch or a local folder, and using volumes to overwrite frontend
resources on the fly

### Inspecting or adjusting the database ###

If you have mapped a port on the host to port 3306 of the container
running the MySQL database, using an option such as `-p 8306:3306`,
you can connect to this port (localhost:8306) using [MySQL
Workbench](https://www.mysql.com/products/workbench/) or another
MySQL client.  Alternatively, you can connect a command-line client
to the container (named `cbioDB` in this example) via the `--net`
option with the following command:

```shell
docker run -it --rm \
    --net=cbio-net \
    -e MYSQL_HOST=cbioDB \
    -e MYSQL_USER=cbio \
    -e MYSQL_PASSWORD=P@ssword1 \
    -e MYSQL_DATABASE=cbioportal \
    mysql \
    sh -c 'mysql -h"$MYSQL_HOST" -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE"'
```

### Updating the database schema ###

If the portal tells you that your database schema is outdated, you
can update it to match the cBioPortal version in the image by running
the following command. Note that this will most likely make your
database irreversibly incompatible with older versions of the portal
code.

```shell
docker run --rm -it --net cbio-net \
    cbioportal-image \
    migrate_db.py -p /cbioportal/src/main/resources/portal.properties -s /cbioportal/core/src/main/resources/db/migration.sql
```
