# Install and Use GoAccess Web Log Analyzer
[GoAccess](https://goaccess.io/) is an open source real-time web log analyzer and interactive viewer that runs in a terminal in *nix systems or through your browser.

It provides fast and valuable HTTP statistics for system administrators that require a visual server report on the fly.

### Prerequisites
You may need to install build tools like gcc, autoconf, gettext, autopoint etc for compiling/building software from source. e.g., base-devel, build-essential, "Development Tools".

    sudo apt install gcc autoconf gettext autopoint libmaxminddb0 libmaxminddb-dev mmdb-bin libncursesw5-dev libncursesw5-dev libgeoip-dev libtokyocabinet-dev build-essential

### Installation
Installing GoAccess is pretty easy. Just download `(make sure to download the last version)`, extract and compile it with:

    wget https://tar.goaccess.io/goaccess-1.9.3.tar.gz

Once the download completes, extract the archive with:

    tar -xzvf goaccess-1.9.3.tar.gz

Change into the newly unpacked directory like this:

    cd goaccess-1.9.3/

Run the configure script found inside this directory:

    ./configure --enable-utf8 --enable-geoip=mmdb

The --enable-utf8 flag ensures GoAccess compiles with wide character support, while --enable-geoip enables GeoLocation support with the original GeoIP databases.

You’ll receive output similar to the following:

    Your build configuration:

      Prefix         : /usr/local
      Package        : goaccess
      Version        : 1.9.3
      Compiler flags :  -pthread
      Linker flags   : -lnsl -lncursesw -lmaxminddb -lpthread
      UTF-8 support  : yes
      Dynamic buffer : no
      ASan           : no
      Geolocation    : GeoIP2
      Storage method : In-Memory with On-Disk Persistent Storage
      TLS/SSL        : no
      Bugs           : hello@goaccess.io

Run the make command to build the makefile required for installing GoAccess:

    make

Finally, install GoAccess using the previously created makefile to the system:

    sudo make install

Ensure that the program was installed successfully by running:

    goaccess --version

If not found, check the installation path:

    whereis goaccess

Check version again:

    /usr/local/bin/goaccess --version

You will receive the following output:

    GoAccess - 1.9.3.
    For more details visit: https://goaccess.io/
    Copyright (C) 2009-2024 by Gerardo Orellana
        
    Build configure arguments:
      --enable-utf8
      --enable-geoip=mmdb

Analyse nginix access logs:

    goaccess /var/log/nginx/access.log    

For geoip location feature, download database from [GeoLite.mmdb](https://github.com/P3TERX/GeoLite.mmdb/blob/main/README.md)

    wget https://git.io/GeoLite2-City.mmdb

Be sure to save this file in an easy to remember location!

Usage:

    goaccess --geoip-database=./GeoLite2-City.mmdb /var/log/nginx/access.log
