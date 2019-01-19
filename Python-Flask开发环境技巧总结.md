### 深度定制Python的Flask框架开发环境的一些技巧总结

1. Flask 环境配置
    使用 virtualenv 管理环境
        pip install virtualenv
        $ virtualenv venv
        $ source venv/bin/activate
        使用 Pip 安装的依赖包会被下载到虚拟环境中而不是全局系统
        停用虚拟环境
        (venv)$ deactivate

    管理项目库（创建虚拟环境目录给项目库带来的混乱）
        virtualenvwrapper
        $ mkvirtualenv rocket

    安装依赖包
        依赖包的列表，requirements.txt 是一个文本文件
        (rocket)$ pip freeze > requirements.txt
        $ workon fresh-env

    人工管理依赖包
        运行应用必须的包放入 require\_run.txt 
        开发应用程序需要的包放入 require\_dev.txt 

    版本控制
        Git
        推荐使用 .gitignore 来控制不需要版本控制的文件
        *.pyc
        instance/
        Instance 文件夹是用于以一种更安全地方式提供给你的应用程序敏感配置变量
    
    调试
        debug = True 不要在生产环境中使用调试模式！
        交互式控制台允许执行任意代码并会是一个巨大的安全漏洞

        Flask-DebugToolbar 推荐使用调试工具
        在调试模式下，它会把一个侧边栏置于你的应用程序的每一页上。
        侧边栏提供了有关 SQL 查询，日志记录，版本，模板，
        配置和其它有趣的信息，使得更容易地跟踪问

2. Flask 工程配置
    config.py 中定义一些变量接着一切就能工作
    
    2-1）管理生产应用的配置的时候
    需要保护 API 密钥以及为不同的环境使用不同的配置（例如，开发和生产环境）
    DEBUG = True 
    # Configuration for the Flask-Bcrypt extension
    BCRYPT_LEVEL = 12 
    # For use in application emails
    MAIL_FROM_EMAIL = "robert@example.com" 

    2-2）需要定义包含敏感信息的配置变量
    隐藏像数据库密码以及 API 密钥的一些敏感信息，或者定义于特定于指定机器的配置变量
    Flask 提供 instance folders 的功能实现（不希望它提交到版本控制）
    
    实例文件
    config.py
    requirements.txt
    run.py
    instance/
        config.py
    yourapp/
        __init__.py
        models.py
        views.py
        templates/
        static/

    使用实例文件
    我们使用 app.config.from_pyfile() 来从一个实例文件夹中加载配置变量。
    当我们调用 Flask() 来创建我们的应用的时候，如果我们设置了 instance_relative_config=True， app.config.from_pyfile() 将会从 instance/ 目录加载指定文件

    # app.py or app/__init__.py
    app = Flask(__name__, instance_relative_config=True)
    app.config.from_object('config')
    app.config.from_pyfile('config.py')

    实例文件夹的隐私性成为在其里面定义不想暴露到版本控制的密钥的最佳候选
    我们通常要求其他用户或者贡献者使用自己的密钥
    # instance/config.py
    SECRET_KEY = 'Sm9obiBTY2hyb20ga2lja3MgYXNz'
    STRIPE_API_KEY = 'SmFjb2IgS2FwbGFuLU1vc3MgaXMgYSBoZXJv'
    SQLALCHEMY_DATABASE_URI= \\"postgresql://user:TWljaGHFgiBCYXJ0b3N6a2lld2ljeiEh@localhost/databasename"

    基于环境的配置
    # config.py
    DEBUG = False
    SQLALCHEMY_ECHO = False
    # instance/config.py
    DEBUG = True
    SQLALCHEMY_ECHO = True

    基于环境变量配置
    requirements.txt
    run.py
    config/
        __init__.py  
        default.py
        production.py
        development.py
        staging.py
    instance/
        config.py
    yourapp/
        __init__.py
        models.py
        views.py
        static/
        templates/
    
    在上面的文件列表中我们有多个不同的配置文件
    为了决定要加载哪个配置文件，我们会调用 'app.config.from_envvar()'
    # yourapp/\\_\\_init\\_\\_.py
    app = Flask(__name__, instance_relative_config=True)
    
    app.config.from_object('config.default')
    
    app.config.from_pyfile('config.py')
    variable# Variables defined here will override those in the default    configuration
    app.config.from_envvar('APP_CONFIG_FILE')

    环境变量的值应该是配置文件的绝对路径
    运行在一个普通的 Linux 服务器上，我们可以编写一个设置环境变量的 shell 脚本并且运行 run.py
    # start.sh
    APP\\_CONFIG\\_FILE=/var/www/yourapp/config/production.py
    python run.py

