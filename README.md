# CHIANTI Data in the Cloud
In this repo, I'll store all the needed instructions and scripts for making CHIANTI data available in the cloud. This can be done through some service such as AWS or Digital Ocean.

Presumably this could all be automated with something like ansible or better yet containerized with Docker.

## Steps
0. Spin up a Digital Ocean droplet with the latest Ubuntu image and log in
1. Install Nginx
```bash
$ sudo apt-get update
$ sudo apt-get install nginx
```
2. Enable firewall
```bash
$ sudo apt-get install ufw
$ sudo ufw allow 'Nginx HTTP'
$ sudo ufw enable
```
3. Install conda
4. Install all needed Python dependencies
5. Download CHIANTI database and build HDF5 version with fiasco
5. Clone and spin up HDF server `h5serv`
6. Reverse proxy through Nginx
