# Django Login, Logout, SignUp, Password Chance and Password Reset

## Como configurar um completo 'user authentication system' com django

## Setup Inicial

### Primeiro criar uma pasta para o seu código

```
# Windows
$ cd desktop
$ mkdir django_auth
$ cd django_auth

# macOS
$ cd ~/desktop/
$ mkdir django_auth
$ cd django_auth
```

### Ativar o ambiente virtual, no meu caso estou usando o Poetry

```
poetry init
poetry shell
poetry add django
```

#### No caso de não estar utilizando o Poetry, vamos ativar o ambiente virtual chamado venv

```
# Windows
python -m venv venv
venv\Script\Activate.ps1
pip install django

# MacOs
python3 -m venv venv
source venv/bin/activate
pip install django
```

### Como o Django já instalado vamos criar um projeto chamado de 'projeto', rodar as migrações e subir o server

```
django-admin startproject project .
python manage.py migrate
python manage.py runserver
```

### O módulo contrib do Django fornece aplicativos integrados para ajudar no desenvolvimento. No arquivo project/settings.py, sob INSTALLED_APPS, você pode ver que auth está listado e disponível para nós.

```
# project/settings.py

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",  <---
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

### Para usar o aplicativo auth, precisamos adicioná-lo ao arquivo project/urls.py em nível de projeto. No topo, importe include e crie um novo caminho de URL em accounts/. Você pode escolher um caminho de URL diferente, mas usar accounts/ é uma prática padrão e requer menos personalização posteriormente.

```
# project/urls.py
from django.contrib import admin
from django.urls import path, include  # new

urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include("django.contrib.auth.urls")),  # new
]
```

### O aplicativo auth que incluímos agora nos fornece várias visualizações e URLs de autenticação para lidar com login, logout, mudança de senha, redefinição de senha, etc. Notavelmente, ele não inclui uma visualização e URL para cadastro, então temos que configurar isso nós mesmos.

- accounts/login/ [name='login']
- accounts/logout/ [name='logout']
- accounts/password_change/ [name='password_change']
- accounts/password_change/done/ [name='password_change_done']
- accounts/password_reset/ [name='password_reset']
- accounts/password_reset/done/ [name='password_reset_done']
- accounts/reset/<uidb64>/<token>/ [name='password_reset_confirm']
- accounts/reset/done/ [name='password_reset_complete']

## Log In Page

### Vamos criar nossa página de login! Por padrão, o Django procurará dentro de uma pasta de modelos chamada registration por modelos de autenticação. O modelo de login é chamado login.html.

### Crie um novo diretório em nível de projeto chamado templates e um diretório chamado registration dentro dele.

```
mkdir templates
mkdir templates/registration
```

### Em seguida, crie um arquivo templates/registration/login.html com seu editor de texto e inclua o seguinte código:

```
<!-- templates/registration/login.html -->
<h2>Log In</h2>
<form method="post">
  {% csrf_token %}
  {{ form }}
  <button type="submit">Log In</button>
</form>

