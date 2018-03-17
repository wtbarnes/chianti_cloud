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
3. [Install conda](https://www.digitalocean.com/community/tutorials/how-to-install-the-anaconda-python-distribution-on-ubuntu-16-04) though it is probably better to use Miniconda, e.g.
    ```bash
    $ curl -O https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
    $ bash Miniconda3-latest-Linux-x86_64.sh
    ```
4. Install all needed Python dependencies using the environment file in this repository
    ```
    $ conda env create -f conda_environment.yml
    $ source activate chianti_cloud
    ```
5. Download CHIANTI database and build HDF5 version with fiasco
    ```python
    >>> import fiasco.util
    >>> paths = fiasco.util.setup_paths()
    >>> fiasco.util.download_dbase(paths['ascii_dbase_root'], ask_before=False) # can take a while
    >>> fiasco.util.build_hdf5_dbase(paths['ascii_dbase_root'], paths['hdf5_dbase_root'], ask_before=False) # can take a while
    ```
6. Configure HDF5 server [`h5serv`](https://github.com/HDFGroup/h5serv)
    * `git clone https://github.com/HDFGroup/h5serv.git`
    * Set the following configuration options in `h5serv/server/config.py`
    * Set privileges to allow only `GET` requests (see the [h5serv docs](http://h5serv.readthedocs.io/en/latest/index.html))
    * Allow only selected user (not for public use yet) (see the [h5serv docs](http://h5serv.readthedocs.io/en/latest/index.html))
    * Copy CHIANTI data into data path
    * Start the server
        ```
        $ cd h5serv/server
        $ python app.py &
        ```
8. Reverse proxy through Nginx

Optionally, install the [micro text editor](https://micro-editor.github.io/) for easier text editing.
```bash
$ curl https://getmic.ro | bash
$ mkdir .local && mkdir .local/bin && mv micro .local/bin
# Add .local/bin to PATH in .bashrc
```
