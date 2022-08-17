---
layout: post
title: Site estático com AWS S3
subtitle: Página html em segs/mins
gh-repo: kledsonhugo/kledsonhugo.github.io
gh-badge: [star, fork, follow]
tags: [AWS, S3, Web]
comments: true
---
<br/><br/>
O objetivo desta atividade é explorar na prática os conceitos de armazenamento utilizando o serviço **AWS Simple Storage Service (S3)**.

O Amazon S3 pode ser utilizado para hospedar sites estáticos.

Hospedar um site estático no Amazon S3 proporciona um site altamente escalável e de alto desempenho por uma fração do custo de um servidor Web tradicional.

Para hospedar um site estático no Amazon S3, configure um bucket do Amazon S3 para hospedagem e faça upload do conteúdo do seu site.

> Referência: [https://docs.aws.amazon.com/pt_br/AmazonS3/latest/userguide/WebsiteHosting.html](https://docs.aws.amazon.com/pt_br/AmazonS3/latest/userguide/WebsiteHosting.html)

<br/><br/>
## Passo-a-passo

1. Faça login no AWS Console.

2. Em **Serviços** selecione **S3**.

3. Selecione o botão **Criar bucket**.

4. Na tela de criação de bucket preencha com as informações abaixo e no final da tela clique em  **Criar bucket**.

   > **ATENÇÃO !!!** Substitua o texto **Bucket-Name** por um nome de Bucket qualquer. Mantenha as demais opções padrões. 

   - **nome**: `Bucket-Name`
   - **região**: Norte da Virgínia
   - **Bloquear todo o acesso público**: desabilitada
   - **Versionamento de Bucket**: habilitado<br/><br/>

5. Clique sobre o nome do Bucket criado.

6. No menu **Permissões** navegue até **Política do bucket**, clique em **Editar**, preencha com as informações abaixo e clique em **Salvar alterações**.

   > **ATENÇÃO !!!** Substitua o texto **Bucket-Name** pelo nome do bucket utilizado no passo anterior. Mantenha as demais opções padrões. 

    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::Bucket-Name/*"
            }
        ]
    }
    ```   

7. Faça download dos arquivos HTML de exemplo **index.html** e **error.html** do repositório GitHub abaixo.
 
   > GitHub repository: [https://github.com/kledsonhugo/kledsonhugo.github.io/tree/master/_data/](https://github.com/kledsonhugo/kledsonhugo.github.io/tree/master/_data/)

8. No menu **Objetos** clique em **Carregar**.

   - Selecione **Adicionar arquivos**
   - Busque pelos arquivos baixados e clique em **Carregar**
   - Clique em **Fechar**

9. No menu **Propriedades** navegue até **Hospedagem de Site estático**, clique em **Editar**, preencha com as informações abaixo e clique em **Salvar alterações**.

   - Hospedagem de site estático: `Ativar`
   - Documento de índice: `index.html`
   - Documento de erro opcional: `error.html`<br/><br/>

   > Mantenha as demais opções padrões. 

10. No menu **Propriedades** navegue até **Hospedagem de Site estático** e clique na url **Endpoint de site de bucket**.

    > **SUCESSO !!!** O sucesso dessa atividade será a abertura de uma página web mostrando o coneúdo do arquivo **index.html**.

<br/><br/>
