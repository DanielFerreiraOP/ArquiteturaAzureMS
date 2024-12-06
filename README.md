# DevOps

## Arquitetura 

Para a proposta de arquitetura, utilizei alguns serviços gerenciados da Microsoft Azure que possam ser bastantes resilientes devido a complexidade do cenário.

### Fluxo

No arquivo [arquitetura.drawio](./diagramas-drawio/arquitetura.drawio), temos, em um cenário macro, a utilização de uma única subscrição com duas redes conectadas por peering:

1. Primeira Rede:

    - Contém um WAF (Web Application Firewall) para proteção contra ataques e um Application Gateway com IP público para acesso externo.
    - As requisições são encaminhadas ao API Management, que gerencia as rotas e políticas de segurança das APIs.

1. Segunda Rede:

    - Composta por três sub-redes:
        - Sub-rede de Containers: Abriga os microserviços e implementa regras de backend para filtragem de IPs nas APIs `auth, order, e checkout`. Para as APIs `order e checkout`, há proteção adicional via token JWT.
        - Sub-rede do GitLab Self-Hosted: Hospeda o código-fonte, a pipeline de CI/CD e garante maior segurança e controle pela equipe DevOps. As pipelines realizam o build das imagens e enviam-nas ao Container Registry, conectado à rede via private endpoint.
        - Sub-rede de Banco de Dados: Contém o Azure Cosmos DB, utilizado pelos microserviços da sub-rede de containers.


### Git Flow

No arquivo [gitflow.drawio](./diagramas-drawio/gitflow.drawio), o fluxo de trabalho e versionamento de código é ilustrado.

1. Branch Principal: O código parte da branch main, que evolui para develop, permitindo a criação de novas branches filhas, chamadas de features.

1. Branch Develop: Abriga todas as novas funciolidades e evoluções do produto.

1. Branches Feature: Nessas branches, os desenvolvedores trabalham em épicos e histórias de usuário, gerando novas funcionalidades. Após o desenvolvimento, o código retorna para develop.

1. Branch de Release: Serve como intermediária, onde é gerada uma nova versão em produção. Após validações e testes, o código é integrado à branch main.

1. Hotfix: Em caso de correções urgentes, uma branch é criada diretamente de main. Após a correção, a mudança é integrada a main e sincronizada com develop para manter a compatibilização.

### CI/CD

No arquivo [cicd.drawio](./diagramas-drawio/cicd.drawio), são apresentadas pipelines de integração e deploy, otimizando entregas e garantindo qualidade.

1. Pipeline de Integração em Merge Request:

    - No momento de integração de código na branch develop, esta pipeline executa:
        - Testes unitários e de integração.
        - Análise estática com o SonarQube para detecção de vulnerabilidades.
        - Testes de carga com K6.
    - Após aprovação por um revisor, o código é integrado.

1. Pipeline de Deploy para Ambiente de Homologação (QA/Dev):

    - Durante o processo de trabalho dos desenvolvedores em novas funcionalidades, o proprio desenvolvedor e QA são os primeiros usuários e necessitam da realização de testes específicos, que muitas vezes não podem ser automatizados, por conta de arquitetura do produto, ou tipo de serviço e teste. Então ter um ambiente semelhante ao produtivo, auxilia na agilidade e coerências dos testes. Essa pipeline executa:
        - Build da tecnologia.
        - Deploy no servidor.


1. Pipeline de Geração de Release:

    - Após validações, esta pipeline gera uma nova versão e integra o código à branch principal (main). Em projetos grandes, pode ser necessário revalidar o código antes da integração final, especialmente em cenários com muitos hotfixes, mas essa opção pode ser para grande volumes de entregas.

1. Pipeline de Deploy para Produção:

    - Pode ser acionada manualmente ou por gatilhos, realizando:
        - Deploy da nova versão em produção.
        - Reinício do container.
    - As versões são armazenadas no Container Registry, facilitando reversões rápidas para versões anteriores, caso necessário.