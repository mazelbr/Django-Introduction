# Django
Django is a MVC-Frameworkand we split our code into `models.py`, `views.py` and `urls.py`
* Models contains the decription of the database represented by a python class. Die Klasse wird `Model` genannt und besitzt methoden für das erstellen, updaten oder löschen von daten (CRUD).
* Views Contains the Business Logic. The functions of it are called views and are producting the content.
* urls.py: directs request to the views dor a given pattern www.example.com/latest calls the latest dunction of view.py
* <...>.htmö is a htmö tepmplate with basic lofic to display data

## Setting up a DataBase
django books reccomends FireBirdSQL
1. install and configure a database server (e.g. ) often set up by the hosting provider
2. install the python library for the specific backend
FireBirdSQL -> packages: kinterbasdb or fdb for Postgresql use psycop2

## Starting a Project
1. create a new directory go into it and run `django-admin startproject <name>` 
This creates the follwing file structure:
```
mysite/
    manage.py           <- commandline stuff 
                        check out python manage.py help
    mysite/                 <-- thats the name of the project
        init.py             <-- necesarry to be a package
        settings.py         <-- configurations
        urls.py             <-- routing file
        wsgi.py             <- entry point for WSGI compatible webserver
```

2. Running the Dev-Server: **only use for developement this is not safe for production**
`python manage.py runserver`. if you want to change the port of the server type `manage.py runserver 0.0.0.0.8080`

## Views and URLconfs
example hello world:
1. create an empty file e.g. `views.py` (names do not matter but views. py is the convention)
2. in the views 
```
from django.http import HttpResponse
def hello(request):
    return HttpResponse("Helll World")
```
> Note that each view function takes at leas one argument called 'request' (convention) here we take the request and responde with HttpResponse

An URLconfs is a table that connects requests with views and is defined in `urls.py`.
Routing in django is defined by regex patterns `urlpatterns = patterns('',('^hello/$', hello), ) # djagno 3 uses repath for regex paths` 
We just told django to take the /hello request and rout it to the hello method in the views.
> Note that by default python adds a / to routes that do not end with / so 'hello' would still find its way to 'hello/'

The home view : you can add a root view the same way we just use ('^$', home) as a route
(anchor ^means start of string $ means end of string)
## How django processes Requests
First it looks up the settings.py file where ROOT_URL_CONF is defined. The routing file is linked by defaulte to the urls.py file.
Django switches to the url.py file and searches for a matching request in urlpatterns. It switches to the corresponding view and returns the HTTPREsponse

## Loose Coupling
is one of the core philosphy behind Django. It states that things shouled stay exchangable and that two pieces that do not depend on each other should  be handeld such that changes in one pice has little to no effect on the other

## Dynamic URL's
Eventhough it is possible to transfer variables with the URL Django provides pretty URLs. Instead of defining variables within the URL (e.g. /time/plus?hour=3) you can mark the values you want to extract with paranthesis  in the rounting path
(r"^time/plus/(\d+)/$", hours_ahead), and the marked variable is given as an input to views.hours_ahead() method.
> Note that you already can and should restric the possibinputs through regex (\d -> integer) (\d{1,2} -> integer with one or two digets)

The hours_aheas(query, offset) takes tha value as the second argument here offset. The first one is always query.
According to loose coupling you should check here the input restrictions again so if we change the routes the view method still works the same way. If you either construct the routoing structure or start with the views is both equally fine
## Django 3 URL dispatcher
Django changed the default rounting in django 3. The default routing is no more pattern based and naming varables now is easier
`path('articles/<int:year>/', views.article_year)`
`repath(r'artickes/(?p<year>[0-9]{4}/$), views.article_year)`
## Usefull stuff
Django has its own python command line interpreter where you can access all the config files and the stuff that one already coded. This is very usefull to test qick lines. It can be accesed with `python manage.py shell`


## Templates
To seperate logic (python) and content (html) and because it is just unintuitive and redundant to generate all the html with print statements, Django offers templates

