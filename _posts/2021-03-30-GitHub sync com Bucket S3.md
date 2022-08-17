---
layout: post
title: Sync do GitHub com Bucket S3
subtitle: Objetos AWS S3 controlados por repo GitHub
gh-repo: kledsonhugo/kledsonhugo.github.io
gh-badge: [star, fork, follow]
tags: [AWS, S3, Web, GitHub, Repo, Sync]
comments: true
---
Demonstrar como realizar sincronismo de arquivos em um repo GitHub com objetos em um Bucket S3.

> Referências
- [https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html)
- [https://docs.github.com/pt/actions/learn-github-actions](https://docs.github.com/pt/actions/learn-github-actions)


## Passo 1

O primeiro passo é criar o Bucket S3 com a opção **site estático**.

> Instruções em [https://kledsonhugo.github.io/2021-03-22-armazenamento-AWS-S3-Site-est%C3%A1tico/](https://kledsonhugo.github.io/2021-03-22-armazenamento-AWS-S3-Site-est%C3%A1tico/)

## Passo 2

O segundo passo é configurar um repositório GitHub para sincronizar o conteúdo com o bucket S3.

1. Acesse o GitHub [https://github.com/](https://github.com/).

2. Selcione ou crie um repositório GitHub e acesse o menu **Settings**.

   > Referência [https://docs.github.com/pt/github/getting-started-with-github/create-a-repo](https://docs.github.com/pt/github/getting-started-with-github/create-a-repo).

3. Em **Secrets** clique em **New repository secret** e adicione as variáveis abaixo.

   - **`AWS_ACCESS_KEY_ID`** : `{AWS_ACCESS_KEY_ID}`
   - **`AWS_SECRET_ACCESS_KEY`** : `{AWS_SECRET_ACCESS_KEY}`<br />

   > Substitua as variáveis `{AWS_ACCESS_KEY_ID}` e `{AWS_SECRET_ACCESS_KEY}` pelas credenciais de acesso à sua conta AWS.

4. Publique arquivos no repositório GitHub que serão sincronizados com o bucket S3.

   > Caso tenha dúvidas para publicar conteúdo em repositório GitHub, consulte a doc oficial em [https://docs.github.com/pt/github/managing-files-in-a-repository/adding-a-file-to-a-repository-using-the-command-line](https://docs.github.com/pt/github/managing-files-in-a-repository/adding-a-file-to-a-repository-using-the-command-line).

5. Publique no repositório GitHub o arquivo `.github/workflows/main.yml` com o conteúdo abaixo.

   > Esse arquivo configura o Workflow de sincronismo do repositório GitHub com o bucket S3.

   > Substitua as variáveis `{REGIÃO_AWS}` e `{NOME_DO_BUCKET}` pelos valores capturados nos passos anteriores.

   ```
   name: Sync GitHub to S3

   on:
     push:
       branches:
       - master

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:

       - name: Checkout repo
         uses: actions/checkout@v1

       - name: Set Credentials
         uses: aws-actions/configure-aws-credentials@v1
         with:
           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
           aws-region: {REGIÃO_AWS}

       - name: Deploy objects to S3 bucket
         run: |
           aws s3 sync ./ s3://{NOME_DO_BUCKET} \
           --exclude '.git/*' \
           --exclude '.github/*' \
           --exclude 'README.md' \
           --acl public-read \
           --follow-symlinks \
           --delete
   ```

## Passo 3

O terceiro passo é alterar um arquivo no repositório do GitHub e observar se o pipeline do GitHub realizou com sucesso o sincronismo com o bucket S3.

1. No repositório GitHub altere o conteúdo de um arquivo.

2. No repositório GitHub acesse a opção **Actions** e verifique o status da execução do Workflow.

   > Nesse ponto é esperado que o Workflow execute com sucesso e sincronize os arquivos do repositório GitHub com o Bucket S3.

3. Faça login no AWS Console.

4. Em **Serviços** selecione **S3**.

5. Verifique a data e hora dos objetos no bucket.
