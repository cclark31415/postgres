This is meant to start up a Postgres instance on a Debian distro (specifically Mint 19) running in a virtual machine (VirtualBox).  I could not connect to the database using DBeaver and I'm a Docker n00b, so I did this docker-compose script to help me out.

This was also modified so the data would be persistent.

I still could not connect from a pgadmin4 container to this database no matter what I did, but DBeaver does everything I need, so I'll stick with it for now.  PGAdmin4 should work if I install it in the OS, so that's a fallback if I need it.

I created a service so this starts up on boot.  If you want to do this,  start out by creating a file called postgres-docker in the /etc/init.d folder with this script in it (you'll need to do it as `root` or use `sudo`):

```bash
#! /bin/sh
# /etc/init.d/blah
#


start() {
	cd <path to docker-compose.yml file>
	docker-compose up -d 
}

stop() {
	docker stop postgres_db_1
	docker rm postgres_db_1
}

case "$1" in 
    start)
       start
       ;;
    stop)
       stop
       ;;
    restart)
       stop
       start
       ;;
    *)
       echo "Usage: $0 {start|stop|restart}"
esac

exit 0

```

Make it executable
```console
sudo chmod 744 postgres-docker 
```

Create the service launcher
```console
cd /etc/rc5.d
sudo ln -s ../init.d/postgres-docker S20postgres-docker
sudo systemctl daemon-reload
```

You should be able to do `sudo service postgres-docker start` to kick it off

I also created an alias to easily jump into the psql command line:

```console
alias ps='docker run -it --rm --network container:postgres_db_1 postgres psql -h localhost -U postgres'
```
Just typing `ps` will get you into psql

If you want to create a new user, you can run this from the command line

```
docker run -it --rm --network container:postgres_db_1 postgres psql -h localhost -U postgres -c " \
        do \
        \$body\$ \
        declare \
          num_users integer; \
        begin \
          SELECT count(*) \
            into num_users \
          FROM pg_user \
          WHERE usename = 'bartSimpson'; \
        \
          IF num_users = 0 THEN \
              CREATE ROLE bartSimpson LOGIN PASSWORD 'AyCaramba'; \
          END IF; \
        end \
        \$body\$ \
        ;"
```
It executes a single psql command in the container