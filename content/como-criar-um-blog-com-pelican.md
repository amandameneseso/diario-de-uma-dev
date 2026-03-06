Title: ✧ Como criar um blog com Pelican
Date: 2025-11-04
Category: Tutoriais
Tags: pelican, front-end
Author: Amanda Meneses
Slug: como-criar-um-blog-com-pelican

Olá, pessoal!

Como estudante de **Análise e Desenvolvimento de Sistemas**, estou sempre procurando ferramentas que me ajudem a documentar meus estudos e a criar projetos de forma eficiente. Recentemente, descobri o **Pelican.**

Ele é um Gerador de Sites Estáticos (SSG) construído em Python. Seu paradigma operacional difere fundamentalmente de CMSs dinâmicos como o WordPress, que processam lógica no servidor e acessam bancos de dados a cada requisição HTTP. O Pelican, por sua vez, pré-renderiza todo o site em arquivos estáticos (HTML, CSS, JS) durante um processo de *build* (compilação).

Isso resulta em uma arquitetura de alta performance e baixa latência, pois o servidor apenas entrega arquivos estáticos. A segurança também é significativamente reforçada, já que a superfície de ataque do lado do servidor é eliminada. Essa abordagem resulta em **melhor desempenho**, **maior segurança** e **facilidade de implantação** em qualquer ambiente de hospedagem estática.

Neste post, vou mostrar o passo a passo de como criei este blog. Vamos lá!

### **1. Crie uma pasta para o projeto**

Abra o terminal e digite:

```bash
mkdir meu-blog
cd meu-blog
```

Ou apenas crie uma pasta para o projeto manualmente.

### **2. O que você precisa ter instalado**

A primeira coisa que você precisa ter é o **Python** instalado no seu computador. Com ele instalado, podemos usar o `pip` (o gerenciador de pacotes do Python) para instalar o Pelican. Abra a pasta do projeto no VS Code e abra o terminal (`Ctrl +`  ou `Ctrl + Shift + '`).

Você pode verificar com:

```bash
python --version
pip --version
```

Se aparecer algo como `Python 3.x.x`, está tudo certo. Se não estiver instalado, [baixe aqui](https://www.python.org/downloads/) e depois volte para o terminal.

### **3. Crie um ambiente virtual**

Isso isola o projeto e evita conflitos de bibliotecas. Ainda no terminal, dentro da sua pasta do projeto:

```bash
python -m venv venv
```

Isso cria uma pasta chamada **venv** que vai guardar as bibliotecas do projeto. Agora, ative o ambiente:

```bash
venv\Scripts\activate
```

O nome `(venv)` aparecerá no início da linha do terminal.

### **4. Instalar o Pelican**

Se tudo estiver certo, instale o Pelican e o Markdown. Com o ambiente ativo, digite:

```bash
pip install pelican markdown
```

Isso instala o **Pelican** e suporte a **Markdown** (linguagem pra escrever as páginas). Você pode confirmar a instalação com:

```bash
pelican --version
```

### **5. Criar a estrutura inicial do site**

Agora, dentro da pasta do projeto, rode:

```bash
pelican-quickstart
```

O Pelican vai te fazer várias perguntas, como:

1. **Where do you want to create your new web site? [.]**
    
    → Caminho onde o site será criado (por padrão, o diretório atual). Apenas pressione Enter.
    
2. **What will be the title of this web site?**
    
    → Título do site (aparece no topo e nos metadados).
    
3. **Who will be the author of this web site?**
    
    → Nome do autor (usado nos artigos e no rodapé).
    
4. **What will be the default language of this web site? [en]**
    
    → Define o idioma padrão do conteúdo (ex: `pt` para português).
    
5. **Do you want to specify a URL prefix? e.g., https://example.com (Y/n)**
    
    → Permite configurar a URL base do site, caso já tenha domínio.
    
6. **Do you want to enable article pagination? (Y/n)**
    
    → Ativa a paginação automática (dividir artigos em várias páginas).
    
7. **How many articles per page do you want? [10]**
    
    → Define quantos artigos aparecem por página na listagem.
    
8. **What is your time zone? [Europe/Paris]**
    
    → Define o fuso horário usado para as datas dos artigos. (Para o Brasil: `America/Sao_Paulo`)
    
9. **Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n)**
    
    → Cria arquivos para automatizar tarefas como gerar, limpar e publicar o site. A opção recomendada é responder sim.
    
10. **Do you want to upload your website using FTP? (y/N)**
11. **Do you want to upload your website using SSH? (y/N)**
12. **Do you want to upload your website using Dropbox? (y/N)**
13. **Do you want to upload your website using S3? (y/N)**
14. **Do you want to upload your website using Rackspace Cloud Files? (y/N)**
15. **Do you want to upload your website using GitHub Pages? (y/N)**
    
    → Se for hospedar no GitHub Pages, responda “y” apenas para essa pergunta.
    
16. **Is this your personal page (username.github.io)? (y/N)**
    
    → Pergunta se o repositório será o da página pessoal ou de projeto. Responda “n”, assim o Pelican configurará o deploy para um **repositório de projeto**.
    

Depois que você responde tudo, o Pelican cria automaticamente uma pasta com estrutura padrão:

```bash
meu-site/
│
├── content/
│   └── (seus arquivos Markdown)
│
├── output/
│   └── (será criado automaticamente com o site gerado)
│
├── [pelicanconf.py](http://pelicanconf.py/)
├── [publishconf.py](http://publishconf.py/)
└── Makefile
```

### **6. Testar o site localmente**

No terminal, dentro do diretório do projeto, você pode gerar o site com:

```bash
pelican content
```

Isso cria ou atualiza a pasta `output/` com os arquivos HTML gerados. Para visualizar localmente, você pode usar:

```bash
pelican --listen
```

Agora abra no navegador no link que aparecer.

### **7. Escrevendo o conteúdo**

Dentro de `content/` você pode criar **páginas** (pages) ou **artigos/posts** (articles). Um arquivo Markdown (.md) de exemplo ficaria assim:

```markdown
Title: Meu Primeiro Artigo
Date: 2025-11-04
Category: Geral
Tags: pelican, exemplo
Author: Seu Nome

Olá! 
Esse é o meu primeiro artigo no blog feito com **Pelican**. É muito fácil escrever usando Markdown.

- Você pode criar listas.
- Adicionar **negrito** ou *itálico*.
- E até mesmo blocos de código.
```

### **8. Customizar o tema**

O tema padrão é bem simples. Você pode explorar outros temas aqui: [https://github.com/getpelican/pelican-themes](https://github.com/getpelican/pelican-themes)

Para usar um tema, clone o repositório para uma pasta dentro do projeto (por exemplo: `pelican-themes`) e adicione no arquivo `pelicanconf.py`:

```python
THEME = 'pelican-themes/nome-do-tema'
```

E é isso! Com esses passos, você tem um blog rápido e seguro, pronto para ser customizado.
