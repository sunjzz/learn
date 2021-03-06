上一篇我们已经将表结构通过model迁移至数据库，接下来我们就来通过操作对象的方式来操作数据库，为了方便演示，应用articles的models定义更改如下：

```python
class Account(models.Model):
    username = models.CharField(max_length=64, unique=True, verbose_name="用户名")
    email = models.EmailField(verbose_name="邮箱")
    password = models.CharField(max_length=128, verbose_name="用户密码")
    register_date = models.DateTimeField(auto_now_add=True, verbose_name="注册日期")
    signature = models.CharField(max_length=128, blank=True, null=True, verbose_name="签名")
    avatar = models.ImageField(upload_to='static/image/', default='static/image/default.jpg',
                               verbose_name="用户图像")


class Article(models.Model):
    title = models.CharField(max_length=100, verbose_name="文章标题")
    content = models.TextField(blank=True, null=True, verbose_name="博客正文")
    account = models.ForeignKey('Account', on_delete=models.CASCADE, related_name='art')
    tags = models.ManyToManyField('Tag', null=True, verbose_name="博客标签")
    pub_date = models.DateTimeField(auto_now_add=True, verbose_name="发表时间")
    read_count = models.IntegerField(verbose_name="阅读量", null=True)
    like = models.IntegerField(verbose_name="点赞量", null=True)


class Tag(models.Model):
    name = models.CharField(max_length=64, unique=True, verbose_name='标签名称')
    date = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
```

说明一下，对于articles这个应用虽然只定义了对应三张表关系映射的三个类，但当我们执行完迁移操作，会自动生成四张表，多出来的那张表就是文章与标签之间ManyToMany关系的中间表。

首先进入项目的根目录,执行`python manage.py shell`,下面的操作都将在此环境中演示.

## 创建对象

### 一般情况

1.直接创建

```python
>>>models.Account.objects.create(username = 'moyu', email = 'moyu@aliyun.com', password = '123456')
<Account: Account object (1)>	   #这里（1）表示的是数据库主键id为1
```

还可以通过字典方式创建：

```python
>>>user_info = {'username':'moyu2', 'email':'moyu2@aliyun.com', 'password':'123456'}
>>>models.Account.objects.create(**user_info)
<Account: Account object (2)>
```

2.先生成对象再创建

```python
>>>obj = models.Account(username = 'moyu5', email = 'moyu5@aliyun.com', password = '654321')
>>>obj
<Account: Account object (None)>        #数据并没有落库，所以显示None
>>>obj.save()
>>>obj							#数据已保存至数据库
<Account: Account object (5)>
```

### 外键约束情况

对于有外键约束关系的表再新增数据的时候也有两种方式：

1.生成对象，通过id方式关联

```python
>>>articles_info = {"title": "摸鱼", "content":"摸鱼今天很帅"}
>>>a = modes.Article(**articles_info)
>>>a.account_id = 1
```

这里指定account_id, 1就是上面我们在account表里新增的第一条主键id为1的那条记录的对象

2.生成对象，然后通过对象关联。承上：

```python
>>>a.ccount = obj	 #这个obj就是上面在创建account时生成的对象
>>>a.save()
```

### 多对多情况

在tag表里按照上面的方式创建几条数据。

```python
>>>a.tags.set([1,2])	    #给当前的这个对象的tags设置标签，标签以列表的方式传入，关联的就是tag表的主键id。set等于是赋值操作
>>>a.save()
>>>a.tags.add(3,4)           #追加标签
>>>a.save()
```

当然它还有`create()`方法用于创建一个新对象，并将其放入关联对象的集合中，`remove()`方法用于删除指定关联对象，`clear()`方法删除所有对象。

## 查询所有对象

```python
>>>models.Account.objects.all()	#返回的一个包含所有Account对象的QuerySet对象
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
```

