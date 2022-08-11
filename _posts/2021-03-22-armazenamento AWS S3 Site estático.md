---
layout: post
title: Site estático com AWS S3
subtitle: Página html em segs/mins
gh-repo: kledsonhugo/kledsonhugo.github.io
gh-badge: [star, fork, follow]
tags: [AWS, S3, Web]
comments: true
---
O objetivo desta atividade é explorar na prática os conceitos de armazenamento utilizando o serviço AWS Simple Storage Service (S3). 

O Amazon S3 pode ser utilizado para hospedar sites estáticos.

Hospedar um site estático no Amazon S3 proporciona um site altamente escalável e de alto desempenho por uma fração do custo de um servidor Web tradicional.

Para hospedar um site estático no Amazon S3, configure um bucket do Amazon S3 para hospedagem e faça upload do conteúdo do seu site.

> Referência: [https://docs.aws.amazon.com/pt_br/AmazonS3/latest/userguide/WebsiteHosting.html](https://docs.aws.amazon.com/pt_br/AmazonS3/latest/userguide/WebsiteHosting.html)

## Passo-a-passo

1. Faça login no AWS Console.

2. Em **Serviços** selecione **S3**.

3. Selecione o botão **Criar bucket**.

4. Na tela de criação de bucket preencha com as informações abaixo e no final da tela clique em  **Criar bucket**.

   - nome: `fiap-cloud-vds-aws-s3-<seu_nome>`
   - região: Selecione uma região de preferência
   - Bloquear todo o acesso público: desabilitada
   - Reconheço que as configurações atuais podem fazer com que este bucket e os objetos dentro dele se tornem públicos: habilitada
   - versionamento: Habilite a opção **Versionamento de Bucket**


   > Mantenha as demais opções padrões. 

5. Clique sobre o nome do Bucket.

6. No menu **Permissões** navegue até **Política do bucket**, clique em **Editar**, preencha com as informações abaixo e clique em **Salvar alterações**.

    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": [
                    "s3:GetObject"
                ],
                "Resource": [
                    "arn:aws:s3:::Bucket-Name/*"
                ]
            }
        ]
    }
    ```
    
    > **ATENÇÃO !!!** Substitua na Política do Bucket o texto **Bucket-Name** por `fiap-cloud-vds-aws-s3-<seu_nome>`. 
    
    > Mantenha as demais opções padrões. 

7. No menu **Propriedades** navegue até **Hospedagem de Site estático**, clique em **Editar**, preencha com as informações abaixo, guarde a url **Endpoint de site de bucket** e clique em **Salvar alterações**.

   - Hospedagem de site estático: `Ativar`
   - Documento de índice: `index.html`
   - Documento de erro opcional: `error.html`

   > Mantenha as demais opções padrões. 

8. Faça download dos arquivos **index.html** e **error.html** disponíveis no repo GitHub abaixo.
 
   - GitHub repository: [https://github.com/FIAP/vds/edit/master/aws/s3/static-site/](https://github.com/FIAP/vds/edit/master/aws/s3/static-site/)

9. No menu **Objetos** clique em **Carregar**.

   - Selecione **Adicionar arquivos**
   - Busque pelos arquivos baixados e clique em **Carregar**
   - Clique em **Fechar**

10. Abra nova guia do navegador e acesse o endpoint capturado.

    > Você deverá receber mensagem **Access Denied**.
    > Demonstra que o Bucket ou Objetos estão sem acesso público. 

11. No menu **Objetos** selcione os arquivos html e clique em **Ações** e **Tornar público**.

12. Clique em **Tornar Público**.

    > Você deverá receber mensagem **Falha ao editar o acesso público pois o Bucket está com acesso restrito**. 
    > Demonstra que objetos não podem ser públicos se um Bucket for privado. 

13. CLique em **Fechar**.

14. No menu **Permissões** clique em **Editar**.

15. Desmarque a opção **Bloquear todo o acesso público** e clique em **Salvar Alterações**.

16. Digite a palavra de confirmação e clique em **Confirmar**.

17. Atualize o conteúdo do navegador com o endpoint.

18. No menu **Objetos** repita o passo de seleção dos arquivos html e clique em **Ações** e **Tornar público**.

19. Atualize seu navegador com o endpoint guardado.

    > Você ainda deverá receber uma mensagem **Access Denied**. 
    > Agora que você tornou o Bucket público, você ainda precisa tornar os objetos públicos. 

20. No menu **Objetos** selcione novamente os arquivos html e clique em **Ações** e **Tornar público**.

21. Clique em **Tornar Público**.

22. Abra uma nova guia do seu navegador e acesso o endpoint.
