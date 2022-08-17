---
layout: post
title: Sync do GitHub com Bucket S3
subtitle: Objetos AWS S3 controlados por repo GitHub
gh-repo: kledsonhugo/kledsonhugo.github.io
gh-badge: [star, fork, follow]
tags: [AWS, S3, Web, GitHub, Repo, Sync]
comments: true
---
<br/><br/>
Demonstrar como realizar sincronismo de arquivos de um repo GitHub com objetos em um Bucket S3.

> Referências
- [https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html)
- [https://docs.github.com/pt/actions/learn-github-actions](https://docs.github.com/pt/actions/learn-github-actions)

<br/><br/>
## Passo 1

O primeiro passo é criar um Bucket S3 com a opção **Hospedagem de site estático** e com permissão **Publicamente acessível**.

> Instruções em [Site estático com AWS S3](https://kledsonhugo.github.io/2021-03-22-armazenamento-AWS-S3-Site-est%C3%A1tico/)

<br/><br/>
## Passo 2

O segundo passo é configurar um repositório GitHub para sincronizar o conteúdo com o bucket S3.

1. Acesse o [GitHub](https://github.com/).

2. Selcione ou crie um repositório GitHub e acesse o menu **Settings**.

   > Referência [Criar um repositório](https://docs.github.com/pt/github/getting-started-with-github/create-a-repo).

3. Em **Secrets** clique em **Actions**.

4. Para cada variável abaixo clique em **New repository secret**.

   - **`AWS_ACCESS_KEY_ID`** : `{AWS_ACCESS_KEY_ID}`
   - **`AWS_SECRET_ACCESS_KEY`** : `{AWS_SECRET_ACCESS_KEY}`
   - **`AWS_SESSION_TOKEN`** : `{AWS_SESSION_TOKEN}`<br/><br/>

   > Substitua as variáveis `{AWS_ACCESS_KEY_ID}`, `{AWS_SECRET_ACCESS_KEY}` e `{AWS_SESSION_TOKEN}` pelas credenciais de acesso à sua conta AWS.

5. Publique no repositório GitHub o arquivo `.github/workflows/main.yml` conforme exemplo disponível em [Exemplo de GitHub Actions File](https://github.com/kledsonhugo/kledsonhugo.github.io/blob/master/_data/main.yml).

   > Esse arquivo configura o Workflow de sincronismo do repositório GitHub com o bucket S3.

   > Substitua o texto **Bucket-Name** pelo nome do seu Bucket configurado nos passos anteriores.

<br/><br/>
## Passo 3

O terceiro passo é alterar o conteúdo do arquivo index.html no repositório GitHub e observar se o pipeline do GitHub realizou com sucesso o sincronismo com o bucket S3.

1. Publique alterações no arquivo **index.html** do repositório GitHub que será sincronizado com o bucket S3.

   > Caso tenha dúvidas para publicar alterações de conteúdo em repositório GitHub, consulte a doc oficial em [Adicionar um arquivo a um repositório](https://docs.github.com/pt/github/managing-files-in-a-repository/adding-a-file-to-a-repository-using-the-command-line).

2. No repositório GitHub acesse a opção **Actions** e verifique o status da execução do Workflow.

   > Nesse ponto é esperado que o Workflow execute com sucesso e sincronize os arquivos do repositório GitHub com o Bucket S3.

3. Faça login no AWS Console.

4. Em **Serviços** selecione **S3**.

5. Verifique a data e hora dos objetos no bucket.

<br/><br/>
