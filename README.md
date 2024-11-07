# Raspberry Pi Docker Swarm instructions

## Initial SD card operating system installation

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Start up Raspberry Pi Imager and use "CHOOSE DEVICE" to select your
   Raspberry Pi model
3. Select "CHOOSE OS" and navigate to
   "Other general-purpose OS" -> Ubuntu -> "Ubuntu Server 24.04.1 LTS (64-bit)"

   NOTE: If you are using a Raspberry Pi 2 or below, or a Raspberry Pi Zero,
   you'll need to pick the 32-bit version instead, and some apps may take some
   effort to get working
4. Insert your SD Card into your computer
5. Select "CHOOSE STORAGE" and select the SD Card you've just inserted. Ensure
   you have the correct device, as it'll be completely wiped and you'll lose
   anything on it
6. Select "NEXT"
7. When asked "Would you like to apply OS customisation settings?", Select "NO"
8. You'll now receive your final prompt to confirm this is the correct storage
   device to wipe. Check this, and then select "YES"
9. When prompted, remove the SD card and close Raspberry Pi Imager

## Customising operating system configuration

1. Insert the SD card you've just freshly flashed with Ubuntu Server
2. Your computer should detect the card and you should be able to find a new
   drive in your file manager. On MacOS this will be called "system-boot"
3. Copy all of the files from this repository's `cloud-init` directory into the
   top directory of the SD card
4. Eject the SD card from your computer

## Raspberry Pi first boot

1. Insert a configured SD card into your Raspberry Pi, and connect ethernet,
   then power
2. Wait a few minutes, and then you should be able to run `ssh
   pi@raspberrypi.local` with the default password `mbt`

## Setting up your swarm

```bash
$ ssh pi@mbt1.local ip -4 a show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.11.102/24 metric 100 brd 192.168.11.255 scope global dynamic eth0
       valid_lft 42906sec preferred_lft 42906sec
$ ssh pi@mbt1.local docker swarm init --advertise-addr 192.168.11.102
Swarm initialized: current node (fc3148voxuromdn6ab0p6zayb) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2y5on6clnwj9yxqlfqqdba68aai1ntcjdx5t9bn5mbznplhpt7-bqj26vi28yfdrq58lhiz3ky86 192.168.11.102:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

$ ssh pi@mbt2.local docker swarm join --token SWMTKN-1-2y5on6clnwj9yxqlfqqdba68aai1ntcjdx5t9bn5mbznplhpt7-bqj26vi28yfdrq58lhiz3ky86 192.168.11.102:2377
This node joined a swarm as a worker.
```

[Install Swarmpit](https://github.com/swarmpit/swarmpit#installation):

```bash
$ ssh -t pi@mbt1.local docker run -it --rm \
  --name swarmpit-installer \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  swarmpit/install:1.9
```

Configure Traefik for HTTP(S) proxying

```bash
docker network create --attachable --driver overlay proxy
# Because we have variable interpolation stuff in traefik.yml, we pre-process
# that first, and then actually run the deploy using the resulting file
docker stack config --compose-file traefik.yml | \
  docker stack deploy --detach --compose-file - traefik
```

[Install and configure GlusterFS](https://thenewstack.io/tutorial-create-a-docker-swarm-with-persistent-storage-using-glusterfs/)

```bash
sudo apt install glusterfs-server
sudo systemctl enable --now glusterd
sudo gluster peer probe mbt2  # Run on mbt1
sudo gluster pool list  # Just a check
sudo mkdir -p /srv/gluster/vol1
sudo gluster volume create vol1 replica 2 \
  mbt1:/srv/gluster/vol1 mbt2:/srv/gluster/vol1 force  # On mbt1
sudo gluster volume start vol1  # On mbt1
sudo mkdir /vol1
echo 'localhost:/vol1 /vol1 glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0' \
  | sudo tee -a /etc/fstab
sudo systemctl daemon-reload
sudo mount -a
sudo chown -R root:docker /vol1
sudo chmod g+w /vol1
```

Set up Kiwix

```bash
pi@mbt1:~$ mkdir /vol1/kiwix
pi@mbt1:~$ wget -nv -P /vol1/kiwix https://download.kiwix.org/zim/wikipedia/wikipedia_en_100_2024-06.zim
2024-11-08 01:05:05 URL:https://md.mirrors.hacktegic.com/kiwix-md/zim/wikipedia/wikipedia_en_100_2024-06.zim [386004857/386004857] -> "/vol1/kiwix/wikipedia_en_100_2024-06.zim" [1]
```
