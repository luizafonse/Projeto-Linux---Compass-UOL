# Configuração de Servidor Web com Monitoramento

#### Objetivo:
Desenvolver e testar habilidades em Linux, AWS e automação de processos através da configuração de um ambiente de servidor web monitorado.

---

## Aplicações utilizadas:

### GitHub, Amazon Web Services, Microsoft Store, WSL, Visual Studio Code, Nginx.

<div style="display: flex; flex-wrap: wrap; gap: 10px; justify-content: center;">
  <br>
  <a href="https://github.com/">
    <img src="/imgs/gitlogo.png" alt="GitHub" width="100">
  </a>&ensp;

  <a href="https://www.googleadservices.com/pagead/aclk?sa=L&ai=DChcSEwjKuL74ltWLAxXoEUQIHY40KqwYABAAGgJkeg&co=1&ase=2&gclid=CjwKCAiA5eC9BhAuEiwA3CKwQp-uZ-EhfKVs_yaTVCZmvhF8olLyCz4sF_rQXc-KTkKjJ6zjkq_KbRoCmx0QAvD_BwE&ei=46i4Z7zJLbLb5OUP_NnOYQ&ohost=www.google.com&cid=CAESVeD2mSl7f0Xe0yyJImaMygYDsAuUvVqE8TXk7HbEuO8df6HhHkyj13nbeuQIUd6NDilzCovM3hpvmJWnXIKlBj1rDcr0Uva9DVYGZCTyi2T-YG-tn0A&sig=AOD64_3dqO5hHHx21zCm5ROWF8TSPV62pA&q&sqi=2&nis=4&adurl&ved=2ahUKEwj8xrT4ltWLAxWyLbkGHfysMwwQ0Qx6BAgIEAE">
    <img src="/imgs/amazonlogo.png" alt="Amazon Web Services" width="100" height="auto">
  </a>&ensp;

  <a href="https://apps.microsoft.com/home?hl=pt-BR&gl=BR">
    <img src="/imgs/micstorelogo.png" alt="Microsoft Store" width="100" height="auto">
  </a>&ensp;

  <a href="https://www.microsoft.com/store/productId/9P9TQF7MRM4R?ocid=libraryshare">
    <img src="/imgs/wsllogo.png" alt="WSL" width="100" height="auto">
  </a>&ensp;

  <a href="https://code.visualstudio.com/">
    <img src="/imgs/vscodelogo.png" alt="Visual Studio Code" width="100" height="auto">
  </a>&ensp;

  <a href="https://nginx.org/">
    <img src="/imgs/nginxlogo.png" alt="Nginx" width="100" height="auto">
  </a>&ensp;
</div>

> [!IMPORTANT]
> Únicas aplicações obrigatórias para utilização no projeto são o **GitHub** e o **Amazon Web Services**. As outras foram questão de preferência.

> [!TIP]
> - **GitHub:** Utilizado para a documentação do projeto.
> - **Amazon Web Services (AWS):** Utilizado para a criação da infraestrutura de TI.
> - **Microsoft Store:** Utilizado para baixar o WSL e o Ubuntu 24.04.
> - **WSL (Windows Subsystem for Linux):** Permite executar um ambiente Linux diretamente no sistema operacional Windows.
> - **Visual Studio Code:** Utilizado como editor de código e terminal.
> - **Nginx:** Utilizado como servidor web.

---

## Linguagens utilizadas:

### Bash, Markdown.

<div align="left">
  <br>
  <a href="https://www.gnu.org/software/bash/">
    <img src="/imgs/bashlogo.jpeg" alt="Bash" width="150" height="150">
  </a>&ensp;

  <a href="https://www.markdownguide.org/">
    <img src="/imgs/marklogo.png" alt="Markdown" width="150" height="150">
  </a>&ensp;
</div>


## Etapa 1: Configuração do Ambiente

### 1. Criar uma VPC na AWS com:
- 2 sub-redes públicas (para acesso externo).
- 2 sub-redes privadas (para futuras expansões).
- Uma Internet Gateway conectada às sub-redes públicas.

![VPC](/imgs/vpc.png)

#### Crie uma Chave PEM
Para criá-la, você pode criar na hora de criar a instância, ou até posteriormente:

