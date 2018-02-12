
> To give a detailed introduction about request.post. we will make the comment fuction to show how request.post works.

- Create comment database in models.py

- views.py / form.py

- template

- ValidationError


# 1. Caselog details html page

Create template log_detail.html

# 2. Create comment model / db

```Python
class Comment(models.Model):
    name = models.CharField(null=True, blank=True,max_length=20)
    comment = models.TextField()
    def __str__(self):
      return self.comment
```
Use the commands to generate/create db.

```Python
python manage.py makemigrations
python manage.py migrate
```

# 3. views.py

```Python
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
```

Create form.py

```Python
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField(max_length=50)
    comment = forms.CharField()
```

> models.Model may look same as forms.Form, BUT totally different!!!

# 4. Template log_detail.html

Rewirte html page by django template language.
change the static path before moving on ```{% load static%}```, ```"{% static `css\semantic.css` &}"```.

```HTML
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
```

```HTML
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
```

explanation.

- {{ form.as_p }} means wrap each field of form in a <p></p>

![1]()
![2]()

- when we try to submit the form, then csrf error occurred.
  {{ csrf_token }} means let the broswer know you are you.

![3]()  
![4]()  

- Django.form validates error automatically
  Use print(form.error) to have a look at what the error is.
  Although we will make validation conditions by ourself.

```Python
if request.method == 'POST':
    form = CommentForm(request.POST) #绑定表单，实现数据校验
        print('==='*30)
        print(form)
        print('==='*30)
context = {}
comment_list = Comment.objects.all()
```

![5]()  

- Save the data that we input in the form

```Python
if request.method == 'POST':
    form = CommentForm(request.POST) #绑定表单，实现数据校验
+    if form.is_valid():
+        name = form.cleaned_data['name']
+        comment = form.cleaned_data['comment']
+        c = Comment(name = name, comment = comment)
+        c.save()
+        return redirect(to='detail') # 'name' in urls
```

![6]()  
