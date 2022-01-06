# bitbucket-codepipeline-codebuild-eks
ci/cd project with the tools mentioned in the title

Esse projeto tem como objetivo mostar os passos necessarios para para criar uma estrutura de ci/cd utilizando o bitbucket, codepipeline, codebuild, ECS e EKS.


# Pré requisitos
Previamente para execução desse projeto, se faz necessario que voce ja tenha as seguintes coisas:

Account AWS

Account Bitbucket

Cluster EKS



# Descrições de serviços que serão utilizados

**Pipeline de AWS**

Totalmente Gerenciado

Construído para Escala

Integrar com serviços AWS

Seguro

**Bitbucket**

O Bitbucket é uma ferramenta de hospedagem e colaboração de código baseada em Git, criada para equipes profissionais de engenharia de software e gestão de projetos. A marca BitBucket foi adquirida pela Atlassian em 2010, o que garante às suas ferramentas integração total com os demais serviços da empresa e ainda workflows do Jira e do Trello.

Desenvolvedor -> Bitbucket -> AWS CodeBuild -> Teste -> Implementar


**AWS CodeBuild**

O AWS CodeBuild é um serviço de integração contínua totalmente gerenciado que compila o código-fonte, executa testes e produz pacotes de software prontos para implantação. Com CodeBuild, você não precisa provisionar, gerenciar e dimensionar seus próprios servidores de construção. CodeBuild escala continuamente e processa várias compilações simultaneamente, para que suas compilações não fiquem esperando em uma fila. Você pode começar rapidamente usando ambientes de construção predefinidos ou pode criar ambientes de construção personalizados que usam suas próprias ferramentas de construção. Com CodeBuild, você é cobrado por minuto pelos recursos de computação que usa.

*Serviço AWS CI*

Semelhante a Jenkins
Escala automaticamente
Desenvolvedor -> Repositório de código -> AWS CodeBuild -> Teste -> Implementar

**AWS CodePipeline**

O AWS CodePipeline é um serviço de entrega contínua totalmente gerenciado que ajuda a automatizar seus pipelines de lançamento para aplicativos rápidos e confiáveis ​​e atualizações de infraestrutura. CodePipeline automatiza as fases de construção, teste e implantação de seu processo de lançamento sempre que houver uma alteração no código, com base no modelo de lançamento que você definir. Isso permite que você forneça recursos e atualizações de forma rápida e confiável. Você pode integrar facilmente o AWS CodePipeline com serviços de terceiros, como GitHub ou com seu próprio plug-in personalizado.

Fase de construção, teste e implantação

Orquestrar todos os componentes de Bitbucket e CodeBuild

Entrega Rápida

Totalmente Gerenciado

Fácil integração com AWS e ferramentas de terceiros