![Criação de Chave PEM](/imgs/createkeypair.png)

Vá em EC2 > Key Pairs, e clique em Criar key pair.  
Sua chave precisa ser RSA e ter o formato de arquivo `.PEM`.

![Formato PEM](/imgs/keypairs.png)

---

### 2. Criar uma instância EC2 na AWS:
![Criação de Instância EC2](/imgs/launchinstance.png)

#### Opcionalmente selecionar TAGS para o monitoramento da Instância:
![Tags da Instância](/imgs/tags.png)

#### Escolher uma AMI baseada em Linux:
Selecione uma AMI baseada em Linux (Ubuntu/Debian/Amazon Linux). Ela será o sistema em que você configurará sua instância. Aqui selecionarei a do Ubuntu Linux.

![AMI do Ubuntu](/imgs/ami.png)

#### Configuração de rede:
- Instalar na sub-rede pública criada anteriormente.
- Marque a VPC criada anteriormente.
- Escolha uma subnet com IP público (exemplo: `10.0.0.0/20`).
- Em "Auto-assign public IP", marque "Enable".
- Associar um Security Group EXISTENTE que permita tráfego HTTP (porta 80) e SSH (porta 22, opcional).

![Configuração de Rede](/imgs/networkinstance.png)

---

### 3. Acessar a instância via SSH para realizar configurações futuras:
No exemplo abaixo, mostro as configurações de INBOUND e OUTBOUND utilizadas para que a instância tenha seu grupo de segurança devidamente preparado.

![Configuração de Security Group](/imgs/sginstance%201.png)

#### Existem dois meios para a configuração de um Security Group:
- Ao selecionar a aba de "Security" nos detalhes da sua instância.
![Edição de Security Group](/imgs/instanceseditsg.png)

- Ou Configurar fora da instância, indo em EC2 > Security Groups.
![Security Groups](/imgs/sg.png)

#### Ao selecionar seu Security Group, ele exibirá seus detalhes, onde você deve configurar as Inbound Rules:
![Inbound Rules](/imgs/inboundrules.png)

#### Suas configurações nesta seção seguem este padrão:
- **HTTP** com "Anywhere - IPV4".
- **SSH** com seu "my IP".

![Configuração de Inbound Rules](/imgs/editinboundrules.png)

#### Para configurar as Outbound Rules é o mesmo princípio:
![Outbound Rules](/imgs/outboundrules.png)

#### E suas regras serão as seguintes:
- **HTTP** com "Anywhere - IPV4".
- **HTTPS** com "Anywhere - IPV4".

![Configuração de Outbound Rules](/imgs/editoutboundrules.png)

Isso serve para fazer com que sua Instância tenha as configurações de segurança devidas que permitam a iniciação de um serviço, como o NGINX no nosso caso.  
As Inbound rules são as configurações de entrada, e as Outbounds de saída. Se elas não forem devidamente alteradas, seu projeto não funcionará.

---

## Etapa 2: Configuração do Servidor Web

### 0. Preparando Ambiente:
Na configuração anterior, selecionamos a AMI do Ubuntu, onde você pode utilizar de um Shell de uma máquina Linux, ou fazer da maneira que realizei, pelo Visual Studio Code.

#### Seus requisitos são:
- Instalar tanto quanto as versões mais recentes do Ubuntu e do WSL na Microsoft Store:
![Ubuntu na Microsoft Store](/imgs/ubuntustore.png)
![WSL na Microsoft Store](/imgs/WSL.png)

- É necessário possuir também o Visual Studio Code instalado com as seguintes extensões:
![Extensões do VS Code](/imgs/extensoesvscode.png)
![Extensões do VS Code](/imgs/extensoesvscode2.png)

Geralmente só de instalar a "Remote - SSH" e a "WSL" todos os outros são instalados, mas é interessante verificar.

#### Posteriormente, selecione o ícone "><":
![Conexão ao WSL](/imgs/aspinhas.png)

#### E, clique em Connect to WSL:
![Connect to WSL](/imgs/connecttowsl.png)

#### Retorne por um instante na AWS:
1. No EC2, marque sua instância e clique em "Connect".
2. Copie a última linha do exemplo de conexão SSH.
3. No terminal do VS Code, cole o comando e conecte-se à instância.

