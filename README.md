
# SNMP-Kafka Project
Network Analytics and Telemetry - Lab

## วัตถุประสงค์
เก็บข้อมูล interface counter จาก Router ด้วย SNMP แล้วส่งเข้า Apache Kafka เพื่อให้ consumer อ่านไปวิเคราะห์ต่อ

## สถาปัตยกรรม

    [GNS3: R1 -- R2] --SNMP(UDP 161)--> [Ubuntu VM: Docker]
                                          |- producer (poll SNMP)
                                          |- Kafka (KRaft)
                                          |- consumer

## เครื่องมือที่ใช้
GNS3, VMware, Ubuntu Server, Docker, Apache Kafka, Python

## ขั้นตอนการรัน
1. สร้าง topology ใน GNS3 และนำคอนฟิกใน `configs/R1.txt`, `configs/R2.txt` ไปใส่ที่ router
2. คัดลอก `.env.example` เป็น `.env` แล้วแก้ `SNMP_TARGETS` เป็น IP ของ fa0/0
3. รัน `docker compose up -d --build`
4. ดูผลด้วย `docker compose logs -f producer` และ `docker compose logs -f consumer`

## ผลการทดลอง
cd ~/snmp-kafka-project
docker compose up -d ^C
neta_admin@networklab1:~/snmp-kafka-project$ snmpget -v2c -c public 192.168.226.11 1.3.6.1.2.1.1.5.0
cd ~/snmp-kafka-project
docker compose up -d --build
docker compose ps
docker compose logs --tail 20 producer
docker compose logs --tail 20 consumer
iso.3.6.1.2.1.1.5.0 = STRING: "R1"
[+] Building 4.4s (14/14) FINISHED
 => [internal] load local bake definitions                                                                         0.0s
 => => reading from stdin 1.03kB                                                                                   0.0s
 => [producer internal] load build definition from Dockerfile                                                      0.0s
 => => transferring dockerfile: 204B                                                                               0.0s
 => [consumer internal] load metadata for docker.io/library/python:3.11-slim                                       2.3s
 => [consumer internal] load .dockerignore                                                                         0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [producer 1/5] FROM docker.io/library/python:3.11-slim@sha256:b7a9e5f6a93d6d4a31f129bbd4bdee69efccb6ded0fb230  0.2s
 => => resolve docker.io/library/python:3.11-slim@sha256:b7a9e5f6a93d6d4a31f129bbd4bdee69efccb6ded0fb230d78aafaf3  0.1s
 => [producer internal] load build context                                                                         0.1s
 => => transferring context: 94B                                                                                   0.0s
 => CACHED [consumer 2/5] RUN apt-get update && apt-get install -y snmp && rm -rf /var/lib/apt/lists/*             0.0s
 => CACHED [consumer 3/5] WORKDIR /app                                                                             0.0s
 => CACHED [consumer 4/5] RUN pip install --no-cache-dir kafka-python                                              0.0s
 => CACHED [consumer 5/5] COPY . .                                                                                 0.0s
 => [consumer] exporting to image                                                                                  0.6s
 => => exporting layers                                                                                            0.0s
 => => exporting manifest sha256:844d88a77001b6c5dbe5677a8f1b2ed19a38f5c585490ee411db3d6dad6cade5                  0.0s
 => => exporting config sha256:fa90d1c8275397bda91c6c09fbc870c9eeb86e5b41888453da71831333dc3c21                    0.0s
 => => exporting attestation manifest sha256:1b151ab3bafb912c644fb5b15d315b2c1e5185c075dadb362c466b549e28aa39      0.1s
 => => exporting manifest list sha256:9e97739d2377532a2a054ec18a8d00b0d86263a73fd1b14b2d22dde51727d31a             0.0s
 => => naming to docker.io/library/snmp-kafka-project-consumer:latest                                              0.0s
 => => unpacking to docker.io/library/snmp-kafka-project-consumer:latest                                           0.0s
 => [producer] exporting to image                                                                                  0.7s
 => => exporting layers                                                                                            0.1s
 => => exporting manifest sha256:7a304ee981596685d2391f20e58d08eb3a7bd7dcb94e86c48b86d60dfd5155f5                  0.0s
 => => exporting config sha256:145fb5aecf2ef7d65a3f3d861108ec5f714b3056b3153ce177dbb0d721111615                    0.0s
 => => exporting attestation manifest sha256:4e11fc41c8d8176e3f3e91208d846b11641feea9357cca04e9ad5fb528b1b223      0.1s
 => => exporting manifest list sha256:b59224ae78ce73a44937a939659413bb6339d0ef724195dd4dbd079d51413217             0.1s
 => => naming to docker.io/library/snmp-kafka-project-producer:latest                                              0.0s
 => => unpacking to docker.io/library/snmp-kafka-project-producer:latest                                           0.0s
 => [consumer] resolving provenance for metadata file                                                              0.1s
 => [producer] resolving provenance for metadata file                                                              0.1s
[+] Running 5/5
 ✔ snmp-kafka-project-consumer              Built                                                                                                                                                           0.0s
 ✔ snmp-kafka-project-producer              Built                                                                                                                                                           0.0s
 ✔ Container kafka                          Running                                                                                                                                                         0.0s
 ✔ Container snmp-kafka-project-producer-1  Started                                                                                                                                                        13.3s
 ✔ Container snmp-kafka-project-consumer-1  Started                                                                                                                                                        13.1s
NAME                            IMAGE                         COMMAND                  SERVICE    CREATED          STATUS          PORTS
kafka                           apache/kafka:3.7.0            "/__cacert_entrypoin…"   kafka      35 minutes ago   Up 35 minutes   0.0.0.0:9092->9092/tcp, [::]:9092->9092/tcp
snmp-kafka-project-consumer-1   snmp-kafka-project-consumer   "python consumer.py"     consumer   13 seconds ago   Up 1 second
snmp-kafka-project-producer-1   snmp-kafka-project-producer   "python producer.py"     producer   13 seconds ago   Up 1 second
consumer-1  | /app/consumer.py:6: DeprecationWarning: value_deserializer does not implement kafka.serializer.Deserializer
consumer-1  |   c = KafkaConsumer(

## ผู้จัดทำ
Mr.Nattapat Ismael   ID:6331501152