这里要说明一个QuerySet代表来自数据库中对象的一个集合，这里方法all()就是一个包含Account表所有对象的QuerySet对象。可以通过filter和exclude来添加过滤条件进行过滤和反向过滤。

比如：

```python
>>>models.Account.objects.filter(username='moyu')	       #用户名为moyu用户
<QuerySet [<Account: Account object (1)>]>
>>>models.Account.objects.exclude(username='moyu')	    #用户名不为moyu的用户
<QuerySet [<Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
```

QuerySet对象有以下特性:

### 链式过滤器

上面的示例也可以等价写成：

```python
>>>models.Account.objects.all().filter(username='moyu')
<QuerySet [<Account: Account object (1)>]>
>>>models.Account.objects.all().exclude(username='moyu')
<QuerySet [<Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
```

通过这种写法，我们也知道了对QuerySet对象进行过滤的结果仍然是一个QuerySet对象。

### 独一无二

每个QuerySet对象都是独一无二的，比如：

```python
>>>q1 = models.Account.objects.all()
>>>q2 = q1.filter(username='moyu')
```

q1，q2是彼此独立的，q1不受q2的过滤影响

### 惰性机制

创建QuerySet不会和数据库发生任何交互，只有在它被计算时才会执行查询数据库操作。比如：

```python
>>>q1 = models.Account.objects.all()	 #这一步不会和数据库发生交互
>>>q1	#这一步执行才会和数据库发生交互，执行完返回了所有Account对象的QuerySet对象
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
```

### 缓存机制

q1这个QuerySet对象在第一次和数据库发生交互后，其值会被缓存起来，之后再次使用q1时不会再次和数据库发生交互，可以反复使用。但不是所有的情况都会缓存结果，比如使用切片或查询QuerySet的子集的部分数据则不会缓存。比如：

第一种情况：

```python
>>>q1 = models.Account.objects.all()
>>>q1[2]	#访问数据库
<Account: Account object (3)>
>>>q1[2]	#再次访问数据库
<Account: Account object (3)>
```

第二种情况：

```python
>>>q1 = models.Account.objects.all()
>>> [userinfo for userinfo in q1]
[<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]
>>>q1[2]
<Account: Account object (3)>
>>>q1[2]
<Account: Account object (3)>
```

### 可迭代

q1这个QuerySet对象和python的列表和元组类似，语法基本一致，支持切片。

注意的是QuerySet对象不推荐使用len()方法计算元素个数，而推荐使用count()方法来计算元素个数，这种方式效率更高。也不推荐使用bool()方法判断它是否为空，而是推荐使用exists()方法来判断，这种方式效率更高。

## 查询单一对象

我们可以通过get()方法来查询单个对象：

```python
>>>a1 = models.Account.objects.get(id=1)
```

然而，我们上面学习到filter()方法，如果我们写成:

```python
>>>a2 = models.Account.objects.filter(id=1)
```

a1和a2等价吗？结果是否定的，因为a1拿到的是模型id=1这个用户实例的对象，而a2是filter()方法过滤的，得到的一个QuerySet对象，即使这个QuerySet对象里面只有一个元素。需要注意的是使用filter()、exclude()方法操作返回的始终是一个QuerySet对象。

另外当a1的get()拿不到对象时，Django会抛出DoesNotExist异常，而同样条件下的a2则不会报错，而是返回一个空的QuerySet对象。

这里说明一下，Django支持主键查询，因为id就是account这张表的主键，所以上面的也可以写成：

```python
>>>a1 = models.Account.objects.get(pk=1)
>>>a2 = models.Account.objects.get(pk=1)
```

## 查询限制条件多个对象

### 切片

```python
>>>q1 = models.Account.objects.all()[:2]      #返回包含前两个对象的QuerySet对象
```

需要注意，和Python列表相比，操作它时下标必须为非负整数。另外有一个特殊情况:

```python
>>>models.Account.objects.all()[1:5:2]
[<Account: Account object (2)>, <Account: Account object (4)>]
```

