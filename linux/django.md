1. install

   ```
   sudo pip3 install Django
   ```

2. 

3. django new project

```shell
django-admin.py startproject hellodjango
```

目录下创建hellodjango项目

目录结构

```
$ cd hellodjango/
$ tree
.
|-- hellodjango
|   |-- __init__.py
|   |-- asgi.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```

目录说明：

- **HelloWorld:** 项目的容器。
- **manage.py:** 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
- **HelloWorld/__init__.py:** 一个空文件，告诉 Python 该目录是一个 Python 包。
- **HelloWorld/asgi.py:** 一个 ASGI 兼容的 Web 服务器的入口，以便运行你的项目。
- **HelloWorld/settings.py:** 该 Django 项目的设置/配置。
- **HelloWorld/urls.py:** 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"。
- **HelloWorld/wsgi.py:** 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。

4. django start

python3 manage.py runserver 0.0.0.0:8000

5. host invalid错误

```
DisallowedHost at /
Invalid HTTP_HOST header: 'zeeham.com:8000'. You may need to add 'zeeham.com' to ALLOWED_HOSTS.
```

​     cd hellodjango

​    vim settings.py

   把zeeham.com添加到ALLOWED_HOSTS中