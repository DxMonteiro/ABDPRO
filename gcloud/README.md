# Installation

## BD Server

1. Create VM:

    ```bash
    gcloud beta compute --project=abd-2020-298010 instances create server --zone=europe-west1-b --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --no-restart-on-failure --maintenance-policy=TERMINATE --preemptible --service-account=1059857475809-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=ubuntu-2004-focal-v20201201 --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-ssd --boot-disk-device-name=server --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
    ```

2. Enter VM:

    ```bash
    gcloud compute ssh server
    ```

3. Software:

    ```bash
    sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
    ```

4. Install PostgreSQL

    ```bash
    sudo apt install -y postgresql-12
    ```

5. Stop PostgreSQL service:

    ```bash
    sudo systemctl stop postgresql && sudo systemctl disable postgresql
    ```

6. Init DB:

    ```bash
    /usr/lib/postgresql/12/bin/initdb -D data
    ```

7. Change DB confInit DB:

    - `data/postgresql.conf`:
      listen_addressses = 'localhost,server'
    - `data/pg_hba.conf`:
      host all <ip_range>/24 trust

8. Start DB
    ```bash
    /usr/lib/postgresql/12/bin/postgres -D data -k.
    ```

## Bench Server

1. Create VM:

    ```bash
    gcloud beta compute --project=abd-2020-298010 instances create bench --zone=europe-west1-b --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --no-restart-on-failure --maintenance-policy=TERMINATE --preemptible --service-account=1059857475809-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=ubuntu-2004-focal-v20201201 --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=bench --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
    ```

2. Enter VM:

    ```bash
    gcloud compute ssh bench
    ```

3. Software:

    ```bash
    sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
    ```

4. Install Java 8 and PostgreSQL Client

    ```bash
    sudo apt install -y openjdk-8-jdk postgresql-client-12
    ```

5. Add bench software (on host):

    ```bash
    gcloud compute scp <path>/target/tpc-c-0.1-SNAPSHOT-tpc-c.tar.gz bench:
    gcloud compute scp <path>/extra.tgz bench:
    gcloud compute scp <path>/tpcc.dump bench:
    ```

6. Export files (on VM):

    ```bash
    tar xvf extra.tgz
    tar xvf tpc-c-0.1-SNAPSHOT-tpc-c.tar.gz
    ```

7. Change Bench conf:

    ```bash
    cd tpc-c-0.1-SNAPSHOT
    vim etc/database-config.properties
    ```

8. Start DB:

    ```bash
    /usr/lib/postgresql/12/bin/postgres -D data -k.
    ```

9. Create DB:

    ```bash
    createdb -h server tpcc
    ```

10. Import data DB:
    ```bash
    pg_restore -h server -c -d tpcc < tpcc.dump
    ```

**NOTE:** Export db: `pg_dump -h server -Fc tpcc > tpcc.dump`
