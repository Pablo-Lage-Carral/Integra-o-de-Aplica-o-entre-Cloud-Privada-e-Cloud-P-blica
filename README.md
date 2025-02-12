# **Documentação: Integração de Aplicação entre Cloud Privada e Cloud Pública**

## **Descrição do Projeto**
Este projeto tem como objetivo demonstrar a implementação de uma solução de integração entre uma cloud privada e uma cloud pública, garantindo a comunicação segura entre os dois ambientes. A aplicação consiste em um **frontend** hospedado na **cloud pública (RedHat Sandbox)** e um **backend (banco de dados)** hospedado na **cloud privada (RedHat Playground)**. Para interligar esses ambientes de forma segura e eficiente, utilizamos o **Skupper**, que permite a interconexão de clusters Kubernetes através de chaves de criptografia.

---

## **Objetivo**
Criar uma aplicação distribuída entre uma cloud privada e uma cloud pública, garantindo a comunicação segura entre os dois ambientes. Para isso, utilizamos o **Skupper**, que permite a interconexão entre os clusters de forma segura.

---

## **Arquitetura da Solução**
- **Frontend**: Hospedado na cloud pública **RedHat Sandbox**.
- **Backend (Banco de Dados)**: Hospedado na cloud privada **RedHat Playground**.
- **Comunicação**: Feita através do **Skupper**, que permite a interconexão entre os clusters de forma segura.
![Captura de tela 2025-02-12 142343](https://github.com/user-attachments/assets/29ac9d48-4013-4c47-900d-e39b51a2c0c1)

---

## **1. Configuração do Cluster na Cloud Pública**
### **1.1. Criando o Deployment do Frontend**
Execute o seguinte comando para aplicar a configuração do frontend:
```sh
oc apply -f https://raw.githubusercontent.com/rpscodes/Patient-Portal-Deployment/main/patient-portal-frontend-deploy.yaml
```

### **1.2. Instalando o Skupper na Cloud Pública**
Instale o Skupper com os seguintes comandos:
```sh
curl https://skupper.io/install.sh | sh
export PATH="/home/user/.local/bin:$PATH"
```

Inicie o Skupper no cluster público:
```sh
skupper init --site-name public --enable-console --enable-flow-collector --console-auth unsecured
```

Verifique o status do Skupper:
```sh
skupper status
```

Após a instalação, o console do Skupper estará disponível no seguinte endereço:
```
https://skupper-rm359176-dev.apps.rm3.7wse.p1.openshiftapps.com
```

Gere um token de autenticação para conexão entre os clusters:
```sh
skupper token create secret.token
```

Exiba o conteúdo do token gerado:
```sh
cat secret.token
```

Copie esse token, pois ele será utilizado na cloud privada.

---

## **2. Configuração do Cluster na Cloud Privada**
### **2.1. Criando o Namespace e Deploy do Banco de Dados**
Crie um novo namespace para o ambiente privado:
```sh
oc new-project private
```

Crie a aplicação do banco de dados:
```sh
oc new-app quay.io/redhatintegration/patient-portal-database:devnation --name=database
```

Aplique a configuração do processador de pagamento:
```sh
oc apply -f https://raw.githubusercontent.com/jbossdemocentral/service-interconnect-sandbox-demo/main/payment-processor-deployment.yaml
```

Verifique se os pods estão rodando corretamente:
```sh
oc get pods
```

### **2.2. Instalando o Skupper na Cloud Privada**
```sh
curl https://skupper.io/install.sh | sh
```

Inicie o Skupper no cluster privado:
```sh
skupper init --ingress none --router-mode edge --enable-console=false
```

---

## **3. Estabelecendo a Conexão entre Cloud Privada e Cloud Pública**
### **3.1. Vinculando os Clusters com o Token**
Copie o token gerado na cloud pública e salve-o na cloud privada, no diretório **.kube**.

Agora, crie o link entre os clusters utilizando o token:
```sh
skupper link create /root/.kube/secret.token
```

### **3.2. Configurando os Serviços para a Interconexão**
Annotate os serviços para configurar o roteamento:
```sh
kubectl annotate service database skupper.io/proxy=tcp
kubectl annotate service payment-processor skupper.io/proxy=http
```

---

## **Conclusão**
Com essa configuração, conseguimos estabelecer a comunicação segura entre os ambientes de cloud pública e cloud privada utilizando o Skupper. Dessa forma, o frontend pode interagir com o backend mesmo estando em diferentes infraestruturas de nuvem.

