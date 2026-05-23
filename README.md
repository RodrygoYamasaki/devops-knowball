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

## Pipeline CI/CD — Azure DevOps

O restante da infraestrutura e o deploy da aplicação são gerenciados pela pipeline no Azure DevOps.

### Configuração do Azure DevOps

**1. Criar a organização e o projeto**
- Crie uma organização.
- Crie um projeto chamado `knowball` (Private, Git, Agile)

**2. Criar o Service Connection com o Azure**
- Vá em **Project Settings → Pipelines → Service connections**
- Clique em **Create service connection → Azure Resource Manager**
- Preencha:
  - **Resource group:** `rg-knowball`
  - **Service Connection Name:** `azure-knowball`
  - Marque **Grant access permission to all pipelines**

**3. Criar a pipeline**
- Vá em **Pipelines → Create Pipeline → GitHub**
- Selecione o repositório
- Configure your repository -> Starter pipeline
- Substitua o conteúdo gerado pelo `azure-pipelines.yml` disponível na raiz do repositório e clique em **Save and run**

---

## Endpoints da API

A API estará disponível em:

```
https://app-knowball.azurewebsites.net
```

Documentação interativa (Swagger):

```
https://app-knowball.azurewebsites.net/swagger-ui.html
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

## Limpeza dos recursos

Para remover todos os recursos criados e evitar cobranças:

```bash
az group delete --name rg-knowball --yes --no-wait
```


## Link do vídeo mostrando a criação dos recursos na Azure

> 🎬 Clique no link para assistir no YouTube

[Assista ao vídeo](https://youtu.be/zLJqqWJSbko?si=ErkBA8X2R_Xu6IU0)


## Integrantes

| Nome Completo           | Foto | Responsabilidade |
| ------------------------| ------ | ----- |
| Gabriel Oliveira Rossi  | <a href="https://github.com/GabrielRossi01"><img src="https://avatars.githubusercontent.com/u/179617228?v=4" height="50" style="border-radius:30px;"></a> | Responsável pela concepção, implementação, testes e documentação da API Knowball. |
| Rodrigo Naoki Yamasaki  | <a href="https://github.com/RodrygoYamasaki"><img src="https://avatars.githubusercontent.com/u/182231531?v=4" height="50" style="border-radius:30px;"></a> | Responsável pela participação nas etapas de design e revisão de código. |
| Patrick Castro Quintana | <a href="https://github.com/castropatrick"><img src="https://avatars.githubusercontent.com/u/179931043?v=4" height="50" style="border-radius:30px;"></a> | Responsável pela participação nas etapas de design e revisão de código. |