Rode estes comandos para mover a sua Chave para os arquivos locais da máquina.
```bash
cd /mnt/c/Users/"Seu usuário"/Desktop/ <-- Usuário da sua máquina/windows - seu ponto de montagem da máquina
mv chave3.pem /home/"Seu usuário"/ <-- Usuário da WSL 
```

Sim, para se conectar a sua AMI da instância é um simples copia e cola. Mas guarde bem o seu link, pois com ele qualquer um acessa e configura sua instância. Lembre-se também de dar permissão para sua chave PEM, usando o comando `chmod` como mostra abaixo.

![Configuração SSH](/imgs/configssh.png)

---

### 1. Instalar o servidor Nginx na EC2:
No seu AMI, digite os comandos de atualização de pacotes e instale o NGINX.
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install nginx
sudo systemctl status nginx
```

Após a instalação, esse é o caminho para o arquivo da sua página web, onde você pode alterar para realizar as configurações do projeto.
```bash
cd /var/www/html/
nano index.html
```

Para verificar se seu NGINX está funcionando apropriadamente:

![Status do NGINX](/imgs/nginx%20status.png)

---

### 2. Criar uma página HTML simples para ser exibida pelo servidor:
![Página HTML no NGINX](/imgs/PRINT%20NGINX.png)

---

## Etapa 3: Monitoramento e Notificações

### 1. Criar um script em Bash ou Python para monitorar a disponibilidade do site:
Para a utilização do Script, recomenda-se entrar conectado na máquina, executá-la e editá-la.

<details align="left">
    <summary style="color: #9400D3;">SCRIPT: </summary>

```bash
#!/bin/bash

# Configurações
SITE_URL="http://localhost"
LOG_FILE="/var/log/monitoramento.log"
DISCORD_WEBHOOK="link_do_webhook"

# Função para registrar logs
log() {
    local message="$1"
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $message" >> "$LOG_FILE"
}

# Função para enviar notificação no Discord
notificar_discord() {
    local message="$1"
    curl -H "Content-Type: application/json" -X POST -d "{\"content\": \"$message\"}" "$DISCORD_WEBHOOK"
}

# Verifica se o site está disponível
response=$(curl -o /dev/null -s -w "%{http_code}" "$SITE_URL")

if [[ "$response" -eq 200 ]]; then
    log "O seu Site $SITE_URL está online. E seu código de resposta: $response"
else
    log "O seu Site $SITE_URL está offline. E seu código de resposta: $response"
    notificar_discord "🚨 ATENÇÃO: O site $SITE_URL está offline! E seu código de resposta: $response"
fi
```
</details>




Cole o script bash do monitoramento e personalize os campos:
- `SITE_URL`: coloque o IP público da sua instância EC2.
- `LOG_FILE`: crie um arquivo `monitoramento.log` e deixe em branco.
- `DISCORD_WEBHOOK`: crie um webhook no Discord para receber notificações e cole seu link.

![Criação de Webhook](/imgs/criarwebhook.png)
<br><br>
![Webhook Criado](/imgs/webhookcriado.png)

Aqui mostra o código na íntegra com as informações requisitadas acima:
![Script de Monitoramento](/imgs/nano%20monitor.sh.png)

---

### 2. O script deve:
- Verificar se o site responde corretamente a uma requisição HTTP.
- Criar logs das verificações em `/var/log/monitoramento.log`.
![Log de Monitoramento](/imgs/monitoramento.log%20rodando.png)

- Enviar uma notificação via Discord, Telegram ou Slack se detectar indisponibilidade.
![Notificação no Discord](/imgs/discordnotifc.png)

---

### 3. Configurar o script para rodar automaticamente a cada 1 minuto usando cron ou systemd timers:
Edite o seu `crontab -e`, e recomendo a utilização da opção 1, o nano, que é mais prático.

![Edição do Crontab](/imgs/crontabnano.png)

Adicione a linha:
```bash
*/1 * * * * /var/www/html/monitor.sh
```


Dê permissão de administrador para seu arquivo `monitor.sh` e execute-o com o seguinte comando:

```bash
sudo chmod +x monitor.sh
./monitor.sh
```