这样使用了步长的操作返回的就是一个列表，而不再是QuerySet对象。

### 带过滤条件

**exact**

```python
>>>q1 = models.Account.objects.filter(username__exact='moyu')
>>>q2 = models.Account.objects.filter(username='moyu')
```

q1和q2等价，这里exact是大小写敏感，有iexact表示大小写不敏感

**in**

```python
>>>models.Account.objects.filter(id__in=[1,2,3])	 #用户ID在列表[1,2,3]里面的对象
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(pk__in=[1,2,3])	#与上面等价
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]>
```

**gt, gte, lt, lte**

```python
>>>models.Account.objects.filter(id__gt=3)		#用户ID大于3的对象
<QuerySet [<Account: Account object (4)>, <Account: Account object (5)>]>
>>>models.Account.objects.filter(id__gte=3)	#用户ID大于等于3的对象
<QuerySet [<Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
>>>models.Account.objects.filter(id__lt=3)		 #用户ID小于3的对象
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>]>
>>>models.Account.objects.filter(id__lte=3)	 #用户ID小于等于3的对象
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]
```

**contains, startswith, endswith**

```python
>>>models.Account.objects.filter(id__lte=3, username__contains='oy')	#用于ID小于等于3并且用户名包含字符串moyu的对象，这里contains是大小写敏感
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(id__lte=3, username__icontains='Oy')	#用于ID小于等于3并且用户名包含字符串moyu的对象，这里icontains是大小写不敏感
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(id__lte=3, username__startswith='mo')	#用户ID小于等于3并且是以用户名是以字符串moyu开头的对象，同样有istartswith表示大小写不敏感
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(id__lte=3, username__endswith='yu')     #用户ID小于等于3并且是以用户名是以字符串moyu结尾的对象，同样有iendswith表示大小写不敏感
<QuerySet [<Account: Account object (1)>]>
```

这里说明一下多个限制条件就用逗号隔开，相当于SQL里面的AND。

**range**

```python
>>>models.Account.objects.filter(register_date__range=('2021-01-28', '2021-01-29'))	#用户注册时间在2021-01-28至2021-01-29的对象
<QuerySet [<Account: Account object (2)>]>
```

注意register_date是一个datetime类型，过滤的条件写的是date格式，实际在查询的时候会转换成2021-01-29 00:00:00，显然这个不包含2021-01-29那天的用户。

**date, year, month, day, hour, minute, second, week, week_day**

```python
>>>models.Account.objects.filter(register_date__date='2021-01-28') 	 #用户注册时间在2021-01-28日的对象
<QuerySet [<Account: Account object (2)>]>
>>>models.Account.objects.filter(register_date__year=2021) 		    #用户注册时间在2021年的对象
<QuerySet [<Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
>>>models.Account.objects.filter(register_date__year__gte=2021) 	      #用户注册时间大于等于2021年的对象
<QuerySet [<Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
>>>models.Account.objects.filter(register_date__month=1) 			 #用户注册时间在5月的对象（每年的5月）
<QuerySet [<Account: Account object (2)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(register_date__day=30)			 #用户注册时间在30号的对象（每年每月的30号）
<QuerySet [<Account: Account object (3)>]>
>>>models.Account.objects.filter(register_date__hour=0) 			  #用户注册时间在0点的对象，同样还支持minute，second
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(register_date__week=5)			 #用户注册时间在每年第5周的对象
<QuerySet [<Account: Account object (4)>, <Account: Account object (5)>]>
>>>models.Account.objects.filter(register_date__week_day=3)		  #用户注册时间在每周二的对象 ，Sunday(1) --> Staturday(7)
<QuerySet [<Account: Account object (4)>, <Account: Account object (5)>]>
```

**isnull**

```python
>>>models.Account.objects.filter(register_date__isnull=True)	#用户注册时间为空的对象
```

**regex**

