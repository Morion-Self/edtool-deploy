# CI/CD for edTool.ru

## Provisioning

1. Create VM: 

    `vagrant up`

1. [Activate Jenkins](http://192.168.56.100:8080)

    Get admin password:
    ```sh
    vagrant ssh << EOF
        sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    EOF
    ```
    Do not install any plugins.

1. Create token for your user.

1. `cp .env.example .env`

1. Put your user_id and token in `.env`-file

1. Apply env variables:

    `source .env`

1. Download CLI:

    `wget http://192.168.56.100:8080/jnlpJars/jenkins-cli.jar`

1. Install plugins:

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ install-plugin ssh-credentials ws-cleanup git publish-over-ssh -restart`

1. [Configure plugin](http://192.168.56.100:8080/configure) Publish over SSH: 
    - **Name**: *edtool (test) / edtool (prd)*
    - **Hostname**: *IP-address*
    - **Username**: *User*
    - **Extended => Use password authentication**: *Password*

1. Create templates for credentials:

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ -webSocket create-credentials-by-xml system::system::jenkins _ < credentials/front-ssh-key.xml`

1. Configure credentials:
    - [For github](http://192.168.56.100:8080/credentials/store/system/domain/_/credential/front-ssh-key/update)
 
1. Create tasks:

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ create-job front-deploy-test< jobs/front-deploy-test.xml`    
    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ create-job front-deploy-prd< jobs/front-deploy-prd.xml`

## Deploy

1. Start VM:

    `vagrant up`

1. Setup env:

    `source .env`

1. Run needed task:

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ -webSocket build front-deploy-test -s -v`
    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ -webSocket build front-deploy-prd -s -v`

## TODO:
1. CI/CD for backend
