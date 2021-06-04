# Django official tutorial learning notes
A small white backend guy is learning Django. Here are what happened when he was struggling with that...

# MarkDown basic grammars
1. listing points: * and contents after that
* demo_one
  * demo_two
2. adding codes: ` twice, with codes inside

    `demo_three`

3. url: [name] first then (url) follows

   [demo_four](https://docs.djangoproject.com/zh-hans/3.0/)

4. big grey: one newline and double indent

        demo_five
# Django tutorials
## 创建项目
* check Django package version: `python -m django --version`
* create a Django project: 
  
  `django-admin startproject <project_name>`
  
  file structure inside the created project is 

        <project_name>/
            manage.py  # manage the proj in cmd https://docs.djangoproject.com/zh-hans/2.0/ref/django-admin/  
            <project_name>/
                __init__.py # python package specification, to avoid mixing with ordinary file
                settings.py # setting app, path, db, directory, url, etc
                urls.py # url manager
                wsgi.py # extra arguments?? Don't know yet
  
* start running:

        python manage.py runserver
        # python manage.py runserver <IP_port_number_default_8000>
* if the running is successful, we can see in terminal:    

        Performing system checks...

        System check identified no issues (0 silenced).

        You have unapplied migrations; your app may not work properly until they are applied.
        Run 'python manage.py migrate' to apply them.

        八月 01, 2018 - 15:50:53
        Django version 2.0, using settings 'mysite.settings'
        # follow the url below and you can see the running server
        Starting development server at http://127.0.0.1:8000/
        Quit the server with CONTROL-C.

## 创建应用
1. Open cmd, cd to the directory of a project containing manage.py
2. Run `python manage.py startapp <app_name>`
3. Resulting in a directory with file structure:

        <app_name>/
            __init__.py # py package specification
            admin.py # manager
            apps.py # app configs
            migrations/
                __init__.py # py subpackage specification
            models.py 
            tests.py # for auto-test usage
            views.py 

## 换时区

change the global variable at
settings.py -> time_zone

## 数据库设置
1. install database bindings, see https://docs.djangoproject.com/zh-hans/2.0/topics/install/#database-installation
2. change DATABASES 'default' in settings.py:

        DATABASES = {
            'default': {
            'ENGINE': 'django.db.backends.oracle', -- DB type
            'NAME': 'xe', -- DB storage path
            'USER': 'a_user', -- DB configs
            'PASSWORD':'a_password',
            'HOST': '',
            'PORT': '',
            }
        }

## 还是用中文吧qwq
# 写并激活模型
模型：数据库结构设计（表）
1. 在<app_name>/models.py更改，样例如下

        import datetime

        from django.db import models
        from django.db.models.base import Model
        from django.utils import timezone

        # class -> 一组字段（表）
        # object -> 一条条数据
        class Question(models.Model):
            # 字段名和字段种类
            question_text = models.CharField(max_length=200)
            pub_date = models.DateTimeField("date published")
            # 指定模型打印输出
            def __str__(self):
                return self.question_text
            # 对象函数 用于查看属性
            def was_published_recently(self):
                # return self.pub_date >= timezone.now() -  datetime.timedelta(days=1)
                return self.pub_date <= timezone.now() and  self.pub_date >= timezone.now() - datetime.  timedelta(days=1)

        class Choice(models.Model):
            # 指定多对一关系 自动创建id_column自动管理?
            # on_delete指定删除主表里object时对副表的操作（默认：   引用者一起删除）
            question = models.ForeignKey(Question,  on_delete=models.CASCADE)
            choice_text = models.CharField(max_length=200)
            # 字段名和字段种类
            votes = models.IntegerField(default=0)
            # 指定模型打印输出
            def __str__(self):
                return self.choice_text
2. 在<project_name>/settings.py里：

        # all Application definition

        INSTALLED_APPS = [
            # 自己写的应用
            # 格式：应用名.apps.应用名首字母大写Config
            'polls.apps.PollsConfig', # 为项目添加应用
            # 系统默认应用
            'django.contrib.admin', # 管理员站点
            'django.contrib.auth', # 认证授权系统
            'django.contrib.contenttypes', # 内容类型框架
            'django.contrib.sessions', # 会话框架 
            'django.contrib.messages', # 消息框架 
            'django.contrib.staticfiles', # 管理静态文件的框架
        ]
3. 在cmd运行 
   
        python manage.py makemigrations polls -- 应用名 
    为模型的改变生成迁移文件。
4. 在cmd运行 
   
        python manage.py migrate 
    来应用数据库迁移。

## Django API 

        python manage.py shell    
或许可用于调试 暂时没深究

# 管理
## 创建管理员账号

        python manage.py createsuperuser
        
        Username: admin
        
        Email address: admin@example.com

        Password: **********
        
        Password (again): *********
        
        Superuser created successfully.
## 管理员登录
1. 
        python manage.py runserver


2. 访问url: 域名/admin/
3. 登录

## 添加管理应用
1. 在<app_name>/admin.py

        from django.contrib import admin

        from .models import Question

        admin.site.register(Question)

        # Register your models here.
此处注册应用中需要被管理的模型，同时应用也被添加

# 视图
## 基于函数的视图 
### general 框架（函数细节）

        

        # emm导入的包可能有多余 暂时放着
        from django.http import HttpResponse, HttpResponseRedirect
        from .models import Question, Choice
        from django.template import loader
        from django.shortcuts import render, get_object_or_404
        from django.http import Http404
        from django.urls import reverse
        from django.views import generic
        from django.utils import timezone
        from django.test import TestCase

        # request: URLconf调用的触发器
        def vote(request, question_id):
            # 鲁棒的查询
            # arg1: 被查询的表名(from clause) arg2: 查询条件(where clause) 
            question = get_object_or_404(Question, pk=question_id)
            try:
                # 用前端输入里choice的信息查找choice object
                selected_choice = question.choice_set.get(pk=request.POST['choice'])
            except (KeyError, Choice.DoesNotExist): 
                # 前端输入没有给出choice
                # render(请求, '模板文件路径', 传入参数)
                return render(request, 'polls/detail.html', {
                    'question': question,
                    'error_message': "you didn't select a choice!",
                })
            else: 
                # 返回后对此data对象实例的投票数做增加
                selected_choice.votes += 1
                selected_choice.save()
                # 软编码使用url：reverse(url_name（在urls.py中指定）, arg=(...))
                # HttpResponseRedirect重定向到指定的url名称，用后面args作为传递给url的参数
                # 视图变换（在一个视图里调另一个）要经过url
                return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))

        def results(request, question_id):
            # 思路：
            #   取数据
            #   显示
            question = get_object_or_404(Question, pk=question_id)
            return render(request, 'polls/results.html', {"question": question})

        def index(request):
            # 查数据并排列 相当于select from group by
            latest_question_list = Question.objects.order_by('-pub_date')[:5]
            # 指定上下文
            context = {'latest_question_list': latest_question_list}
            return render(request, 'polls/index.html', context)
### 按钮界面跳转原理
* 前端模板按钮被按下 -触发->
* url（通过url名称） -触发-> 
* 视图（函数1） -HttpResponseRedirection触发->
* url -触发-> 
* 另一个视图（函数2） --> 
* 返回render(request, "跳转页模板", 上下文字典)

视图函数作用分别是：
* 函数1(vote即第一次url绑定的函数)：更新数据库
* 函数2(HttpResponse过去的函数)：跳转页面


### 视图内模板调用原理
1. 在<app_name>/templates/<app_name> 下放进模板文件xx.html（加一级目录是为了增加命名空间，防止不同应用重名的模板调用错误）
2. 在app的视图里 用render(request, "<app_name>/xx.html", context) 使用模板
3. 组织context为字典{"html里的变量名": 变量值}，具体变量值格式应该取决于前端吧qwq


### 视图添加和更改
1. In <app_name>/view.py, add view function

        # from http.py in django package import HttpResponse(...) function
        from django.shortcuts import render
        from django.http import HttpResponse

        # function to be binded to a url through a URLconf
        def func_name(request,...):
            return render(request, "template_dir", context_dict)
2. add the template to <app_name>/templates/<app_name>
3. create **urls.py** under <app_name> directory (if exists, skip it)
4. in the application, add app_url mapping to the view, remember to extract proper arguments for the view function

        # from urls.py in django package import path class
        from django.urls import path

        # import from current directory (<app_name>)
        from . import views
        # path('url_postfixes', view_function, name="view_function")
        urlpatterns = [
            path('url_with_extracting_necessary_arguments', views.func_name, name='any_name_u_like'),
        ]
5. in the project, update urlpatterns in <project_directory>/urls.py if necessary

        urlpatterns = [
            # concatenate "test/" to app's specification of url
            path('test/', include('<app_name>.urls')),
            # default url for admin
            path('admin/', admin.site.urls), 
        ]

## 基于类的视图
### general 框架（函数细节）

        # 一个问题选项具体内容
        class DetailView(generic.DetailView):
            model = Question
            template_name = 'polls/detail.html'
            def get_queryset(self): # 防止用户猜到url访问
                """
                Excludes any questions that aren't published yet.
                """
                return Question.objects.filter(pub_date__lte=timezone.now())

        class IndexView(generic.ListView):
            template_name = 'polls/index.html'
            context_object_name = 'latest_question_list'
            # 改良：先查找pub_date在过去的问题
            # 再筛选前五个
            def get_queryset(self):
                """Return the last five published questions."""
                # pub_date_lte=:返回某字段值小于等于（lte）某值的所有对象
                return Question.objects.filter(pub_date__lte=timezone.now()).order_by('-pub_date')[:5]

        # 一个问题各选项得票数
        class ResultsView(generic.DetailView):
            model = Question
            template_name = 'polls/results.html'

        # request: URLconf调用的触发器
        def vote(request, question_id):
            # 鲁棒的查询
            # arg1: 被查询的表名(from clause) arg2: 查询条件(where clause) 
            question = get_object_or_404(Question, pk=question_id)
            try:
                # 用前端输入里choice的信息查找choice object
                selected_choice = question.choice_set.get(pk=request.POST['choice'])
            except (KeyError, Choice.DoesNotExist): 
                # 前端输入没有给出choice
                # render(请求, '模板文件路径', 传入参数)
                return render(request, 'polls/detail.html', {
                    'question': question,
                    'error_message': "you didn't select a choice!",
                })
            else: 
                # 返回后对此data对象实例的投票数做增加
                selected_choice.votes += 1
                selected_choice.save()
                # 软编码使用url：reverse(url_name（在urls.py中指定）, arg=(...))
                # HttpResponseRedirect重定向到指定的url名称，用后面args作为传递给url的参数
                # 视图变换（在一个视图里调另一个）要经过url
                return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
### 有待补充

# 模板
模板：页面显示格式的指定，放在<app_name>/templates下

是前端管的事吧 暂时没有深究Html, CSS的语法qwq

但应该根据上下文(context)需要的数据类型提供数据

# 自动化测试
## general 框架 （函数细节），模型测试及视图测试

        import datetime
        from django.test import TestCase
        from django.utils import timezone
        from django.urls import reverse
        from .models import Question
        
        # 此函数用于创建问题对象并插入数据库（模型）
        def create_question(question_text, days):
            time = timezone.now() + datetime.timedelta(days=days)
            return Question.objects.create(question_text=question_text, pub_date=time)
        
        # 基于类进行的测试
        # 每个测试保证应用的一小部分代码的安全性
        # 无论它以后成长到多么复杂，要和其他代码进行怎样的交互，我们都能保证进行过测试的那些方法的行为永远是符合预期的。

        # 模型内函数测试
        class QuestionModelTests(TestCase):

            def test_was_published_recently_with_future_question(self):
                """
                was_published_recently() returns False for questions whose pub_date
                is in the future.
                """
                # 未来30d后的时间
                time = timezone.now() + datetime.timedelta(days=30)
                # 建立问题
                future_question = Question(pub_date=time)
                # 希望：问题对象最近发布为假
                self.assertIs(future_question.was_published_recently(), False)

            def test_was_published_recently_with_recent_question(self):
                # 23h59m59s前的时间
                time = timezone.now() - datetime.timedelta(days=0, hours=23, minutes=59, seconds=59)
                # 建立问题
                future_question = Question(pub_date=time)
                # 希望：问题对象最近发布为假
                self.assertIs(future_question.was_published_recently(), True)

            def test_was_published_recently_with_old_question(self):
                # 1d23h59m59s前的时间
                time = timezone.now() - datetime.timedelta(days=1, hours=23, minutes=59, seconds=59)
                # 建立问题
                future_question = Question(pub_date=time)
                # 希望：问题对象最近发布为假
                self.assertIs(future_question.was_published_recently(), False)

        # 视图函数测试
        class QuestionIndexViewTests(TestCase):
            def test_no_questions(self):
                """
                If no questions exist, an appropriate message is displayed.
                """
                # 基于模板变量值对视图返回结果的检测（前提：模板上下文正常）
                # 登录按名字索引的url 返回已渲染的模板
                response = self.client.get(reverse('polls:index'))
                # 检测访问页面成功打开
                self.assertEqual(response.status_code, 200)
                # 检测页面打印有无相应字符串
                self.assertContains(response, "No polls are available.")
                # 检测query结果正确性
                self.assertQuerysetEqual(response.context['latest_question_list'], [])

            def test_past_question(self):
                """
                Questions with a pub_date in the past are displayed on the
                index page.
                """
                # 创建测试问题对象
                create_question(question_text="Past question.", days=-30)
                # 模拟访问url
                response = self.client.get(reverse('polls:index'))
                # 测试前端模板里上下文展示和预期数据对象是否相同
                self.assertQuerysetEqual(
                    response.context['latest_question_list'],
                    ['<Question: Past question.>']
                )

## 运行测试
cd 到项目目录下后：

        python manage.py test <app_name>


# 界面
* 静态文件：除html外渲染网络界面的文件
* .css文件（指定字体颜色等）直接放在<app_name>/statics/<app_name>
* 图像文件存储在<app_name>/statics/<app_name>/images下供css文件调用