```python
>>>models.Account.objects.filter(username__regex=r'^(m|z)')        #用户注册用户名以m者z开头的的对象，大小写敏感，同样有__iregex用法
<QuerySet [<Account: Account object (1)>, <Account: Account object (2)>, <Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (5)>]>
```

### 排序

```python
>>> models.Account.objects.filter(pk__gt=2)		#主键id大于2的用户对象
<QuerySet [<Account: Account object (3)>, <Account: Account object (4)>]>
>>>models.Account.objects.filter(pk__gt=2).order_by('-register_date')	#主键id大于2且以注册时间降序返回用户对象，使用order_by指定字段默认时升序，加个-表示降序。
<QuerySet [<Account: Account object (4)>, <Account: Account object (3)>]>
```

还可以使用？表示随机排序，比如：

```python
>>>models.Account.objects.filter(pk__gte=2).order_by('?')
<QuerySet [<Account: Account object (4)>, <Account: Account object (3)>, <Account: Account object (2)>]>
>>>models.Account.objects.filter(pk__gte=2).order_by('?')
<QuerySet [<Account: Account object (2)>, <Account: Account object (4)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(pk__gte=2).order_by('?')
<QuerySet [<Account: Account object (3)>, <Account: Account object (4)>, <Account: Account object (2)>]>
```

还可以使用reverse()方法来反向返回查询的顺序，需要注意的是该方法一般只在有定义顺序的QuerySet上调用，比如定义了默认顺序或者使用了order_by()，如果没有定义排序，那么使用该方法也将没有意义。

```python
>>>models.Account.objects.filter(pk__gt=2).order_by('register_date')
<QuerySet [<Account: Account object (3)>, <Account: Account object (4)>]>
>>>models.Account.objects.filter(pk__gt=2).order_by('register_date').reverse()
<QuerySet [<Account: Account object (4)>, <Account: Account object (3)>]>
>>>models.Account.objects.filter(pk__gt=2).order_by('register_date').reverse()[:1]	#取排序后的最后一个对象（QuerySet切片不支持负整数）
<QuerySet [<Account: Account object (3)>]>
```

## 跨关系查询

这个就类似SQL中的JOIN操作，比如上面Articles类的account字段外键关联Account类，那么我们可以有如下操作：

```python
>>>a1 = models.Article.objects.filter(account__username='moyu')	      #查询发表文章的所有者account的username为moyu的所有文章对象
```

可以看到跨模型查询操作时使用关联字段名account，字段名由双下划线分割，这里字段用的是username。

也可以反向跨关系查询，默认情况使用模型的小写名称在查找中引用反向关系，比如：

```python
>>>a2 = models.Account.objects.filter(article__content__contains='摸鱼今天很帅')	#查找文章内容包含”摸鱼今天很帅“的作者
```

当然你也可以继续使用关联字段，跨关系深度不受限制。

```python
>>>a3 = models.Account.objects.filter(article__tags__name="生活")				#查找文章标签名有”生活“的作者
```

## 模型字段查询

上面学习的都是filter()都是模型字段与常量做比较，Django提供一种模型字段与同一模型的另一字段做比较查询。这个就是F()

```python
>>>from django.db.models import F	# 导入F类
>>>a1 = models.Article.objects.filter(read_count__gt=F('like') * 2 )        #返回阅读量大于两倍点赞量的所有文章的对象
```

甚至F()还支持通过双下划线跨关系查询。例如要查询所有文章标签与文章作者名字相同的文章对象：

```python
>>>a2 = models.Article.objects.filter(account__username=F('tags__name'))
```

## 复杂查询

上面再学习filter()时我们知道，如果需要同时满足多个条件时，只需要使用逗号就可以，类似于SQL里写where条件的and操作，那怎么进行or操作呢？这就需要使用Django为我们提供的Q()。Q对象能通过 & 和| 操作符连接起来。当操作符被用于两个Q对象之间时会生成一个新的Q对象。

