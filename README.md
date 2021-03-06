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
    >>> import os
    >>> import fiasco.util
    >>> paths = fiasco.util.setup_paths()
    >>> paths['hdf5_dbase_root'] = os.path.join(os.environ['HOME'], 'dbase', 'chianti.h5')
    >>> if not os.path.exists(os.path.dirname(paths['hdf5_dbase_root'])):
            os.makedirs(os.path.dirname(paths['hdf5_dbase_root']))
    >>> fiasco.util.download_dbase(paths['ascii_dbase_root'], ask_before=False) # can take a while
    >>> fiasco.util.build_hdf5_dbase(paths['ascii_dbase_root'], paths['hdf5_dbase_root'], ask_before=False) # can take a while
    ```
    **NOTE**: It may be necessary to enable a 1GB swapfile in order to build the HDF5 file without the droplet killing it. Especially likely on small droplets. [See this discussion](https://www.digitalocean.com/community/questions/npm-gets-killed-no-matter-what?answer=18115) or [these instructions](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-12-04). 
6. Configure HDF5 server [`h5serv`](https://github.com/HDFGroup/h5serv)
    * `git clone https://github.com/HDFGroup/h5serv.git`
    * Run tests and make sure all pass
    * Set the following configuration options in `h5serv/server/config.py`
        * `port`: 5000
        * `datapath`: `$HOME/dbase`
        * `domain`: `fiasco.data`
        * `log_file`: `$HOME/h5serv.log`
    * Set privileges to allow only `GET` requests (see the [h5serv docs](http://h5serv.readthedocs.io/en/latest/index.html))
        ```bash
        $ cd $HOME/h5serv/util/admin
        $ python setacl.py -file $HOME/dbase/chianti.h5 +r-cudep
        ```
    * Allow only selected user (not for public use yet) (see the [h5serv docs](http://h5serv.readthedocs.io/en/latest/index.html))
    * Start the server
        ```
        $ cd $HOME/h5serv/server
        $ python app.py &
        ```
8. Reverse proxy through Nginx

Optionally, install the [micro text editor](https://micro-editor.github.io/) for easier text editing.
```bash
$ curl https://getmic.ro | bash
$ mkdir .local && mkdir .local/bin && mv micro .local/bin
# Add .local/bin to PATH in .bashrc
```
