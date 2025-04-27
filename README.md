# README - Infraestrutura de Acesso aos Serviços

[![Contribua com o projeto 🚀](https://img.shields.io/badge/Contribua%20com%20o%20projeto-🚀-brightgreen)](CONTRIBUTING.md)

Este documento descreve as instruções para acesso aos serviços de infraestrutura que estão disponíveis na rede VPN. É necessário que os membros da equipe de infraestrutura disponibilizem acesso à VPN e as credenciais para cada serviço, conforme descrito abaixo.

---

## Pré-requisitos
1. **Acesso à VPN**: Para acessar os serviços mencionados abaixo, é obrigatório estar conectado à rede VPN fornecida pela equipe de infraestrutura.
2. **Solicitação de Credenciais**: As credenciais de cada serviço devem ser solicitadas diretamente à equipe de infraestrutura.

---

## Serviços Disponíveis

### MinIO
- **Descrição**: Serviço de armazenamento de objetos.
- **Endereço**: `10.0.0.70:30006`
- **Acesso**: Conectar-se ao host indicado após autenticação via VPN.

### Superset
- **Descrição**: Plataforma de visualização de dados.
- **Endereço**: [ipea-superset.lappis.rocks](https://ipea-superset.lappis.rocks/)
- **Acesso**: Acessível pelo navegador, após conexão VPN.

### Airflow
- **Descrição**: Gerenciamento de fluxos de trabalho.
- **Endereço**: [ipea-airflow.lappis.rocks](https://ipea-airflow.lappis.rocks/)
- **Acesso**: Acessível pelo navegador, após conexão VPN.

### Analytics Database
- **Descrição**: Banco de dados para armazenamento e consulta de dados analíticos.
- **Endereço**: `10.0.0.73`
- **Porta**: `5432`
- **Acesso**: Necessário configurar cliente de banco de dados (como DBeaver) e estabelecer conexão via VPN.

### JupyterHub
- **Descrição**: Ambiente de notebooks colaborativo.
- **Endereço**: `10.0.0.70:30010`
- **Acesso**: Conectar-se ao endereço do host após autenticação via VPN.

---

## Observações Importantes
- **Configuração do DBeaver**: Para quem preferir, o acesso ao banco de dados Analytics pode ser configurado localmente usando o aplicativo DBeaver, sem a necessidade de uma versão compartilhada.
- **Equipe de Suporte**: Em caso de dúvidas sobre a conexão à VPN ou sobre a configuração dos serviços, entre em contato com a equipe de infraestrutura.

---