```

### Este código é um formulário padrão do Django usando POST para enviar dados e tags {% csrf_token %} por preocupações de segurança, principalmente para evitar um Ataque CSRF. O conteúdo do formulário é exibido com {{ form }}, e então adicionamos um botão "submit".

### Em seguida, atualize o arquivo settings.py para informar ao Django para procurar uma pasta de modelos no nível do projeto. Atualize a configuração DIRS dentro de TEMPLATES com a seguinte alteração de uma linha.

```
# project/settings.py
TEMPLATES = [
    {
        ...
        "DIRS": [BASE_DIR / "templates"],  # new
        ...
    },
]
```

### Nossa funcionalidade de login agora funciona, mas devemos especificar para onde redirecionar o usuário após um login bem-sucedido usando a configuração LOGIN_REDIRECT_URL. No final do arquivo settings.py, adicione o seguinte para redirecionar o usuário para a página inicial.

```
# project/settings.py
LOGIN_REDIRECT_URL = '/'
```

### Agora navegue até http://127.0.0.1:8000/accounts/login/

### Só podemos fazer login se tivermos uma conta de usuário. E como adicionar um formulário de registro ainda está por vir, a abordagem mais direta é criar uma conta de superusuário a partir da linha de comando. Pare o servidor com Control+c e depois execute o comando python manage.py createsuperuser. Responda às solicitações e observe que sua senha não aparecerá na tela ao digitar por razões de segurança.

### Agora inicie o servidor novamente com python manage.py runserver e atualize a página em http://127.0.0.1:8000/accounts/login/. Insira as informações de login para o superusuário que você acabou de criar.

### Nosso login funcionou porque nos redirecionou para a página inicial, mas ainda precisamos criar essa página inicial, então vemos o erro "Página não encontrada". Vamos corrigir isso!

## Crie uma Página Inicial

### Queremos uma página inicial simples exibindo uma mensagem para usuários logados e outra para usuários desconectados. Crie dois novos arquivos com seu editor de texto: templates/base.html e templates/home.html. Observe que esses arquivos existem dentro da pasta templates, mas não dentro de templates/registration/, onde o Django auth procura por padrão por modelos de autenticação de usuário.

```
<!-- templates/base.html -->
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <title>{% block title %}Django Auth Tutorial{% endblock %}</title>
</head>

<body>
  <main>
    {% block content %}
    {% endblock %}
  </main>
</body>

</html>
```

```
<!-- templates/home.html -->
{% extends "base.html" %}

{% block title %}Home{% endblock %}

{% block content %}
{% if user.is_authenticated %}
Hi {{ user.username }}!
{% else %}
<p>You are not logged in</p>
<a href="{% url 'login' %}">Log In</a>
{% endif %}
{% endblock %}
```

### Enquanto estamos nisso, podemos atualizar também o arquivo login.html para estender nosso novo arquivo base.html:

```
<!-- templates/registration/login.html -->
{% extends "base.html" %}

{% block title %}Login{% endblock %}

{% block content %}
<h2>Log In</h2>
<form method="post">
  {% csrf_token %}
  {{ form }}
  <button type="submit">Log In</button>
</form>
{% endblock %}
```

### Agora, atualize o arquivo project/urls.py para que possamos exibir a página inicial. Importe TemplateView na terceira linha e adicione um padrão de URL para ele no caminho, " ".

```
# project/urls.py
from django.contrib import admin
from django.urls import path, include
from django.views.generic.base import TemplateView  # new

urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include("django.contrib.auth.urls")),
    path("", TemplateView.as_view(template_name="home.html"), name="home"),  # new
]
```

### E terminamos. Se você iniciar o servidor Django novamente com python manage.py runserver e navegar até a página inicial em http://127.0.0.1:8000/, você verá o Hi, e o nome do seu username

### Mas como fazemos logout? Atualmente, a única opção é acessar o painel de administração em http://127.0.0.1:8000/admin/ e clicar no link "Log Out" no canto superior direito.

### O link "Logout" nos desconectará.

### Se você voltar para a página inicial novamente em http://127.0.0.1:8000/ e atualizar a página, é visível que estamos desconectados.

## Botão de Logout

### Uma das mudanças no Django 5.0, conforme observado nas notas de lançamento, é a remoção do suporte para logout via solicitações GET. Nas versões anteriores do Django, você poderia adicionar um link de logout como <a href="{% url 'logout' %}">Logout</a> em um arquivo de modelo. Mas agora é necessário uma solicitação POST através de um formulário.

### Vamos demonstrar isso agora, adicionando um botão "Logout" à página inicial. Podemos adicionar um botão de logout ao arquivo home.html sob a seção {{ user.username }}.

```
<form action="{% url 'logout' %}" method="post">
  {% csrf_token %}
  <button type="submit">Log Out</button>
</form>
```

### Então, atualize settings.py com nosso link de redirecionamento, LOGOUT_REDIRECT_URL. Adicione-o logo ao lado do nosso redirecionamento de login para que o final do arquivo settings.py fique da seguinte forma:

```
# project/settings.py
LOGIN_REDIRECT_URL = "/"  # redirecionamento após o login
LOGOUT_REDIRECT_URL = "/"  # redirecionamento após o logout