We basically can write a html file as usual and when it is needed we ca dynamically pull from python variables.
```python
<html>
<body>
    <p> My name is {{ name }} </p>
    <ul> {% for item in item_list}
        <li> {{ item }} </li>
        {% endfor %}
    </ul>
```
We also could use `{%if %} {% else %} {% endif %}`
In short we use double bracelets for variables and `{% %}` for logic.
A nice feature is the filter option. Similar to the | pipe command in unix, one can pipe the variable into a filter an specify its formating `{{ ship_date | date: "F j, M"  }}`.
At first comes the name and after the pipe we specify the type of the variable and then the formating

### The Template System
> Note that even though one will use the template system mostly  together with view it is in fact a standalone module of django
 
 How it works:
 1. import the module `from django import template`
 2. create a template object providing a string ` t= template.Template('My name is {{ name }}')`
 3. use `render()` to fill it with content
 ```python
 c = template.Context({'name':'Adrian'})
 print( t.render(c))
 ```
Note that we are generating template **objects** and not strings.

Once you created a tempalte object you can start filling it with data and giving it a context. A context is similar to dictionaries but with addiotional functionality.

Best practice here is to create a string variable with all the extended html and all the python tags that are needed for that template first, then create a template obkect that takes that string and finally render it.

### Multiple Context with same Template
Django template rendering is quite fast beause it does not rely on XML, but you should avoid generating the same template over and over and only loop through the rendering process.
```python
# Bad
for name in ('John', 'Julie', 'Pat'):
    t = Template('Hello, {{ name }}')
    print t.render(Context({'name': name}))

# Good
t = Template('Hello, {{ name }}')
for name in ('John', 'Julie', 'Pat'):
    print t.render(Context({'name': name}))

```
### Lookups
Django provides adddiotional functionality to acces class attributes, lists, dictionaries and even simple functions within the templates.
All of them are accesed by the dot `.` e.g. `person.name`. In the same way we can refer to keys of dictionaries `dict.key` or list indexes `list.0`. The lists are a little bit different from the normal indexing since we are using `.i` instead of `[i]`.
As mentioned we also can use simple methods e.g. simple string mathods as `upper()` or `isdigit()`. Note that we only can use methods that take no arguments. Since we call them attribute-like without paranthesis e.g. `"hello".upper`
This is intended as templates should not contain any business logic but only present the content.
> Methods that we look up get called. This means if they do have side effects those gonna happen. This can be foolish at it's best but also quite danagerous due to security leaks. Consider a `delete` method in a template. Everytime we render the template the corresponing data would be deleted. You can prevent this from happening by defining a `alter_data` attribute to the method.

## Tags and filter
### Tags
Views have quite a big toolbox of tags like `{% if %} { % else %}` . Since it is a common task to check if we have data entries to display django templates have their own *Truthiness*. That meand that empty lists, dictionaries, tulple, strings, the number 0, False and None are all evaluated as False. For logical concatenation we have `and`, `or` and `not`.
e.g. `{% if [] %}` would not trigger the if statement.

The `for` can be used to iterate over sequences. Note that we have to end the for loop explicitly as we have no indentations (use `{% endfor %}`)

But there are additional tags like `reverse`, `{% empty %}` which allows you to define the behaviour if a sequence is empty.and furher `forloop.counter`, `forloop.last`, `forloop.parentloop`.  
```python 
{% for athlete in athlete_list %}
    <p>{{ athlete.name }}</p>
{% empty %}
    <p>There are no athletes. Only computer programmers.</p>
{% endfor %}
```

Comparisons have also a own tag `{% ifequal %} {% endifequal %}`
> Note that you can only access lists and parameters you explicitly defined in the Context Method

### Filters
can be chained `list|first|upper`
other are `truncatewords`, `length

### Philosphies
* Business logic shoudl be seperated of presentational logic
* syntax should be seperated from html

## How To use Templates in Views
Obviously you do not want to define the same template over and over for each method possibly want to inheriate vies later.
Therefore we store the templates in a `template` folder in the same directory as `settings.py`
To make sure that django finds the templates there we have to specify the directory in the`TEMPLATE[... DIRS[ ] ...].
You can use `os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/')` to store the template dir dynamically in the same directory as `settings.py` or alternatively use explicit file paths. 
The `'APP_DIRS': True` allows django to search in the App directory for templates as well.

