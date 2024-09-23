# Setup docker-compose service with SSL self signed certificate
Setting up docker compose to running with ssl self signed certificate for an angular application.

#### Create .docker directories:

    mkdir .docker        
    mkdir .docker/nginx
    mkdir .docker/certs

#### Edit etc/hosts, add domains and save:

    ##
    # Host Database
    #
    # localhost is used to configure the loopback interface
    # when the system is booting.  Do not change this entry.
    ##
    127.0.0.1       localhost
    255.255.255.255 broadcasthost
    ::1             localhost
    127.0.0.1       example.com
    127.0.0.1       demo.example.com

#### Edit etc/ssl/openssl.conf, add domains under [  alternate_names ] and save:

    [ req ]
    #default_bits                   = 2048
    #default_md                     = sha256
    #default_keyfile                = privkey.pem
    distinguished_name              = req_distinguished_name
    req_extensions                  = req_ext
    x509_extensions                 = x509_ext
    attributes                      = req_attributes

    [ req_distinguished_name ]
    countryName                     = Country Name (2 letter code)
    countryName_min                 = 2
    countryName_max                 = 2
    stateOrProvinceName             = State or Province Name (full name)
    localityName                    = Locality Name (eg, city)
    0.organizationName              = Organization Name (eg, company)
    organizationalUnitName          = Organizational Unit Name (eg, section)
    commonName                      = Common Name (eg, fully qualified host name)
    commonName_max                  = 64
    emailAddress                    = Email Address
    emailAddress_max                = 64

    [ x509_ext ]
    subjectKeyIdentifier            = hash
    authorityKeyIdentifier          = keyid,issuer
    keyUsage                        = digitalSignature, keyEncipherment
    subjectAltName                  = @alternate_names
    nsComment                       = "OpenSSL Generated Certificate"
    
    [ req_ext ]
    subjectKeyIdentifier            = hash
    
    basicConstraints                = CA:FALSE
    keyUsage                        = digitalSignature, keyEncipherment
    subjectAltName                  = @alternate_names
    nsComment                       = "OpenSSL Generated Certificate"

    [ alternate_names ]
    DNS.1                           = localhost
    DNS.2                           = localhost.localdomain
    DNS.3                           = 127.0.0.1
    DNS.4                           = example.com
    DNS.5                           = demo.example.com
    
    [ req_attributes ]
    challengePassword               = A challenge password
    challengePassword_min           = 4
    challengePassword_max           = 20

#### Generate certificate:

    openssl req -config /etc/ssl/openssl.cnf -new -x509 -sha256 -newkey rsa:4096 -nodes -days 365 -keyout ./.docker/certs/example-key.pem -out ./.docker/certs/example.com.pem
    
  Type required information during the flow.

#### Add certificate to chain:

##### MAC OS:
  - Search for `Keychain Access` and open it
  - Select `System` from left side `System Keychains`
  - Click on `Certificates` tab
  - Drag the created certificate from `./.docker/certs/cert.pem` inside certificates list
  - Doble click on certificate item which dragged into the list to open cert dialog
  - Expand `Trust` section
  - Change option to: When using this certificate: `Always Trust`
  - Close cert dialog

#### Create `.docker/nginx/nginx.conf` file:

    events { worker_connections 1024; }

    http {
        server_tokens off;
        include                 /etc/nginx/mime.types;
        default_type            application/octet-stream;
        sendfile                on;
        keepalive_timeout       65;

        gzip                    on;
        gzip_comp_level         6;
        gzip_vary               on;
        gzip_min_length         1000;
        gzip_proxied            any;
        gzip_types              text/plain text/css application/json application/clientX-javascript text/xml application/xml application/xml+rss text/javascript;
        gzip_buffers            16 8k;


        server {
            listen              80;
            server_name         localhost;
            root                /usr/share/nginx/html;
    
    
            # Only for local development
            listen              443 default_server ssl;
            ssl_password_file   /usr/share/nginx/certs/ssl_passwords;
            ssl_certificate     /usr/share/nginx/certs/example.com.pem;
            ssl_certificate_key /usr/share/nginx/certs/example-key.pem;
    
    
            # Cache static assets and media files.
            location ~ ^/example/(media|assets)/(.*) {
                expires         30d;
            }

    
            location / {
                try_files       $uri /example/index.html?$args;
            }
    
    
            # Cache styles and scripts.
            location ~ \.(css|js)$ {
                expires         30d;
                add_header      Cache-Control "public, max-age=2592000";
            }
        }
    }
            

#### Create docker compose file: `docker-compose.yaml` 
    services:
      eample:
        container_name: example
        image: nginx:alpine
        ports:
          - "80:80"
          - "443:443"
        networks:
          - local-network
        volumes:
          - ./.docker/nginx/example.conf:/etc/nginx/nginx.conf
          - ./.docker/certs/:/usr/share/nginx/certs:delegated
          - ./dist/apps/example/browser/:/usr/share/nginx/html/example:delegated

  Run example service: `docker-compose up -d example`.
  Access it: `https://example.com`
