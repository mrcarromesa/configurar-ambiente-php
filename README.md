<h1>O que preciso para programar php no mac OS?</h1>

<strong>Referências</strong>
- [Home brew](https://brew.sh/index_pt-br)
- [mac OS + apache + homebrew + php](https://getgrav.org/blog/macos-catalina-apache-multiple-php-versions)
- [Install php with homebrew](https://samsonasik.wordpress.com/2019/12/01/install-php-7-4-in-macos-sierra-with-homebrew/)
- [Xdebug](https://xdebug.org/docs/install)
- [Installing PHP on Apache](https://www.petefreitag.com/item/516.cfm)
- [Php Debug](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)
- [How do I change the PHP version my shell uses?](https://help.dreamhost.com/hc/en-us/articles/214202148-How-do-I-change-the-PHP-version-my-shell-uses-)

---

<h2>Start<h2>
<p>O apache e o php que já vem instalado por padrão no mac OS, tem algumas limitações, e Homebrew, já traz várias facilidades para trabalhar melhor em instalar e gerenciar novas versões do php, bem como funcionalidades que o padrão limita o desenvolvedor, como no caso de realizar debug com o php.</p>

---

<h2>Mão na massa</h2>

<h3>

[Homebrew](https://brew.sh/index_pt-br)

</h3>

- Caso não tenha a última versão do Xcode no seu Mac, instalar o ```command line tools``` , necessárias para o [Homebrew](https://brew.sh/index_pt-br).

```bash
    xcode-select --install
```

 - Instalar o [Homebrew](https://brew.sh/index_pt-br):

 ```bash
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- Após instalação verificar se o homebrew está funcionando:

```bash
    brew --version
```
- Verificar se não há erros ou pendências do Homebrew:

```bash
    brew doctor
```

- Caso só houver avisos, em geral podem ser ignorados, se não só seguir as instruções em tela que o Homebrew exibe.

- (Opcional)Instalar algumas bibliotecas para o sistema operacional, ```particularmente não precisei fazer isso```:

```bash
    brew install openldap libiconv
```

<h3>Apache via Homebrew</h3>

- Para o apache:

```bash
    sudo apachectl stop
```

- Parar os processos associados:

```bash
    sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

- Instalar via Homebrew:

```bash
    brew install httpd
```

- Iniciar o serviço:

```bash
    sudo brew services start httpd
```

- Aguardar até o serviço iniciar e acessar no browser [http://localhost:8080](http://localhost:8080)

- Em caso de não funcionar, tente executar o comando:

```bash
    sudo apachectl -k restart
```

---

<strong>Configurando...</strong>

<p>

Por padrão o serviço ```httpd``` executará na porta 8080, vamos configurar isso

</p>

- Acessar o arquivo /usr/local/etc/httpd/httpd.conf:

```bash
    sudo nano /usr/local/etc/httpd/httpd.conf
```

- Buscar por ```Listen 8080``` e alterar por ```Listen 80```

- Buscar por ```DocumentRoot``` alterar o caminho ```DocumentRoot "/usr/local/var/www"``` por ```DocumentRoot "/Library/WebServer/Documents/http"```

- Buscar por ```<Directory``` alterar por ```<Directory "/Library/WebServer/Documents/http">```, juntamente com caminho anterior.

- Buscar por ```AllowOverride None``` e alterar por ```AllowOverride All```

- Buscar por ```mod_rewrite.so``` e descomentar a linha removendo o ```#``` ficando assim: ```LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so```

- Em algum lugar no arquivo terá o seguinte:

```
User _www
Group _www
```

Alterar para:

```
User SEU_USUARIO
Group staff
```

- O ```SEU_USUARIO``` deverá ser o seu usuário do mac.

- Procuar por ```#ServerName www.example.com:8080``` e alterar por ``` ServerName localhost```

- Salvar o arquivo;

- Executar o comando para restart o apache:

```bash
    sudo apachectl -k restart
```

- Por fim criar a pasta ```http``` dentro de ```/Library/WebServer/Documents/```

```bash 
    sudo mkdir /Library/WebServer/Documents/http
```

---

<strong>Instalar o PHP</strong>

- Instalar o php 7.4:

```bash
    brew install php@7.4
```

- Após instalação é necessário adicionar as configurações do php no apache, dessa forma abra o arquivo ```sudo nano /usr/local/etc/httpd/httpd.conf``` novamente, (pode acontecer de durante a instalação ele já ter sido inserido, caso já, só conferir ao invés de inserir) e insira a seguinte informação:


```
LoadModule php7_module /usr/local/opt/php/lib/httpd/modules/libphp7.so

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
<Files *.php>
    SetHandler application/x-httpd-php
</Files>

```

- Salve o arquivo e execute o comando:

```bash
    sudo apachectl -k restart
```

- configura php linha de comando, caso utilize o terminal padrão do mac OS, acesse o arquivo ```nano ~/.bash_profile```, caso utilize outro como o ohmyzshrc digite ```nano ~/.zshrc``` e adicione:

```
    export PATH=/usr/local/opt/php/bin:$PATH
```

- Salve o arquivo

- Execute o comando, caso foi alterado o arquivo ```~/.bash_profile```: 

```bash
    . ~/.bash_profile
```

- Execute o comando, caso foi alterado o arquivo ```~/.zshrc```:

```bash
    . ~/.zshrc
```

- Verificar a versão do php, execute o comando:

```bash
    php -v
```

- deverá aparecer as informações de versão do php.

- Crie um arquivo ```index.php``` em ```/Library/WebServer/Documents/http``` e adicione:

```php
    <?php
        phpinfo();
```

- Salve o arquivo e acesse no browser: ```http://localhost/index.php```

- Se tudo der certo aparecerá as informações do php

---

<strong>Debug com PHP</strong>

- Instalar o [Xdebug](https://xdebug.org/docs/install)

- Executar o comando:

```bash
    pecl install xdebug
```

- Pode ser que após a instalação já insira a configuração de extensão no em ```sudo nano /usr/local/etc/php/7.4/php.ini```, só verificar se foi inserido: 

```
    zend_extension="xdebug.so"
```

caso contrário só inserir.

- Adicionalmente deve ser inserido caso não houver, nesse mesmo arquivo o seguinte:

```
    [XDebug]
    xdebug.remote_enable = 1
    xdebug.remote_autostart = 1
```

- Feito isso, salve o arquivo e restart o apache:

```bash
    sudo apachectl -k restart
```

---

<strong>Instalando e configurando no VSCode</strong>

- Caso não tenha instalado o [VSCode](https://code.visualstudio.com), realize a instalação.

- Instale o plugin: [Php Debug](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)

- Abra a pasta ```/Library/WebServer/Documents/http```, (<strong>Tem que ser a pasta e não só um arquivo isolado.</strong>), no vscode;

- Adicione mais códigos no seu arquivo index.php, para testes;

- No vscode clique no icone de debug, no menu na esquerda do vscode.

- Clique em create a lauch.json file

- Escolha PHP, no passo seguinte

- será gerado um arquivo de configuração

- Salve esse arquivo.

- vá até o arquivo index.php, e insira um breakPoint

---

<strong>Outros plugins</strong>
<p>Para ganhar mais produtividade e código formatado em PHP, além de autocomplet, alguns plugins ajudam muito nisso, porém alguns precisam de pacotes do composer instalado para funcionar.</p>

- Inicialmente instale o composer, só seguir as instruções em: ```https://getcomposer.org/download/```

- Após o composer devidamente instalado execute o comando para instalar o [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer#installation):

```bash
composer global require friendsofphp/php-cs-fixer
```

- Após instalado precisamos instalar mais outro pacote do composer o ``` squizlabs/php_codesniffer ```:

```bash
composer global require squizlabs/php_codesniffer
```

- Com os pacotes instalados, precisamos baixar alguns plugins que são eles:

    - [phpcs](https://marketplace.visualstudio.com/items?itemName=ikappas.phpcs)

    - [php cs fixer](https://marketplace.visualstudio.com/items?itemName=junstyle.php-cs-fixer)

- Após instalar os plugins, falta configurar:

- Pressione command + shift + P

- Busque por ```JSON```

- Clique em ```Preferences: Open Settings (JSON)```

- Adicione o seguinte:

```js
    "php-cs-fixer.executablePath": "/Users/SEU_USUARIO_AQUI/.composer/vendor/bin/php-cs-fixer",
    "php-cs-fixer.executablePathWindows": "",   //eg: php-cs-fixer.bat
    "php-cs-fixer.onsave": true,
    "php-cs-fixer.rules": "@PSR12",
    "php-cs-fixer.config": ".php_cs;.php_cs.dist",
    "php-cs-fixer.allowRisky": false,
    "php-cs-fixer.pathMode": "override",
    "php-cs-fixer.exclude": [],
    "php-cs-fixer.autoFixByBracket": true,
    "php-cs-fixer.autoFixBySemicolon": true,
    "php-cs-fixer.formatHtml": false,
    "php-cs-fixer.documentFormattingProvider": true,
    "[php]": {
        "editor.defaultFormatter": "bmewburn.vscode-intelephense-client"
    },

    "phpcs.executablePath": "/Users/SEU_USUARIO_AQUI/.composer/vendor/bin/phpcs",
    "phpcs.showSources": true,
    "phpcs.standard": "PSR12",
    "simple-php-cs-fixer.save": true,
    "simple-php-cs-fixer.rules": "@PSR1,@PSR2,trailing_comma_in_multiline_array",
```

- ***Detalhe***: Substitua ```SEU_USUARIO_AQUI``` pelo seu nome de usuário, ou para saber qual a pasta do composer digite no terminal:

```bash
composer config --list --global
```

- Esse comando irá retornar uma lista, encontre ```[home]```, o valor dele colocar no lugar de ```/Users/SEU_USUARIO_AQUI/.composer``` no código acima.

- Ok feito isso salve o arquivo de Settings em JSON do vscode.

- Um último componente que deve ser utilizado antes de testar qualquer coisa é o [PHP Intelephense](https://marketplace.visualstudio.com/items?itemName=bmewburn.vscode-intelephense-client)


- Agora tudo deverá estar funcionando.


