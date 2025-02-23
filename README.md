
# **Configuração do Zabbix com Alertas via E-mail e Containerização**

![GitHub](https://img.shields.io/badge/Zabbix-6.0-blue) ![Docker](https://img.shields.io/badge/Docker-Supported-blue)

Este projeto demonstra como configurar um servidor Zabbix completamente funcional com alertas via e-mail, utilizando Docker para containerizar todos os componentes necessários (banco de dados, servidor Zabbix, interface web e agente). O objetivo é fornecer um ambiente fácil de configurar e manter para monitoramento de infraestrutura.

---

## **Índice**

1. [Descrição do Projeto](#descrição-do-projeto)
2. [Tecnologias Utilizadas](#tecnologias-utilizadas)
3. [Pré-requisitos](#pré-requisitos)
4. [Passo a Passo para Configurar o Zabbix](#passo-a-passo-para-configurar-o-zabbix)
   - [Configuração do Servidor de E-mail no Zabbix](#configuração-do-servidor-de-e-mail-no-zabbix)
   - [Criar um Usuário para Receber os Alertas](#criar-um-usuário-para-receber-os-alertas)
   - [Criar uma Ação para Enviar Alertas](#criar-uma-ação-para-enviar-alertas)
   - [Testar o Envio de E-mails](#testar-o-envio-de-e-mails)
5. [Containerização do Zabbix com Docker](#containerização-do-zabbix-com-docker)
6. [Contribuição](#contribuição)
7. [Licença](#licença)

---

## **Descrição do Projeto**

Este projeto fornece um guia completo para configurar o Zabbix, uma ferramenta de monitoramento de infraestrutura open-source, com alertas via e-mail. Além disso, o ambiente é containerizado usando Docker, permitindo uma implantação rápida e fácil em qualquer sistema que suporte Docker.

---

## **Tecnologias Utilizadas**

- **Zabbix**: Ferramenta de monitoramento de infraestrutura.
- **MariaDB**: Banco de dados utilizado pelo Zabbix.
- **Docker**: Plataforma para containerização do ambiente Zabbix.
- **SMTP**: Protocolo usado para enviar alertas via e-mail.
- **Nginx**: Servidor web para a interface do Zabbix.

---

## **Pré-requisitos**

Antes de iniciar, certifique-se de ter as seguintes ferramentas instaladas:

- **Docker**: Para executar os containers.
- **Docker Compose**: Para gerenciar os serviços do Zabbix.
- **Um provedor de e-mail SMTP**: Como Gmail ou Outlook, para enviar alertas por e-mail.

Você pode verificar se o Docker está instalado corretamente executando:
```bash
docker --version
```

---

## **Passo a Passo para Configurar o Zabbix**

### **Configuração do Servidor de E-mail no Zabbix**

1. No Zabbix, vá até **Administração → Tipos de Mídia**.
2. Clique em **Criar tipo de mídia** e preencha:
   - Nome: `Email`
   - Tipo: `SMTP`
   - SMTP Server: O servidor SMTP do seu provedor de e-mail (ex.: `smtp.gmail.com` para Gmail).
   - SMTP Port: Geralmente `465` (SSL) ou `587` (TLS).
   - Autenticação:
     - Usuário SMTP: Seu e-mail (ex.: `seuemail@gmail.com`).
     - Senha SMTP: A senha ou App Password do e-mail.
   - E-mail do remetente: O e-mail que será usado para enviar as notificações.
3. Clique em **Atualizar**.

---

### **Criar um Usuário para Receber os Alertas**

1. Vá em **Administração → Usuários** e clique em **Criar usuário**.
2. Defina um nome de usuário (ex.: `Alertas Zabbix`).
3. Na aba **Mídia**, clique em **Adicionar**:
   - Tipo: `Email`.
   - Enviar para: Insira o e-mail do destinatário (ex.: `admin@empresa.com`).
   - Quando ativo: `Sempre`.
   - Gravidade: Marque os tipos de alertas que deseja receber.
4. Clique em **Adicionar** e depois em **Atualizar**.

---

### **Criar uma Ação para Enviar Alertas**

1. Vá em **Configuração → Ações → Criar ação**.
2. No campo **Nome**, coloque algo como `Alerta por E-mail`.
3. Na aba **Condições**, escolha quando deseja receber alertas.
4. Na aba **Operações**, clique em **Adicionar nova operação**:
   - Enviar para usuários: Escolha o usuário criado anteriormente.
   - Enviar via: `Email`.
   - Mensagem:
     ```
     🚨 [Zabbix Alert] 🚨  
     Host: {HOST.NAME}  
     Problema: {TRIGGER.NAME}  
     Gravidade: {TRIGGER.SEVERITY}  
     Tempo: {EVENT.DATE} {EVENT.TIME}
     ```
5. Clique em **Adicionar** e depois em **Atualizar**.

---

### **Testar o Envio de E-mails**

1. Vá em **Administração → Tipos de Mídia → Email**.
2. Clique em **Testar** e envie um e-mail de teste.
3. Verifique se o e-mail foi recebido no endereço configurado.

---

## **Containerização do Zabbix com Docker**

O arquivo `docker-compose.yml` abaixo define todos os serviços necessários para rodar o Zabbix em containers Docker.

```yaml
version: '3.7'

services:
  # Serviço do banco de dados (MariaDB)
  zabbix-db:
    image: mariadb:10.6
    container_name: zabbix-db
    networks:
      - zabbix-net
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
    volumes:
      - zabbix-db-data:/var/lib/mysql
    restart: always

  # Serviço do servidor Zabbix
  zabbix-server:
    image: zabbix/zabbix-server-mysql:ubuntu-6.0-latest
    container_name: zabbix-server
    networks:
      - zabbix-net
    depends_on:
      - zabbix-db
    environment:
      DB_SERVER_HOST: zabbix-db
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      MYSQL_DATABASE: zabbix
    ports:
      - "10051:10051"
    restart: always

  # Serviço da interface web do Zabbix
  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:ubuntu-6.0-latest
    container_name: zabbix-web
    networks:
      - zabbix-net
    depends_on:
      - zabbix-server
      - zabbix-db
    environment:
      DB_SERVER_HOST: zabbix-db
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      MYSQL_DATABASE: zabbix
      ZBX_SERVER_HOST: zabbix-server
    ports:
      - "8080:8080"
    restart: always

  # Serviço do agente Zabbix (opcional)
  zabbix-agent:
    image: zabbix/zabbix-agent:ubuntu-6.0-latest
    container_name: zabbix-agent
    networks:
      - zabbix-net
    environment:
      ZBX_HOSTNAME: zabbix-agent
      ZBX_SERVER_HOST: zabbix-server
    restart: always

# Configuração da rede personalizada
networks:
  zabbix-net:
    driver: bridge

# Volumes para persistência de dados
volumes:
  zabbix-db-data:
```

### **Como Executar**

1. Salve o arquivo acima como `docker-compose.yml`.
2. Execute o comando abaixo para iniciar os containers:
   ```bash
   docker-compose up -d
   ```
3. Acesse a interface web do Zabbix em:
   ```
   http://localhost:8080
   ```

---

## **Contribuição**

Contribuições são bem-vindas! Siga estas etapas:

1. Faça um fork deste repositório.
2. Crie uma nova branch:
   ```bash
   git checkout -b feature/nova-funcionalidade
   ```
3. Faça suas alterações e commit:
   ```bash
   git commit -m "Adiciona nova funcionalidade"
   ```
4. Envie suas alterações:
   ```bash
   git push origin feature/nova-funcionalidade
   ```
5. Abra um Pull Request neste repositório.

---

## **Licença**

Este projeto está licenciado sob a **MIT License**. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## **Contato**

Se tiver dúvidas ou sugestões, entre em contato:

- **Nome**: Luan Castro
- **Email**: luandecastrosilva@gmail.com
- **LinkedIn**: [linkedin.com/in/seu-perfil](https://www.linkedin.com/in/luancastrosilva/)
- **GitHub**: [github.com/seu-usuario](https://github.com/luangitdev)