```

### Se você fizer login e revisitar a página inicial, será redirecionado para a nova página inicial com um link "logout" para usuários conectados.

### Clicando nele, você será levado para a página inicial desconectada com um link "Log In".

## Página de Cadastro

### Agora que resolvemos os problemas de login e logout, é hora de adicionar uma página de cadastro ao nosso site básico do Django. Se você se lembra, o Django não fornece uma visualização ou URL integrada para isso, então precisamos codificar o formulário e a página nós mesmos.

### Para começar, pare o servidor web local com Control+c e crie um aplicativo dedicado chamado accounts, que usaremos para nossa lógica de conta personalizada.

#### Eu gosto de criar uma pasta apps, e criar meus apps django dentro desta pasta, você pode fazer diferente se preferir

```
mkdir apps
cd apps
django-admin startapp accounts
```

### Dentro de apps/accounts no arquivo apps.py:
```
name = 'apps.accounts'
```
### Certifique-se de adicionar o novo aplicativo à configuração INSTALLED_APPS no arquivo project/settings.py:
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    'apps.accounts', <--
]
```
### Em seguida, adicione um caminho de URL em project/urls.py que esteja acima do nosso aplicativo de autenticação Django incluso. A ordem é importante aqui porque o Django procura padrões de URL de cima para baixo. Queremos manter o padrão de ter nossa lógica de autenticação de usuário em accounts/, mas garantir que a página de cadastro seja carregada primeiro.
```
from django.contrib import admin
from django.urls import path, include
from django.views.generic.base import TemplateView

urlpatterns = [
    path("admin/", admin.site.urls),
    path("accounts/", include("apps.accounts.urls")),  # novo
    path("accounts/", include("django.contrib.auth.urls")),
    path("", TemplateView.as_view(template_name="home.html")),
]
```
### Em seguida, crie um novo arquivo chamado accounts/urls.py com o seu editor de texto e adicione o seguinte código.

```
from django.urls import path

from .views import SignUpView

urlpatterns = [
    path("signup/", SignUpView.as_view(), name="signup"),
]
```
### Agora, para o arquivo accounts/views.py:

```
from django.contrib.auth.forms import UserCreationForm
from django.urls import reverse_lazy
from django.views.generic import CreateView

class SignUpView(CreateView):
    form_class = UserCreationForm
    success_url = reverse_lazy("login")
    template_name = "registration/signup.html"
```

### Ok, agora para a etapa final. Crie um novo modelo, templates/registration/signup.html, e preencha-o com este código que se parece quase exatamente com o que usamos para login.html.

```
{% extends "base.html" %}

{% block title %}Cadastre-se{% endblock %}

{% block content %}
<h2>Cadastre-se</h2>
<form method="post">
  {% csrf_token %}
  {{ form }}
  <button type="submit">Cadastrar</button>
</form>
{% endblock %}
```

## Alteração de Senha

### O Django fornece uma implementação padrão da funcionalidade de alteração de senha. Para experimentá-la, faça logout de sua conta de superusuário e faça login com seu usuário regular.

### A página padrão de "Alteração de Senha" está localizada em http://127.0.0.1:8000/accounts/password_change/.

### Digite sua senha antiga e, em seguida, uma nova senha duas vezes. Clique no botão "Alterar Minha Senha" e você será redirecionado para a página "Alteração de Senha Bem-Sucedida".

### Se você deseja personalizar essas duas páginas de alteração de senha para combinar com a aparência do seu site, é necessário apenas substituir os modelos existentes. O Django já nos fornece as visualizações e URLs necessárias. Para fazer isso, crie dois novos arquivos de modelo no diretório de registro:

#### templates/registration/password_change_form.html
#### templates/registration/password_change_done.html

