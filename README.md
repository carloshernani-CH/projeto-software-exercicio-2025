# API de Filmes - Back-End

API REST para gerenciamento de catálogo de filmes com autenticação Auth0.

## Funcionalidades

- Cadastro de filmes (nome, descrição, nota 0-5, diretor)
- Listagem de todos os filmes
- Exclusão de filmes (apenas administradores)
- Autenticação e autorização via Auth0

## Tecnologias

- Java 21
- Spring Boot 3.5.7
- Spring Security + OAuth2 Resource Server
- MySQL 8.0
- Docker

## Configuração Local

### Pré-requisitos

- Java 21
- Maven
- MySQL 8.0 (ou Docker)

### Variáveis de Ambiente

```bash
DB_URL=jdbc:mysql://localhost:3306/filmes_db?createDatabaseIfNotExist=true
DB_USERNAME=root
DB_PASSWORD=sua_senha
```

### Executar Localmente

```bash
# Com Maven
mvn spring-boot:run

# Com Docker Compose
docker-compose up -d

# Ou apenas MySQL com Docker
docker network create rede
docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=filmes_db --network=rede -p 3306:3306 -d mysql:8.0
```

A API estará disponível em `http://localhost:8080`

## Endpoints

### GET /filmes
Lista todos os filmes (requer autenticação)

### POST /filmes
Cadastra um novo filme (requer autenticação)

**Body:**
```json
{
  "nome": "Nome do Filme",
  "descricao": "Descrição",
  "nota": 5,
  "diretor": "Nome do Diretor"
}
```

### DELETE /filmes/{id}
Exclui um filme (requer permissão de ADMIN)

## Deploy na AWS

### Configuração de Secrets no GitHub

Acesse: `Settings > Secrets and variables > Actions > New repository secret`

Configure os seguintes secrets:

1. **DOCKERHUB_USERNAME**: Seu usuário do Docker Hub
2. **DOCKERHUB_TOKEN**: Token de acesso do Docker Hub
3. **HOST_TEST**: IP público da instância EC2 AWS
4. **KEY_TEST**: Chave privada SSH (.pem) da EC2
5. **DB_PASSWORD**: Senha do banco MySQL na AWS

### Preparar EC2

Conecte-se à sua instância EC2:

```bash
ssh -i chave.pem ubuntu@SEU_IP_EC2
```

Instale Docker e Docker Compose:

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Docker
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Criar rede Docker
docker network create rede

# Relogar para aplicar as permissões
exit
```

### Deploy Automático

O deploy é feito automaticamente via GitHub Actions ao fazer push na branch `main`:

```bash
git add .
git commit -m "Deploy: implementação da API de filmes"
git push origin main
```

O pipeline irá:
1. Compilar o projeto com Maven
2. Construir a imagem Docker
3. Fazer push para o Docker Hub
4. Conectar via SSH na EC2
5. Fazer pull da nova imagem
6. Parar e remover o container antigo
7. Iniciar o novo container

### Verificar Deploy

```bash
# Na EC2, verificar containers rodando
docker ps

# Ver logs da aplicação
docker logs filmes-app -f

# Testar endpoint (substitua SEU_IP_EC2)
curl http://SEU_IP_EC2:8080/filmes
```

## Configuração Auth0

1. Acesse [Auth0 Dashboard](https://manage.auth0.com/)
2. Crie uma API com identifier: `https://dev-vlbhbshl7b1pxe8e.us.auth0.com/api/v2/`
3. Configure permissões:
   - `ADMIN` ou `delete:filmes` para exclusão de filmes
4. Atribua permissões aos usuários no painel de usuários do Auth0

## Estrutura do Projeto

```
src/main/java/br/edu/insper/exercicio/
├── config/
│   └── SecurityConfig.java        # Configuração de segurança
├── filmes/
│   ├── Filme.java                 # Entidade JPA
│   ├── FilmeRepository.java       # Repository
│   ├── FilmeService.java          # Serviço de negócio
│   └── FilmeController.java       # Controller REST
└── ExercicioApplication.java      # Main
```

## Licença

Projeto acadêmico - Insper