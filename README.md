Configurando um proxy reverso com Nginx
=======================================

Este guia explica como configurar um proxy reverso usando o servidor web Nginx em um sistema Ubuntu. O proxy reverso permite que você direcione o tráfego da internet para um servidor web que esteja sendo executado em uma porta diferente ou em uma máquina diferente. Essa configuração é útil quando você deseja hospedar vários sites em uma única máquina ou quando deseja executar um aplicativo em um servidor diferente, mas ainda acessível pelo nome do domínio.

Instalando o Nginx
------------------

Antes de configurar o proxy reverso, você precisa instalar o Nginx em seu sistema Ubuntu. Você pode fazer isso executando os seguintes comandos:

```
sudo apt-get update
```
```
sudo apt-get install nginx -y
```

Configurando o arquivo de site do Nginx
---------------------------------------

Em seguida, você precisa criar um arquivo de configuração para o seu site no diretório `/etc/nginx/sites-available`. Você pode criar o arquivo usando o editor de texto `nano` ou outro editor de sua escolha. O nome do arquivo deve ser o dominio que pretende usar. Por exemplo:

```
sudo nano /etc/nginx/sites-available/meu-app.com.br
```

Dentro do arquivo de configuração, adicione o seguinte código:

```
server {
    listen 80;
    server_name meu-app.com.br www.meu-app.com.br;
    location / {
        proxy_pass http://10.10.10.100:6464; # Porta em que o site está sendo executado
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # Substitui 'http://' por 'https://' em todas as respostas HTTP
        sub_filter 'http://' 'https://';
        sub_filter_once off;
    }
}

```

Este código configura o Nginx para escutar as solicitações HTTP para `meu-app.com.br` e `www.meu-app.com.br`. Quando uma solicitação é recebida, o Nginx encaminha a solicitação para o endereço IP `10.10.10.100` na porta `6464`, onde o site está sendo executado. Os cabeçalhos `Host`, `X-Real-IP` e `X-Forwarded-For` são definidos para garantir que as solicitações sejam encaminhadas corretamente para o servidor de destino. Além disso, o código inclui um filtro para substituir todas as ocorrências de `http://` por `https://` em todas as respostas HTTP.

Habilitando o site no Nginx
---------------------------

Depois de criar o arquivo de configuração, você precisa habilitar o site no Nginx. Para fazer isso, crie um link simbólico do arquivo de configuração no diretório `/etc/nginx/sites-enabled`. Você pode fazer isso executando o seguinte comando:

```
sudo ln -s /etc/nginx/sites-available/meu-app.com.br /etc/nginx/sites-enabled/
```

Testando a configuração do Nginx
--------------------------------

Antes de reiniciar o Nginx, é uma boa prática testar a configuração para garantir que não haja erros. Você pode fazer isso executando o seguinte comando:

```
sudo nginx -t
```

Se a configuração estiver correta, você verá uma mensagem de confirmação. Caso contrário, o comando informará o erro encontrado na configuração.

Reiniciando o serviço do Nginx
------------------------------

Após testar a configuração do Nginx, você pode reiniciar o serviço para aplicar as alterações. Execute o seguinte comando:

```
sudo systemctl restart nginx
```

Configurando SSL com Let's Encrypt
----------------------------------

Para habilitar SSL em seu site, você pode usar o Let's Encrypt, um serviço de certificação SSL gratuito e automatizado. Para isso, você precisa instalar o cliente Certbot do Let's Encrypt e executar o comando `certbot` para obter um certificado SSL válido. Você pode fazer isso executando os seguintes comandos:

```
sudo apt-get update -y sudo apt-get install certbot python3-certbot-nginx -y 
```
```
sudo certbot --nginx -d meu-app.com.br --agree-tos --redirect --hsts --email meu-email@mail.com
```

Este comando instala o Certbot e o plugin do Nginx, executa o comando `certbot` com as opções necessárias para obter um certificado SSL válido para `meu-app.com.br`, redirecionar todas as solicitações HTTP para HTTPS, ativar o mecanismo HSTS (HTTP Strict Transport Security) e fornecer um endereço de e-mail para receber avisos de expiração do certificado.

Verificando a renovação automática do certificado SSL
-----------------------------------------------------

Por padrão, o certificado SSL do Let's Encrypt é válido por 90 dias. Para garantir que seu certificado seja sempre válido, você precisa configurar a renovação automática. Você pode fazer isso executando o seguinte comando:


```
sudo certbot renew --dry-run
```

Este comando verifica se a renovação automática do certificado SSL está configurada corretamente. Ele não renova o certificado, mas apenas verifica se o Certbot consegue localizar e renovar o certificado quando necessário.

Com isso, você concluiu a configuração do proxy reverso e SSL para o seu site com o Nginx. Agora, seu site está configurado para redirecionar todo o tráfego HTTP para HTTPS e para encaminhar solicitações para um servidor web em uma porta diferente ou em uma máquina diferente.
