# Notes

## CLI access

Once the console is up, you can connect to it with the minio console:

```sh
# start a container with the  cli
docker run -it --entrypoint=/bin/sh minio/mc

sh-4.4#
# note the console changed, as we are in the container ^^
```

```sh
# connect to the minio instance (within the container)
mc alias set minio https://s3.<! your domain!> minio-admin <!your secret key!>

Added `minio` successfully.
sh-4.4#

```

```sh
# test the connection
mc admin info minio

●  s3.fabricesemti.net
   Uptime: 27 minutes
   Version: 2022-08-25T07:17:05Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1

Pools:
   1st, Erasure sets: 1, Disks per erasure set: 1

1 drive online, 0 drives offline
sh-4.4#

```

After this we can interact with Mino, for example list / create / remove users

```sh

sh-4.4# mc admin user list minio
enabled    velero                readwrite,diagnos...
enabled    thanos                readwrite
sh-4.4# mc admin user add minio testuser password123!
Added user `testuser` successfully.
sh-4.4# mc admin user list minio
enabled    testuser
enabled    thanos                readwrite
enabled    velero                readwrite,diagnos...
sh-4.4# mc admin user remove minio testuser
Removed user `testuser` successfully.
sh-4.4# mc admin user list minio
enabled    thanos                readwrite
enabled    velero                readwrite,diagnos...
sh-4.4#

```

### Postgres setup

- crete the Postgres user

```sh
mc admin user add minio postgresql <pw from cluster secrets>
# Added user `postgresql` successfully.
```

- create bucket

```sh
mc mb minio/postgresql
# Bucket created successfully `minio/postgresql`.
```

- create policy and apply it on the bucket

```sh

cat <<EOF > postgresql-user-policy.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:ListBucket",
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:DeleteObject"
                ],
                "Effect": "Allow",
                "Resource": ["arn:aws:s3:::postgresql/*", "arn:aws:s3:::postgresql"],
                "Sid": ""
            }
        ]
    }
EOF
# this saves the policy file

mc admin policy add minio postgresql-private postgresql-user-policy.json
# Added policy `postgresql-private` successfully.
# this applies the policy file
```

- finally, associate the policy with the user

```sh
mc admin policy set minio postgresql-private user=postgresql
# Policy `postgresql-private` is set on user `postgresql`

```

### Velero setup

- create bucket

```sh
mc mb minio/velero
```

### Thanos setup

- create bucket

```sh
mc mb minio/thanos
```

### Finally...

Confirm the buckets are created

```sh
sh-4.4# mc ls minio
[2022-09-09 07:10:54 UTC]     0B postgresql/
[2022-09-09 07:19:55 UTC]     0B thanos/
[2022-09-09 07:19:47 UTC]     0B velero/
sh-4.4#
```

reference:

- <https://github.com/minio/mc/blob/master/docs/minio-client-complete-guide.md>

- <https://www.stackhero.io/en/services/MinIO/documentations/Getting-started/Use-the-MinIO-CLI>

- <https://hub.docker.com/r/minio/mc/>
