# 📃 Postgres Replication Server

To set up a Postgres replica server, you can use streaming replication to create
a standby server that continuously receives updates from the primary server. Here’s a high-level
setup using Docker Compose for a replica setup:

***Preparation Steps:***



* Ensure that your primary server allows replication connections.

* Edit the **_postgresql.conf_** file to enable replication.

* Add a user for replication.

* Set Up Docker Compose for Replica Server

* Configure a secondary container with streaming replication enabled.

* Connect the primary and secondary via a network that allows inter-server communication.

***Steps in Detail***

1. Configure the Primary Server 
In your primary server configuration, do the following:

   * Update **_postgresql.conf_** to enable replication:
   * Add these settings to **_postgresql.conf_**:
     ```bash
     wal_level = replica
     max_wal_senders = 10
     hot_standby = on
     ```
   ***You may find `postgresql.conf` in `/var/lib/postgresql/data` or in a custom location based on your setup.***
Add a Replication User to the primary database:
     
     * Log into the Postgres shell in the primary container:
       ```bash 
       docker exec -it postgres psql -U postgres
       ```
     * Create a replication user:
       ```mysql
       CREATE USER replica_user WITH REPLICATION ENCRYPTED PASSWORD 'your_password';
       ```
   * Configure **_pg_hba.conf_** to allow the replica server’s IP for replication:
     ```bash
     host replication replica_user replica_server_ip/32 md5
     ```
     ***Replace replica_server_ip with the IP address of the server where the replica Postgres instance will run***


2. In the replica server, run `docker compose up -d` with the configuration provided for the replica postgres.
     

       
       
     


