# Configuração de Servidor Web com Monitoramento

#### Objetivo:
Desenvolver e testar habilidades em Linux, AWS e automação de processos através da configuração de um ambiente de servidor web monitorado.

---

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
No exemplo acima, mostro as configurações de INBOUND e OUTBOUND utilizadas para que a instância tenha seu grupo de segurança devidamente preparado.

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

Sim, para se conectar a sua AMI da instância é um simples copia e cola. Mas guarde bem o seu link, pois com ele qualquer um acessa e configura sua instância. Lembre-se também de dar permissão para sua chave PEM, usando o comando `chmod` como mostra abaixo.

![Configuração SSH](/imgs/configssh.png)

---

### 1. Instalar o servidor Nginx na EC2:
No seu AMI, digite os comandos de atualização de pacotes e instale o NGINX.

Após a instalação, esse é o caminho para o arquivo da sua página web, onde você pode alterar para realizar as configurações do projeto.

Para verificar se seu NGINX está funcionando apropriadamente:

![Status do NGINX](/imgs/nginx status.png)

---

### 2. Criar uma página HTML simples para ser exibida pelo servidor:
![Página HTML no NGINX](/imgs/PRINT NGINX.png)

---

## Etapa 3: Monitoramento e Notificações

### 1. Criar um script em Bash ou Python para monitorar a disponibilidade do site:
Para a utilização do Script, recomenda-se entrar conectado na máquina, executá-la e editá-la.

Cole o script bash do monitoramento e personalize os campos:
- `SITE_URL`: coloque o IP público da sua instância EC2.
- `LOG_FILE`: crie um arquivo `monitoramento.log` e deixe em branco.
- `DISCORD_WEBHOOK`: crie um webhook no Discord para receber notificações e cole seu link.

![Criação de Webhook](/imgs/criarwebhook.png)
![Webhook Criado](/imgs/webhookcriado.png)

Aqui mostra o código na íntegra com as informações requisitadas acima:
![Script de Monitoramento](/imgs/nano monitor.sh.png)

---

### 2. O script deve:
- Verificar se o site responde corretamente a uma requisição HTTP.
- Criar logs das verificações em `/var/log/monitoramento.log`.
![Log de Monitoramento](/imgs/monitoramento.log rodando.png)

- Enviar uma notificação via Discord, Telegram ou Slack se detectar indisponibilidade.
![Notificação no Discord](/imgs/discordnotifc.png)

---

### 3. Configurar o script para rodar automaticamente a cada 1 minuto usando cron ou systemd timers:
Edite o seu `crontab -e`, e recomendo a utilização da opção 1, o nano, que é mais prático.

![Edição do Crontab](/imgs/crontabnano.png)

Dê permissão de administrador para seu arquivo `monitor.sh` e execute-o com o seguinte comando:

```bash
sudo chmod +x monitor.sh
./monitor.sh