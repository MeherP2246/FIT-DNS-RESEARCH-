
# PowerDNS with FALCON Signature + DNSSEC

### Step 1: Creating VM 

Create your own  new virtual machine (ubuntu) 

### Step 2 : Installing the liboqs provides

Note : Replace all the path as per your configuration. 

i. Installing pre-requisites before moving on to install the required libraries and programs.

```
 sudo apt install astyle cmake gcc ninja-build libssl-dev python3-pytest python3-pytest-xdist unzip xsltproc doxygen graphviz python3-yaml valgrind
```
ii. Getting the source for libous 

```
sudo git clone -b main https://github.com/open-quantum-safe/liboqs.git
```
iii. Now traverse to the liboqs directory 

``` $ cd liboqs ```

iv. And build:

```
sudo mkdir build && cd build
sudo apt-get install doxygen
sudo  cmake -GNinja ..
sudo ninja
ninja run_tests
ninja install
cd ~ 
```

### Step 3  : then 1st install openssl 

i. Installing pre-requisites before moving on to install the required libraries and programs.

```
sudo apt install cmake gcc libtool libssl-dev make ninja-build git
$sudo git clone --branch OQS-OpenSSL_1_1_1-stable https://github.com/open-quantum-safe/openssl.git openssl
```


### Step 4 : Install power DNS

i.Source code is available on GitHub:

```sudo git clone https://github.com/PowerDNS/pdns.git```

ii. Now traverse to the liboqs directory 

```
cd pdns
sudo git submodule init
sudo git submodule update
sudo ./builder/build.sh sdist
```
Note : sdist is for python, feel free to change depending on your linux arch by ruuning ( ```sudo ./builder/build.sh```)

iii. Compiling Authoritative Server

```
sudo apt install libcurl4-openssl-dev luajit lua-yaml-dev libyaml-cpp-dev libtolua-dev lua5.3 autoconf automake ragel bison flex g++ libboost-all-dev libtool make pkg-config libssl-dev lua-yaml-dev libyaml-cpp-dev libluajit-5.1-dev libcurl4 gawk libsqlite3-dev python3-venv

sudo apt install libsodium-dev

sudo apt install default-libmysqlclient-dev

sudo apt install libpq-dev

sudo apt install libsystemd0 libsystemd-dev

sudo apt install libmaxminddb-dev libmaxminddb0 libgeoip1 libgeoip-dev

```
iv. Then generate the configure file: 

```sudo autoreconf -vi```

v. To compile , and To use a OpenSSL library, use the following:

``` sudo ./configure --with-modules="" --disable-lua-records --with-libcrypto=/home/falcon/openssl```

vi. Need to add some required files before executing the make command, use the following:

```sudo nano /home/pdns/docs/requirements.txt ```

   And add  the below in the bottom of requirements.txt file 
```
pytz==2023.3 \
	--hash=sha256:a151b3abb88eda1d4e34a9814df37de2a80e301e68ba0fd856fb9b46bfbbbffb 
 
importlib-resources==5.12.0 \
	--hash=sha256:7b1deeebbf351c7578e09bf2f63fa2ce8b5ffec296e0d349139d43cca061a81a

pkgutil-resolve-name==1.3.10 \
	--hash=sha256:ca27cc078d25c5ad71a9de0a7a330146c4e014c2462d9af19c6b828280649c5e

zipp==3.15.0 \
	--hash=sha256:48904fc76a60e542af151aded95726c1a5c34ed43ab4134b597665c86d7ad556

```
vii. After that use the following:
 
 ```
 sudo make
 sudo make install
 cd ..

```


###Step 5  : Installing the PowerDNS with FALCON Signature Scheme

i. Clone below repository, install docker and docker-compose and run the python file. 

````
sudo git clone https://github.com/nils-wisiol/dns-falcon.git

sudo apt  install docker && docker-compose

cd dns-falcon

sudo rm -rf pdns

cp /home/falcon/pdns  /home/falcon/dns-falcon  

sudo docker-compose up -d

sudo python3 -m venv venv

sudo source venv/bin/activate

sudo python3 -m pip install dnspython requests 

sudo python3 setup.py

````

ii. Run the below command to used FALCON to authenticate a DNSSEC query 

``` $ dig TXT @localhost -p 5302 falcon.example. +dnssec   ```

The query output will look like as follows:

```
dig TXT @localhost -p 5302 falcon.example. +dnssec

; <<>> DiG 9.16.1-Ubuntu <<>> TXT @localhost -p 5302 falcon.example. +dnssec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46653
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 512
;; QUESTION SECTION:
;falcon.example.   		 IN    TXT

;; ANSWER SECTION:
falcon.example.   	 3600    IN    TXT    "FALCON DNSSEQ PoC; details: github.com/nils-wisiol/dns-falcon"
falcon.example.   	 3600    IN    RRSIG    TXT 13 2 3600 20230713000000 20230622000000 22665 falcon.example. xfDi+wscvrEip0e2L9K+7PQv9ywELh7Al6IG4hCIs3rzcgKrKWu34JLp N8FOztoFvVJMk83AIfFU4HILT//Xow==

;; Query time: 4 msec
;; SERVER: 127.0.0.1#5302(127.0.0.1)
;; WHEN: Sun Jul 02 14:40:32 EDT 2023
;; MSG SIZE  rcvd: 227
```

