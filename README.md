![Imagem](https://drive.google.com/uc?export=view&id=1q4KTXTHXzJ55r81k-iWLjAIaXT2QPpc_)

[![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://www.java.com/)
[![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)](https://maven.apache.org/)
[![Lombok](https://img.shields.io/badge/Lombok-963484?style=for-the-badge&logo=projectlombok&logoColor=white)](https://projectlombok.org/)
[![Spring Data JPA](https://img.shields.io/badge/Spring_Data_JPA-6DB33F?style=for-the-badge&logo=spring&logoColor=white)](https://spring.io/projects/spring-data-jpa)
[![Bean Validation](https://img.shields.io/badge/Bean_Validation-5A6980?style=for-the-badge&logo=hibernate&logoColor=white)](https://beanvalidation.org/)
[![Oracle](https://img.shields.io/badge/Oracle-F80000?style=for-the-badge&logo=oracle&logoColor=white)](https://www.oracle.com/database/)
[![JDBC](https://img.shields.io/badge/JDBC-007396.svg?style=for-the-badge&logo=java&logoColor=white)](https://www.oracle.com/java/technologies/javase/javase-apis-jdbc.html)
[![API REST](https://img.shields.io/badge/API_REST-000?style=for-the-badge&logo=api&logoColor=white)](https://www.redhat.com/pt-br/topics/api/what-is-a-rest-api)
[![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)](https://swagger.io/)
[![Insomnia](https://img.shields.io/badge/Insomnia-4000BF?style=for-the-badge&logo=insomnia&logoColor=white)](https://insomnia.rest/)

## Knowball API — Deploy Guide

## Pré-requisitos

- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) instalado e autenticado (`az login`)
- [Java 17](https://adoptium.net/) instalado
- [Maven](https://maven.apache.org/) instalado
- [Git](https://git-scm.com/) instalado
- Conta Azure ativa com permissão para criar recursos

---

## 1. Criar o Resource Group

```bash
az group create --name rg-knowball --location southafricanorth
```

---

## 2. Registrar o provedor de Web Apps

```bash
az provider register --namespace Microsoft.Web
```

---

## 3. Criar o SQL Server

```bash
az sql server create \
  --name sql-knowball \
  --resource-group rg-knowball \
  --location southafricanorth \
  --admin-user user-knowball \
  --admin-password Fiap@2tdspa
```

---

## 4. Criar o banco de dados

```bash
az sql db create \
  --resource-group rg-knowball \
  --server sql-knowball \
  --name db-knowball \
  --edition Basic
```

---

## 5. Liberar acesso do Azure ao SQL Server (Firewall)

```bash
az sql server firewall-rule create \
  --resource-group rg-knowball \
  --server sql-knowball \
  --name AllowAllAzureServices \
  --start-ip-address 1.2.3.4 \  # adicione o seu IP
  --end-ip-address 1.2.3.4      # adicione o seu IP
```

> Sem essa regra, o App Service não consegue conectar ao banco e a aplicação não sobe.

---

## 6. Criar o App Service Plan

```bash
az appservice plan create \
  --name planSites \
  --resource-group rg-knowball \
  --sku B1 \
  --is-linux
```

---

## 7. Clonar o repositório

```bash
git clone https://github.com/knowball-oracle/devops-knowball.git
cd devops-knowball
```

---

## 8. Criar o Web App

```bash
az webapp create \
  --name knowball-api \
  --resource-group rg-knowball \
  --plan planSites \
  --runtime "JAVA:17-java17"
```

---

## 9. Configurar variáveis de ambiente

```bash
az webapp config appsettings set \
  --name knowball-api \
  --resource-group rg-knowball \
  --settings \
  "DB_URL=jdbc:sqlserver://sql-knowball.database.windows.net:1433;databaseName=db-knowball;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;" \
  "DB_USERNAME=user-knowball" \
  "DB_PASSWORD=Fiap@2tdspa"
```

---

## 10. Compilar o projeto

```bash
mvn clean package -DskipTests
```

---

## 11. Realizar o deploy

```bash
az webapp deploy \
  --name knowball-api \
  --resource-group rg-knowball \
  --src-path target/knowball-0.0.1-SNAPSHOT.jar \
  --type jar
```

> O deploy pode levar alguns minutos. O terminal ficará exibindo `Status: Starting the site...` — aguarde até aparecer `Deployment successful` ou acompanhe os logs (passo 12).

---

## 12. Acompanhar os logs

```bash
az webapp log tail --name knowball-api --resource-group rg-knowball
```

A aplicação está no ar quando aparecer:

```
Started App in X.XXX seconds (process running for X.XXX)
```

A API estará disponível em:

```
https://knowball-api.azurewebsites.net
```

---

## Testando o CRUD

Importe o arquivo `knowball_crud_insomnia.json` no [Insomnia](https://insomnia.rest/) para testar os endpoints.

### Championship

| Operação | Método | Endpoint |
|---|---|---|
| Criar | `POST` | `/championships` |
| Listar todos | `GET` | `/championships` |
| Buscar por ID | `GET` | `/championships/{id}` |
| Atualizar | `PUT` | `/championships/{id}` |
| Deletar | `DELETE` | `/championships/{id}` |

**Exemplo de body (POST/PUT):**
```json
{
  "name": "Copa SP de Futebol Júnior",
  "category": "SUB_20",
  "year": 2026
}
```

Valores aceitos para `category`: `SUB_13`, `SUB_15`, `SUB_17`, `SUB_20`

---

### Team

| Operação | Método | Endpoint |
|---|---|---|
| Criar | `POST` | `/teams` |
| Listar todos | `GET` | `/teams` |
| Buscar por ID | `GET` | `/teams/{id}` |
| Atualizar | `PUT` | `/teams/{id}` |
| Deletar | `DELETE` | `/teams/{id}` |

**Exemplo de body (POST/PUT):**
```json
{
  "name": "São Paulo FC",
  "city": "São Paulo",
  "state": "SP"
}
```

> **Atenção:** Não é possível deletar um team que possua participações cadastradas. Remova as participações associadas antes de deletar o time.

---

## Documentação da API (Swagger)

A documentação interativa está disponível em:

```
https://knowball-api.azurewebsites.net/swagger-ui.html
```

---

## Limpeza dos recursos

Para remover todos os recursos criados e evitar cobranças:

```bash
az group delete --name rg-knowball --yes --no-wait
```


## Link do vídeo mostrando a criação dos recursos na Azure

> 🎬 Clique no link para assistir no YouTube

[Assista ao vídeo](https://youtu.be/bawC0tl42Gs?si=jGCAK973dvBb2pdf)


## Integrantes

| Nome Completo           | Foto | Responsabilidade |
| ------------------------| ------ | ----- |
| Gabriel Oliveira Rossi  | <a href="https://github.com/GabrielRossi01"><img src="https://avatars.githubusercontent.com/u/179617228?v=4" height="50" style="border-radius:30px;"></a> | Responsável pela concepção, implementação, testes e documentação da API Knowball. |
| Rodrigo Naoki Yamasaki  | <a href="https://github.com/RodrygoYamasaki"><img src="https://avatars.githubusercontent.com/u/182231531?v=4" height="50" style="border-radius:30px;"></a> | Responsável pela participação nas etapas de design e revisão de código. |
| Patrick Castro Quintana | <a href="https://github.com/castropatrick"><img src="https://avatars.githubusercontent.com/u/179931043?v=4" height="50" style="border-radius:30px;"></a> | Responsável pela participação nas etapas de design e revisão de código. |

