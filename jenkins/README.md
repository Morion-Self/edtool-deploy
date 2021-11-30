# CI/CD for edtool․ru

У меня есть [пет-проект](https://edtool.ru) на стеке JavaScript (OpenUI5) + PHP + MySQL. Полноценного CI/CD у меня нет, потому что у меня нет отдельного сервера, работающего 24/7 (*пет-проект хостится на классическом виртуальном хостинге, там нельзя установить Jenkins и т.п.*). Я пишу код локально, а когда его нужно выкатить в тест или прод — запускаю Jenkins-таск.

Это — один из примеров, как у меня настроены задачи. Он почти 1 в 1 совпадает с моим действующим пайплайном, кроме некоторых небольших изменений. Например, в настоящем пайплпайне у меня только одна нода Jenkins: она master и в то же время собирает проект. Но для портфолио я хочу показать сборку на отдельной ноде, поэтому в этом примере я создаю отдельную ноду и запускаю сборку на ней.

Насколько это возможно, я следую принципам IaC. Есть некоторые ручные вмешательства (*например, настройка кредов*), но все что возможно описано в коде. Если мне понадобится создать такую же среду на другом компе, я смогу быстро это сделать.

Для создания ВМ я использую [Vagrant](https://www.vagrantup.com/). Он же устанавливает Jenkins и нужные зависимости. В этом примере я не стал заморачиваться с Ansible, потому что цель этого раздела в портфолио — показать работу с Jenkins, а не Ansible.

Сам по себе пайплайн простой: клонируем репозиторий с гитхаба, собираем проект и копируем его на сервер.

Ниже две инструкции. 
- Первая — Provisioning. Её нужно выполнить один раз, для создания ВМ и настройки Jenkins.
- Вторая — для деплоя кода на сервер.

## Первоначальная настройка Jenkins
*Чтобы поднять и настроить Jenkins к работе, следуй этим инструкциям*:

1. Создать ВМ: 

    `cd vagrant && vagrant up && cd ..`

1. [Активировать Jenkins](http://192.168.56.100:8080)

    Чтобы получить пароль админа:
    ```sh
    cd vagrant && vagrant ssh jenkins << EOF
        sudo cat /var/lib/jenkins/secrets/initialAdminPassword 
    EOF
    cd ..
    ```
    Не устанавливать никаких плагинов.

1. Создать токен для пользователя.

1. Экспортировать переменные:

    ```bash
    export JENKINS_USER_ID=<user_id>
    export JENKINS_API_TOKEN=<token>
    ```

1. Скачать CLI:

    `wget http://192.168.56.100:8080/jnlpJars/jenkins-cli.jar`

1. Установить плагины:

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ install-plugin ssh-slaves ssh-credentials ws-cleanup git publish-over-ssh -restart`

1. [Настроить плагин](http://192.168.56.100:8080/configure) Publish over SSH: 
    - **Name**: edtool
    - **Hostname**: IP-адрес
    - **Username**: Пользователь
    - **Extended => Use password authentication**: Пароль (*На классических тарифах моего хостинга нельзя авторизоваться по ключам, поэтому использую пароль*)

1. Создать заглушки для кредов:

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ -webSocket create-credentials-by-xml system::system::jenkins _ < credentials/ssh-key-github.xml`

    - `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ -webSocket create-credentials-by-xml system::system::jenkins _ < credentials/ssh-key-node1.xml`

1. Настроить креды:
    - [Для гитхаба](http://192.168.56.100:8080/credentials/store/system/domain/_/credential/ssh-key-github/update).
    - [Для node1](http://192.168.56.100:8080/credentials/store/system/domain/_/credential/ssh-key-node1/update) (Получить ключ: `cat vagrant/.vagrant/machines/node1/virtualbox/private_key`)

1. Создать ноду: 

    `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ create-node node1 < nodes/node1.xml`

1. Создать задачу: 

    `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ create-job edtool-deploy-to-prd < jobs/edtool-deploy-to-prd.xml`

## Deploy
*Деплой на продакшен-сервер*:

1. Запустить ВМ:

    `cd vagrant && vagrant up && cd ..`

1. Экспортировать переменные:

    ```bash
    export JENKINS_USER_ID=<user_id>
    export JENKINS_API_TOKEN=<token>
    ```

1. Деплой:
    
    `java -jar jenkins-cli.jar -s http://192.168.56.100:8080/ -webSocket build edtool-deploy-to-prd -s -v`