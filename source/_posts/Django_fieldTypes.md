layout: post
title: "python-django"
date: 2017-2-23 
categories: Python3
comments: false
tags: django
---

#### 常见的field类型： 
1. AutoField 自增字段，它是一个根据ID自增长的IntegerField字段，通常不用自己设置，如果没有设置主键，django会自动添加它为主键字段

2. CharField(max_length=none[, **options]) 一个字符串字段，必须有一个最大长度的参数，它作用于数据库层级和django数据验证层级。 django的管理后台用 单行输入框来表示它。

3. CommaSeparatedIntegerField（max_length=none[, **options]） 用来存放以逗号间隔的整数序列，有必须的最大长度属性，考虑到数据库的移植性，max_length参数应该必选。
4. DateField([auto_now=False, auto_now_add=False, **options]) 该字段利用python的datetime.date实例来表示日期，第一次保存对象时候，有auto_now参数自动保存当前时间，一般用来表示最后修改时间， auto_now_add在第一次创建对象时候django将该字段值自动设置为当前时间，用来表示对象创建时间。 django管理后台使用一个带有javascript日历的 来表示该字段，带有一个当前日期的快捷选择。
<!-- more -->
5. DateTimeField([auto_now=False, auto_now_add=False, **options]) 该字段利用datetime.datetime实例来分表表示时间和日期，类似DateField。django后台使用2个来分别表示日期和时间，同样带有javascript快捷选项。

6. DecimalField（max_digits=None，decimal_places=None[, **options]）用decimal实例表示固定精度的十进制数的字段，有两个必须参数，max_digits数字允许的最大位数，decimal_places小数的最大位数。django后台用表示该字段， 通常用来表示金额

7. EmailField([maxlength=75, **options]) 带有email合法性检测的一个CharField

8. FileField(upload_to=None[, max_length=100, **options])文件上传字段，该字段不支持primary_key和unique参数，否则类型错误。有一个必须参数upload_to，用于保存文件的本地文件系统

9. FilePathField(path=None[, match=None, recursive=False, max_length=100， **options])他是一个CharField，用来选择文件系统下某个目录里面的某些文件，它有三个专有参数，只有path是必须的。path是一个目录的绝对路径，match是一个正则表达式字符串，用来过滤文件名称；recursive为bool，指定是否包含path下的子目录。

10. BigIntegerField 
64位的整型数值 (-2^63) – (2^63-1)

11. BinaryField 
存储原始的2进制数据，功能有限，仅支持字节分配

12. BolleanField 
布尔型和NullBooleanField有区别，true/false，本类型不允许出现null。

13. FloatField 
与 python 里的 float 实例相同，django使用来表示它，虽然 FloatField 与 DecimalField 都是表示实数，但却是不同的表现形式，FloatField 用的是 python d float 类型，但是 DecimalField 用的却是 Decimal 类型。

14. ImageField([upload_to=None, height_field=None, width_field=None, max_length=100, **options]) 
继承了FileField的所有属性和方法。参数除upload_to外，还有height_field，width_field等属性。

15. IntegerField 
[-2147483648,2147483647 ]的取值范围对Django所支持的数据库都是安全的。

16. IPAddressField 
点分十进制表示的IP地址，如10.0.0.1

17. GenericIPAddressField 
ip v4和ip v6地址表示，ipv6遵循RFC 4291section 2.2,

18. NullBooleanField 
可以包含空值的布尔类型，相当于设置了null=True的BooleanField。

19. PositiveIntegerField 
正整数或0类型，取值范围为[0 ,2147483647]

20. PositiveSmallIntegerField 
正短整数或0类型，类似于PositiveIntegerField，取值范围依赖于数据库特性，[0 ,32767]的取值范围对Django所支持的数据库都是安全的。

21. SlugField 
只能包含字母，数字，下划线和连字符的字符串，通常被用于URLs表示。可选参数max_length=50，prepopulate_from用于指示在admin表单中的可选值。db_index，默认为True。

22. SmallIntegerField 
小整数字段，类似于IntegerField，取值范围依赖于数据库特性，[-32768 ,32767]的取值范围对Django所支持的数据库都是安全的。

23. TextField 
文本类型

24.TimeField 
时间，对应Python的datetime.time

25.URLField 
存储URL的字符串，默认长度200；verify_exists(True)，检查URL可用性。





