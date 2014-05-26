This how I managed to let firefox sync with services all running on localhost.
It may not be the most elegant way to do so, but it should get you a working 
configuration.

# system setup

## debian
update to jessie

install libgmp-dev, git, curl, nodejs, nodejs-legacy, python, g++, bzip2

    curl https://www.npmjs.org/install.sh | sh

    npm install -g grunt-cli
    npm install -g phantomjs

(incomplete)

## ArchLinux

install nodejs

(incomplete)

# Installing / configuring fxa-auth-server

    git clone git://github.com/elelay/fxa-auth-server.git
    cd fxa-auth-server
    fxa-auth-server ❯ git checkout -b runlocal origin/runlocal
    fxa-auth-server ❯ npm install
    fxa-auth-server ❯ npm start
    (fails missing key.pem) 


# Generating ssl certificate/key
See http://nodejs.org/api/tls.html

    fxa-auth-server❯ openssl genrsa -out key.pem 1024
    fxa-auth-server ❯ openssl req -new -key key.pem -out csr.pem
     -----
     Country Name (2 letter code) [AU]:FR
     State or Province Name (full name) [Some-State]:NA
     Locality Name (eg, city) []:
     Organization Name (eg, company) [Internet Widgits Pty Ltd]:elelay
     Organizational Unit Name (eg, section) []:
     Common Name (e.g. server FQDN or YOUR name) []:localhost
     Email Address []:
     
     Please enter the following 'extra' attributes
     to be sent with your certificate request
     A challenge password []:
     An optional company name []:

    fxa-auth-server ❯ openssl x509 -req -in csr.pem -signkey key.pem -out cert.pem

    fxa-auth-server ❯ npm start
    Ctrl-C















# Installing/configuring fxa-content-server

    git clone git://github.com/elelay/fxa-content-server.git
    cd fxa-content-server
    fxa-content-server ❯ git checkout -b runlocal origin/runlocal
    (already configured
    fxa-content-server ❯ cp server/config/local.json-dist server/config/local.json)
    fxa-content-server ❯ npm install
    fxa-content-server ❯ cp ../fxa-auth-server/key.pem .
    fxa-content-server ❯ cp ../fxa-auth-server/cert.pem .
    fxa-content-server ❯ npm start
    Ctrl-C
    fxa-content-server ❯ npm stop


# testing fxa
new firefox profile:
 - goto https://localhost:9000/ 
   add ssl exception
 - goto https://localhost:3030/ 
   add exception
 - create account (to verify that it's working properly)  
   toto@titi.fr  
   azertyuiop  
   CREATE  
 - grab the verify email url from the terminal running fxa-auth-server  
   (sthing like https://localhost:3030/v1/verify_email?uid=cb42b35066074c71bfd0b9dc669a62c9&code=5210ec9a11f690f5d3d8e97eb1d13c63) 
 - paste in new tab, goto
 - success

# Installing/Configuring syncserver
    tmp_mzsync ❯ git clone git://github.com/elelay/syncserver.git
    tmp_mzsync ❯ cd syncserver
    syncserver ❯ git checkout -b runlocal
    syncserver ❯ make build
    syncserver ❯ make test
    syncserver ❯ make serve

goto http://localhost:5000  
It works!

files modified but not mandatory:
 - local/lib/python2.7/site-packages/browserid/verifiers/\_\_init\_\_.py  
	to add debug info
 - local/lib/python2.7/site-packages/browserid/verifiers/local.py  
	to add debug info
 - local/lib/python2.7/site-packages/browserid/supportdoc.py  
	to add debug info

# Configuring firefox:
- goto about:config
   - services.sync.tokenServerURI = http://127.0.0.1:5000/token/1.0/sync/1.5  
   - identity.fxaccounts.settings.uri = https://localhost:3030/settings  
   - identity.fxaccounts.remote.signup.uri = https://localhost:3030/signup?service=sync&context=fx_desktop_v1   
   - identity.fxaccounts.remote.signin.uri = https://localhost:3030/signin?service=sync&context=fx_desktop_v1  
   - identity.fxaccounts.remote.force_auth.uri = https://localhost:3030/force_auth?service=sync&context=fx_desktop_v1  
   - identity.fxaccounts.auth.uri = https://localhost:9000/v1  

 - about:accounts  
    tutu@me.com  
    azertyuiop  
    CREATE
 - grab the verify email url from the terminal running fxa-auth-server  
 (sthing like https://localhost:3030/v1/verify_email?uid=6b11bebc7ec641ba9699e6a798974b0a&code=075c5f7cf0341fe6caa12a07aa451269&service=sync)
 - paste in new tab
 - "success!"
 - "Manage Sync"  
   in the sync prefs, mail should be registered and propose to disconnect
 - add the "sync now" button to the main menu
 - about:config  
    services.sync.log.appender.file.logOnSuccess = true
 - menu > "sync now"
 - about:sync-logs  
   a new file like "success-1400944961710.txt" should appear