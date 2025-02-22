## ✅ **Passo a passo para configurar alertas do Zabbix via e-mail**

### 📌 **1) Configurar o Servidor de E-mail no Zabbix**
1. No **Zabbix**, vá até **Administração → Tipos de Mídia**.  
2. Clique em **Criar tipo de mídia** e preencha:  
   - **Nome**: `Email`  
   - **Tipo**: SMTP  
   - **SMTP Server**: O servidor SMTP do seu provedor de e-mail (exemplo: `smtp.gmail.com` para Gmail, `smtp.office365.com` para Outlook).  
   - **SMTP Port**: Geralmente `465` (SSL) ou `587` (TLS).  
   - **Authentication**:  
     - **Usuário SMTP**: Seu e-mail (exemplo: `seuemail@gmail.com`).  
     - **Senha SMTP**: A senha ou **App Password** do e-mail (alguns provedores exigem isso).  
   - **E-mail do remetente**: O e-mail que será usado para enviar as notificações.  
3. Clique em **Atualizar**.  

---

### 📌 **2) Criar um Usuário para Receber os Alertas**
Agora, crie um usuário no Zabbix para receber os alertas por e-mail.  

1. Vá em **Administração → Usuários** e clique em **Criar usuário**.  
2. Defina um **Nome de usuário** (exemplo: `Alertas Zabbix`).  
3. Vá para a aba **Mídia** e clique em **Adicionar**.  
   - **Tipo**: Selecione `Email`.  
   - **Enviar para**: Insira o e-mail do destinatário (exemplo: `admin@empresa.com`).  
   - **Quando ativo**: Sempre.  
   - **Gravidade**: Marque os tipos de alertas que deseja receber.  
4. Clique em **Adicionar** e depois em **Atualizar**.  

---

### 📌 **3) Criar uma Ação para Enviar Alertas**
Agora, configure o Zabbix para disparar e-mails quando houver incidentes.  

1. Vá em **Configuração → Ações → Criar ação**.  
2. No campo **Nome**, coloque algo como `Alerta por E-mail`.  
3. Vá até a aba **Condições** e escolha quando deseja receber alertas.  
4. Na aba **Operações**, clique em **Adicionar nova operação** e configure:  
   - **Enviar para usuários**: Escolha o usuário que você criou para receber os alertas por e-mail.  
   - **Enviar via**: `Email`.  
   - **Mensagem**:  
     ```
     🚨 [Zabbix Alert] 🚨  
     Host: {HOST.NAME}  
     Problema: {TRIGGER.NAME}  
     Gravidade: {TRIGGER.SEVERITY}  
     Tempo: {EVENT.DATE} {EVENT.TIME}
     ```
5. Clique em **Adicionar** e depois em **Atualizar**.  

---

### 📌 **4) Testar o Envio de E-mails**
Agora, faça um teste para garantir que os e-mails estão sendo enviados corretamente:  

1. Vá em **Administração → Tipos de Mídia → Email**.  
2. Clique em **Testar** e envie um e-mail de teste.  
3. Se tudo estiver certo, você receberá o e-mail no endereço configurado.  

---

### 🎯 **Conclusão**
Agora, o Zabbix enviará alertas automaticamente por e-mail sempre que um problema ocorrer.
