# 🚀 Aplicação Maker/Webrun no OKD

Imagem de container certificada Red Hat para deploy da aplicação **Webrun/CONSIGLOG** no OKD, baseada em **JBoss Web Server 5.8 + OpenJDK 8 + Apache Tomcat 9**.

![Red Hat](https://img.shields.io/badge/Red%20Hat-JWS%205.8-EE0000?logo=redhat&logoColor=white)
![Java](https://img.shields.io/badge/Java-OpenJDK%208-007396?logo=openjdk&logoColor=white)
![Tomcat](https://img.shields.io/badge/Tomcat-9-F8DC75?logo=apachetomcat&logoColor=black)
![SQL Server](https://img.shields.io/badge/JDBC-SQL%20Server%2013.2.0-CC2927?logo=microsoftsqlserver&logoColor=white)

---

## 📋 Stack

| Componente | Versão |
|---|---|
| Base Image | `jboss-webserver-5/jws58-openjdk8-openshift-rhel8:latest` |
| JBoss Web Server | 5.8 |
| Apache Tomcat | 9 |
| Java | OpenJDK 8 |
| Driver JDBC | `mssql-jdbc-13.2.0.jre8` |

> ✅ **Driver JDBC `13.2.0.jre8`** — release estável e atual para JRE 8.
> - 🐙 [GitHub — mssql-jdbc v13.2.0](https://github.com/Microsoft/mssql-jdbc/releases/tag/v13.2.0)
> - 📄 [Microsoft — Release Notes JDBC Driver](https://learn.microsoft.com/pt-br/sql/connect/jdbc/release-notes-for-the-jdbc-driver)

---

## 📁 Estrutura de Arquivos

Antes de realizar o build, certifique-se de que os seguintes arquivos estão presentes no diretório raiz:

```
.
├── Dockerfile
├── ROOT.war                        # Motor Webrun (deploy principal)
├── CONSIGLOG_API.wfre              # Módulo de sistema Webrun
├── CONSIGLOG_API.jar               # Biblioteca da API CONSIGLOG
└── mssql-jdbc-13.2.0.jre8.jar      # Driver JDBC para SQL Server
```

---

## ⚙️ Variáveis de Ambiente

| Variável | Valor | Descrição |
|---|---|---|
| `WEBRUN_SYSTEMS` | `/home/jboss/softwell/webrun/systems` | Diretório dos sistemas Webrun |
| `DEPLOY_DIR` | `/deployments` | Diretório de deploy prioritário do OKD |
| `TOMCAT_HOME` | `/opt/jws-5.8/tomcat` | Instalação do Tomcat |

---

## 🗂️ Diretórios Internos do Container

| Caminho | Conteúdo |
|---|---|
| `/deployments/ROOT.war` | Aplicação principal |
| `/opt/jws-5.8/tomcat/lib/` | Driver JDBC SQL Server |
| `/home/jboss/softwell/webrun/systems/` | Módulos do sistema Webrun |

---

## 📖 Explicação do Dockerfile

### Etapa 1 — Imagem Base
```dockerfile
FROM registry.redhat.io/jboss-webserver-5/jws58-openjdk8-openshift-rhel8:latest
```
Utiliza a imagem certificada Red Hat com **JWS 5.8 + OpenJDK 8**, otimizada para o OKD/OpenShift. Inclui o Tomcat 9 pré-configurado e compatível com as políticas de segurança da plataforma.

### Etapa 2 — Limpeza e Preparação
```dockerfile
RUN rm -rf ${TOMCAT_HOME}/webapps/* && \
    rm -rf ${DEPLOY_DIR}/* && \
    mkdir -p ${WEBRUN_SYSTEMS}
```
Garante um ambiente limpo antes do deploy, removendo arquivos residuais dos diretórios `webapps/` e `deployments/`. Cria também a estrutura de diretórios necessária para os sistemas Webrun.

### Etapa 3 — Driver JDBC
```dockerfile
COPY mssql-jdbc-13.2.0.jre8.jar ${TOMCAT_HOME}/lib/
```
Instala o driver JDBC do SQL Server na `lib/` global do Tomcat, tornando-o disponível para todas as aplicações do container.

### Etapa 4 — Deploy da Aplicação
```dockerfile
COPY ROOT.war ${DEPLOY_DIR}/ROOT.war
COPY CONSIGLOG_API.wfre ${WEBRUN_SYSTEMS}/
COPY CONSIGLOG_API.jar ${WEBRUN_SYSTEMS}/
```
Copia os artefatos para seus destinos. O `ROOT.war` é o motor Webrun deployado no diretório prioritário do OKD. Os arquivos `.wfre` e `.jar` são os módulos do sistema CONSIGLOG.

### Etapa 5 — Permissões
```dockerfile
RUN chown -R 185:0 ${DEPLOY_DIR} ${WEBRUN_SYSTEMS} ${TOMCAT_HOME}/webapps && \
    chmod -R g+rwX ${DEPLOY_DIR} ${WEBRUN_SYSTEMS} ${TOMCAT_HOME}/webapps
```
Define a propriedade para o usuário `185` com grupo `0` (root) e aplica permissões `g+rwX`. Necessário para compatibilidade com o OKD, que executa containers com **UIDs arbitrários** dentro do grupo root.

### Etapa 6 — Usuário e Inicialização
```dockerfile
USER 185
CMD ["/opt/jws-5.8/tomcat/bin/launch.sh"]
```
Executa o container como usuário não-root `185`, seguindo as boas práticas de segurança do OKD. O `launch.sh` é o ponto de entrada oficial da imagem JWS, responsável por inicializar o Tomcat com todas as configurações da plataforma.

---

## 🛡️ Segurança

| Prática | Detalhe |
|---|---|
| Usuário não-root | Container executa com `UID 185`, compatível com SCC do OKD |
| Grupo root | `GID 0` com `g+rwX` garante suporte a UIDs arbitrários |
| Ambiente limpo | `webapps/` e `deployments/` são limpos no build para evitar conflitos |
| Imagem certificada | Base oficial Red Hat, com suporte e atualizações de segurança |

---

## 🔧 Inicialização

O container é iniciado via script oficial do JWS:

```bash
/opt/jws-5.8/tomcat/bin/launch.sh
```

---

## 🔗 Referências da Base Image

A imagem base utilizada é a **`jws58-openjdk8-openshift-rhel8`**, publicada no Red Hat Ecosystem Catalog na versão `5.8.5-3`. Abaixo os links para acompanhar releases, vulnerabilidades e documentação oficial:

| Recurso | Link |
|---|---|
| 📦 Red Hat Ecosystem Catalog | [jws58-openjdk8-openshift-rhel8](https://catalog.redhat.com/en/software/containers/jboss-webserver-5/jws58-openjdk8-openshift-rhel8/660f3adc50d2fb6af01be264) |
| 📋 Release Notes JWS 5.8 | [Red Hat Documentation](https://docs.redhat.com/en/documentation/red_hat_jboss_web_server/5.8/html-single/red_hat_jboss_web_server_5.8_release_notes/index) |
| 📖 Guia OpenShift JWS 5.8 | [Red Hat Documentation](https://docs.redhat.com/en/documentation/red_hat_jboss_web_server/5.8/html-single/red_hat_jboss_web_server_for_openshift/index) |
| 🐙 Imagens OCI (GitHub) | [jboss-container-images/jboss-webserver-5-openshift-image](https://github.com/jboss-container-images/jboss-webserver-5-openshift-image) |

> ⚠️ **Atenção:** No Red Hat Ecosystem Catalog é possível verificar o status de segurança da imagem (CVEs), o Containerfile e as tags de versão disponíveis. Recomenda-se monitorar esse painel regularmente.


✍️ Autoria
Documento elaborado por Matheus Corteletti "Wise" em 10/03/2026.
