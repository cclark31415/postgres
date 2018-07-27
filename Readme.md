This is meant to start up a Postgres instance on a Debian distro (specifically Mint 19) running in a virtual machine (VirtualBox).  I could not connect to the database using DBeaver and I'm a Docker n00b, so I did this docker-compose script to help me out.

I still could not connect from a pgadmin4 container to this database no matter what I did, but DBeaver does everything I need, so I'll stick with it for now.  PGAdmin4 should work if I install it in the OS, so that's a fallback if I need it.

I created a service so this starts up on boot.  Create a file called postgres-docker in the /etc/init.d folder with this script in it:
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
sudo chmod 755 postgres-docker 
```

Create the service launcher
```console
cd /etc/rc5.d
sudo ln -s ../init.d/postgres-docker S20postgres-docker
sudo systemctl daemon-reload
```

You should be able to do `sudo service postgres-docker start` to kick it off