Now Django can find all templates you define in the `templates` folder but you have to change the way how those gets loaded. Forst of all you import the `get_template()` method from `django.template.loder` an specify the filename in it.
```
from django.template.loader import get_template
from django.template import Template
def hello(request):
    t = get_template('tempname.html')
```
YOu could then render the template as usual but there is a better way, namely `render_to_response`
```
from django.shortcuts import rener_to_response
def hello(request):
    return render_to_response('tempname.html', {'var1': val1})
```
### The locals() trick
One propably can imaging that you will create a lot of variables within each view that we then pass to the respond_to_render method. Especially in more complex applications the numer of local variables can grow quite quckly which we would than have to pass manually. But there is a more efficient way to do that since `locals()` just includes ALL locally definded variables. This can be a cute shortcut but keep in mind that in eventually would also pass sensitive information to the template.

## Subdirectories
Of course you can further specify the file path in get_template() e.g.g `get_template('dir/hello.html')` This allows you to seperate views for different models and you can a consistent scheeme for CRUD actions similar to Rails

## Inheritance of Tempaltes
In real world applications the overall structure of the site hardly changes but rather some section are updated and filled with data. Therefore it comes in handy if you can e.g. define the Nav-bar elsewhere and load at the top of each side. So if you change the navbar you can do it in one place and not in all subpages. In the same way you can try do reuce redundancy for all pages.
One could create an `includes` directory to store all the global pages. Django the provides an include tag `{% include "includes/nav.html" %}` to load e.g. the navbar.

An even better approach is the so called template inheritance, where you define one global scheme in e.g. `base.html` and then specify differences in child themes that inheriate from the base theme through blocks.
lets consider the following base.html
```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html lang="en">
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <h1>My helpful timestamp site</h1>
    {% block content %}{% endblock %}
    {% block footer %}
    <hr>
    <p>Thanks for visiting my site.</p>
    {% endblock %}
</body>
</html>
```
This base theme specifies all the things that all child themes have in common. For all the differences its the child theme's job to implement them.
This means:
```
{% extends "base.html" %}

{% block title %}The current time{% endblock %}

{% block content %}
<p>It is now {{ current_date }}.</p>
{% endblock %}
```
That we can specify all the block ins the child theme
> Note that the block names within one template should be unique as they are identifiers in both directions.

>Note further that you do not have to specify all blocks of the parent theme. If defined django used then the same block of the parent as a fallback or if not does it fails silently

Some more guidelines are:
* {% extends %} has to be the first tag
* The more tags defined in the parent theme the better
* if you need a tag from the parent you can use {{block.super}}

# MVC vs MTV
Similarly to the model view controller django has a Model, Template, View structure where:
* Model: The model handels everything about the data, the access, the validation, the retrival and the relationships
* The Templates are the presentation layer. Here all presentation related decsions are contained and how it is represented
* The Views handle all the business logic. It connects the database with the templates

Views in django are similar to the controler layer in rails

## Configuring the Database
Depending on the backend you choose the process is slightly different and can be optained from the docs of ddjango and the database connector. Then we have to set the configurations inside the `settings.py` file in the database section

> Note that if you are using SQLight you do not further configure the database. Django by default sets up a SQLight db for you

## Your first app
This might be confusing but now you have to add an app to your project. By now you only have created a project that can contain several apps. Lateron you would seperate big websites in apps for e.g. onlinesshop, payment etc. Technically the only requirement for the project is to contain a setting.py file

An App is a portable set of django functionalities with its own moels and views
Note that apps not necessary. You could also use the rounting part of django only.

1. Start a new app `python manage.py startapp appname` creates a app inside the project directory it contains of 
```
books/
    __init__.py
    admin.py
    views.py
    models.py
    apps.py
    tests.py
```

## Your first model
> Note that the way to create models changed a bit in newer versions for django and the way provoced in the book does not work anymore.

first of all we have to create the Models as classes inside the `model.py ` file
```python
class Publisher(models.Model):
    name = models.CharField(max_length=30)
    adress = models.CharField(max_length=50)
    city = models.CharField(max_length=60)

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
```
The attributes are created by predefined field types. Those refere directly to SQL statements and there are additional contraints available. You can also link databases but you have to define the `on_delete` attribute

We then have to ens