```python
>>>from django.db.models import Q 	#导入Q类
>>>a3 = models.Account.objects.filter(Q(username\__startswith='moyu') | Q(email__endswith='aliyun.com'))	#返回用户名以'moyu'开头的的对象或者用户邮箱是以'aliyun.com'结尾的对象
```

上面的|换成&就是且的关系。Q对象还可以通过~支持反转操作，比如：

```python
>>>a4 = models.Account.objects.filter(Q(username__startswith='moyu') | ~Q(email__endswith='aliyun.com'))	        #返回用户名以'moyu'开头的的对象或者用户邮箱不是以'aliyun.com'结尾的对象
```

我们还可以混合使用Q对象和关键字参数进行更复杂的查询操作，比如：

```python
>>>a5 = models.Account.objects.filter(Q(register_date='2021-01-29')| Q(register_date='2021-01-30'), username__startswith='moyu')	#返回用户名以'moyu'开头的并且用户注册时间是2021-01-29或者2021-01-30号的对象
```

这里需要注意如果查询提供了Q对象和关键字参数，Q对象必须位于所有关键字参数之前。下面这种写法是不生效的，且会抛出异常：

```python
>>>a6 = models.Account.objects.filter(username__startswith='moyu', Q(register_date='2021-01-29')| Q(register_date='2021-01-30'))
  File "<console>", line 1
    a6 = models.Account.objects.filter(username__startswith='moyu', Q(register_date='2021-01-29')| Q(register_date='2021-01-30'))
                                                                                                                                ^
SyntaxError: positional argument follows keyword argument
```

## 关联对象

如果我们是通过get()方法则会得到一个model的对象，那么同样我们可以在一对多关联，多对多关联的情况下进行有关操作。

### 正向关联

```python
>>>a1 = models.Article.objects.get(pk=1)
>>>a1.account		  #通过a1的account属性拿到该文章关联的Account对象
<Account: Account object (1)>
```

### 反向关联

同样，我们可以在关联对象的另一边拿到有关对象，比如通过一个Account对象拿到与之关联的所有的Article对象

```python
>>>u1 = models.Account.objects.get(pk=1)
>>>u1.article_set.all()
<QuerySet [<Article: Article object (1)>, <Article: Article object (2)>]>
```

注意，这里默认的类名是模型名小写加_set查询，返回的是一个QuerySet对象。可以在ForeignKey时设置related_name参数重写这个类名。

```python
class Article(models.Model): 
    ......   
    account = models.ForeignKey('Account', on_delete=models.CASCADE, related_name='art')
    ......
```

上面的就可以改写成：

```python
>>>u1.art.all()
```

需要注意的是一对一关系比较特殊，它的反向关联查询时返回的仅仅是一个对象，而不是一个只包含一个对象的QuerySet对象。

## 删除和修改

掌握了上面的一堆查询操作，删除和修改就比较容易了。下面就来两个示例：

```python
>>>models.Account.objects.filter(username='moyu5').delete()
(1, {'articles.Account': 1})
>>>models.Account.objects.filter(username='moyu4').update(password='135246')
1
```

## 总结

1. 其他ORM框架要求在模型的两边都定义关联关系，而Django仅要求在模型的一边定义关联关系即可，这是因为Django开发者认为这违反DRY原则，而Django之所以能这样，是因为在项目的settings.py有INSTALLED_APPS配置定义了项目的各个应用注册。而Django会帮忙持续追踪这些关联，并且在关联模型导入时添加关联关系。所以我们在项目开发过程中一定不要忘记在这里添加应用注册 ，否则反向关联将不能正常工作。
2. ORM并不是万金油，毕竟是基于原生SQL做了封装，它所生成的代码还是要转成SQL语句来操作，会牺牲一定的性能。所以在复杂场景下我们还是需要DBA帮忙写一条很牛逼的SQL。但在绝大部分场景下它还是有其存在的价值。