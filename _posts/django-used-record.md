title: "django使用记录"
date: 2013-12-13 23:11
tags:
- django
categories: 
- Web开发
---
# Django 开发中碰到的问题合集

1. `django`的form问题
	- 问题描述
		使用`Django`的`forms`模块做登录页面时，在点击表单进行登录时， 出现 `'NoneType' object has no attribute '__getitem__'`异常错误。
	- 问题原因
		原因：在实现`Django`的`Form`时，覆盖了`clean`方法，主要代码如下：
		
			def clean(self):
				if not self.is_valid():
					raise forms.ValidationError(u"用户名和密码为必填项")
				else:
					cleaned_data = super(LoginForm, self).clean() 
		
		注意，就是最后一行导致，将cleaned_data中的数据清理完了导致，出现问题描述中的异常信息
	- 解决办法
		原因找到后，那么也就容易了。将`clean`方法体修改如下：

			def clean(self):
				if not self.is_valid():
					raise forms.ValidationError(u"用户名和密码为必填项")
				return self.cleaned_data
				
2. `django`数据库执行原生SQL
	- 问题描述
		使用`Django`的ORM执行原生的SQL语句。
	- 解决方法
		使用如下方式

			look_person = {}
			person_query = Person.objects.raw(
				'SELECT * FROM jkyl_person jp WHERE jp.id IN  (SELECT person_id FROM jkyl_contact_person WHERE contact_id =%s)',
				[person.id])
			for _person in person_query:
				look_person = _person