### Vamos adicionar um link para a página de alteração de senha em nosso modelo home.html. Também adicionei tags "p" para algum espaçamento.
```
{% extends "base.html" %}

{% block title %}Início{% endblock %}


{% block content %}
{% if user.is_authenticated %}
<p>Olá, {{ user.username }}!</p>
<p><a href="{% url 'password_change' %}">Alterar Senha</a></p>
<form action="{% url 'logout' %}" method="post">
  {% csrf_token %}
  <button type="submit">Sair</button>
</form>
{% else %}
<p>Você não está logado</p>
<a href="{% url 'login' %}">Entrar</a>
{% endif %}

{% endblock %}
```
### Este link redirecionará os usuários autenticados para a página de alteração de senha quando clicado.

## Redefinição de Senha

### Uma página de redefinição de senha é útil quando um usuário esquece suas informações de login: um usuário pode inserir seu endereço de e-mail e receber um e-mail criptograficamente seguro com um link único para uma página de redefinição de senha. Isso geralmente está disponível para usuários desconectados. O Django possui funcionalidade integrada para isso que requer apenas uma pequena quantidade de configuração.

### Vamos adicionar um link para a página padrão de redefinição de senha que estará disponível para usuários desconectados.

```
{% extends "base.html" %}

{% block title %}Início{% endblock %}

{% block content %}
{% if user.is_authenticated %}
<p>Olá, {{ user.username }}!</p>
<p><a href="{% url 'password_change' %}">Alterar Senha</a></p>
<form action="{% url 'logout' %}" method="post">
  {% csrf_token %}
  <button type="submit">Sair</button>
</form>
{% else %}
<p>Você não está logado</p>
<p><a href="{% url 'password_reset' %}">Redefinição de Senha</a></p>
<p><a href="{% url 'login' %}">Entrar</a></p>
{% endif %}
{% endblock %}
```
### O modelo padrão é simples e estilizado para corresponder ao administrador, mas é funcional. Queremos experimentá-lo, mas há um problema: nossa conta de usuário regular não possui um endereço de e-mail associado a ela. O formulário de criação de usuário padrão do Django que estendemos para nosso formulário de cadastro não inclui um campo de e-mail! É possível atualizá-lo para incluir um campo de e-mail, mas fazer isso está além do escopo deste tutorial.

### No entanto, há uma solução fácil. Faça login no administrador, clique em Usuários e selecione o nome de usuário para sua conta de usuário regular para abrir a página de edição de usuário onde você pode adicionar um e-mail.

### Certifique-se de clicar no botão "Salvar" na parte inferior da página. Em seguida, clique no botão "Sair" no canto superior direito do painel de administração ou volte à página inicial.

### O Django utiliza por padrão um backend de e-mail SMTP que requer alguma configuração. Para testar o fluxo de redefinição de senha localmente, podemos atualizar o arquivo project/settings.py para enviar e-mails para o console em vez disso. Adicione esta linha ao final do arquivo.

### Por fim, podemos tentar a página de Redefinição de Senha novamente em http://127.0.0.1:8000/accounts/password_reset/. Insira o endereço de e-mail para sua conta de usuário regular e clique no botão "Alterar Minha Senha". Isso irá redirecioná-lo para a página de envio de redefinição de senha.

### Por razões de segurança, o Django não fornecerá nenhuma notificação sobre se você inseriu um e-mail que existe no banco de dados ou não. Mas se você olhar no seu terminal/console agora, poderá ver o conteúdo do e-mail exibido lá.

### Digite uma nova senha e clique no botão "Alterar minha senha". Você será redirecionado para a página de Redefinição de Senha Completa.

### Redefinição de Senha Completa. Para confirmar que tudo funcionou corretamente, navegue até a página inicial e faça login em sua conta com a nova senha.

### Se você deseja personalizar os modelos envolvidos na redefinição de senha, eles estão localizados nos seguintes locais; você precisa criar novos arquivos de modelo para substituí-los.

  - templates/registration/password_reset_confirm.html
  - templates/registration/password_reset_form.html
  - templates/registration/password_reset_done.html


## Conclusão

### Implementamos um fluxo de autenticação de usuário robusto para nosso aplicativo da web com login, logout, cadastro, alteração de senha e redefinição de senha. O Django requer mais configuração do que outros frameworks da web, mas permite personalização completa. Esse é o trade-off envolvido.