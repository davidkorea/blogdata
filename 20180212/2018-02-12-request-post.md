---
layout: post
title: "Request POST method"
date: 2018-02-11
tags: request
---

> To give a detailed introduction about request.post. we will make the comment fuction to show how request.post works.

- Create comment database in models.py

- views.py / form.py

- template

- ValidationError


# 1. Caselog details html page

Create template log_detail.html and define a url for this page in urls.py.

```Python
{% raw %}
from django.conf.urls import url
from django.contrib import admin
from webapp.views import default, detail

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^default/', default, name='default'),
    url(r'^detail/', detail, name='detail'),
]
{% endraw %}
```

# 2. Create comment model / db

```Python
{% raw %}
class Comment(models.Model):
    name = models.CharField(null=True, blank=True,max_length=20)
    comment = models.TextField()
    def __str__(self):
      return self.comment
{% endraw %}
```
Use the commands to generate/create db.

```Python
{% raw %}
python manage.py makemigrations
python manage.py migrate
{% endraw %}
```

# 3. views.py

```Python
{% raw %}
from django.shortcuts import render, HttpResponseRedirect, redirect
from webapp.models import Caselog, Comment
from webapp.form import CommentForm # This function could define here.

def detail(request):
    if request.method == 'GET':
        form = CommentForm #创建表单
    if request.method == 'POST':
        form = CommentForm(request.POST) #绑定表单，实现数据校验
        if form.is_valid():
            name = form.cleaned_data['name']
            comment = form.cleaned_data['comment']
            c = Comment(name = name, comment = comment)
            c.save()
            return redirect(to='detail') # 'name' in urls
    context = {}
    comment_list = Comment.objects.all()
    context['comment_list'] = comment_list
    context['form'] = form
    return render(request, 'log_detail.html', context)
{% endraw %}
```

Create form.py

```Python
{% raw %}
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField(max_length=50)
    comment = forms.CharField()
{% endraw %}
```

> models.Model may look same as forms.Form, BUT totally different!!!

# 4. Template log_detail.html

Rewirte html page by django template language.

Using ```{% load static%}```, ```"{% static `css\semantic.css` &}"``` to change the static path before moving on .


```HTML
{% raw %}
<div class="ui comments">
    {% for comment in comment_list %}
        <div class="comment">
            <div class="avatar">
                <img src="http://semantic-ui.com/images/avatar/small/matt.jpg" alt="" />
            </div>
            <div class="content">
                <a href="#" class="author">{{ comment.name }}</a>
                <div class="metadata">
                    <div class="date">2 days ago</div>
                </div>
                <p class="text" style="font-family: 'Raleway', sans-serif;">
                    {{ comment.comment }}
                </p>
            </div>
        </div>
    {% endfor %}
</div>
{% endraw %}
```

```HTML
{% raw %}
<form class="ui form" action="" method="post">
-    <div class="field">
-        <label> name</label>
-        <input type="text" name="name" value="">
-    </div>
-    <div class="field">
-        <label>comment</label>
-        <textarea></textarea>
-    </div>

+  # {{ form }}

+   {{ form.as_p }}
+   {{ csrf_token }}

    <button type="submit" class="ui blue button" >Click</button>
</form>
{% endraw %}
```

Explanation.

- {{ form.as_p }} means wrap each field of form in a `<p></p>`

![1](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/1.png)

![2](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/2.png)

- when we try to submit the form, then csrf error occurred.
  {{ csrf_token }} means let the broswer know you are you.

![3](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/3.png)  

![4](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/4.png)  

- Django.form validates error automatically
  Use print(form.error) to have a look at what the error is.
  Although we will make validation conditions by ourself.

```Python
{% raw %}
if request.method == 'POST':
    form = CommentForm(request.POST) #绑定表单，实现数据校验
        print('==='*30)
        print(form)
        print('==='*30)
context = {}
comment_list = Comment.objects.all()
{% endraw %}
```

![5](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/5.png)  

- Save the data that we input in the form

```Python
{% raw %}
if request.method == 'POST':
    form = CommentForm(request.POST) #绑定表单，实现数据校验
+    if form.is_valid():
+        name = form.cleaned_data['name']
+        comment = form.cleaned_data['comment']
+        c = Comment(name = name, comment = comment)
+        c.save()
+        return redirect(to='detail') # 'name' in urls
{% endraw %}
```

![6](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/6.png)  

> By Now, comment function can work well.

-----

# 5. Customize django.form ValidationError

## 5.1 form.py

```Python
{% raw %}
from django import forms
from django.core.exceptions import ValidationError

def words_validator(comment):
    if len(comment) < 4:
        raise ValidationError('Not Enough Letters')

def comment_validator(comment):
    if 'a' in comment:
        raise ValidationError('Do Not Use the Word!')

class CommentForm(forms.Form):
    name = forms.CharField(max_length=50)
    comment = forms.CharField(
        widget=forms.Textarea(), # change CharField to TextField
        error_messages={
            'required': 'why no words?' # pop up error_messages
        },
        validators=[words_validator, comment_validator] # Customized error conditions
)
{% endraw %}
```

## 5.2 log_detail.html, customize render form.

```HTML
{% raw %}
<form class="ui form" action="" method="post">
-      {{ form.as_p }}

+ #     {% for field in form %}
+ #        <div class="field">
+ #             {{ field.label }}
+ #             {{ field }}
+ #         </div>
+ #     {% endfor %}

    {% if form.errors %}

        <div class="ui error message">
            {{ form.errors }}
        </div>
        {% for field in form %}
            <div class=" {{ field.errors|yesno:'error, '}} field">
                {{ field.label }}
                {{ field }}
            </div>
        {% endfor %}

    {% else %}

        {% for field in form %}
            <div class="field">
                {{ field.label }}
                {{ field }}
            </div>
        {% endfor %}

    {% endif %}

    {% csrf_token %}

    <button type="submit" class="ui blue button" >Click</button>
{% endraw %}
```

- The key point is that the error field will have a error css of form，
    ```<div class=" {{ field.errors|yesno:'error, '}} field">```
  which means if field.errors is yes, then ```<div class=" error field">```,
  or it will be ```<div class=" field">```.

![7](https://raw.githubusercontent.com/davidkorea/blogdata/master/20180212/image/7.png)  
