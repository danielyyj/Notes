##项目基本配置
Config类
先在当前类中定义配置的类，并从中加载配置
````
app = Flask(__name__)

class Config(object):
    """工程配置信息"""
    DEBUG = True

app.config.from_object(Config)
运行测试
````
SQLAlchemy
导入数据库扩展，并在配置中填写相关配置
from flask_sqlalchemy import SQLAlchemy
````
...

class Config(object):
    """工程配置信息"""
    DEBUG = True
    # 数据库的配置信息
    SQLALCHEMY_DATABASE_URI = "mysql://root:mysql@127.0.0.1:3306/information"
    SQLALCHEMY_TRACK_MODIFICATIONS = False

app.config.from_object(Config)
db = SQLAlchemy(app)
在终端创建数据库
mysql> create database information charset utf8;
运行测试
````
Redis
创建redis存储对象，并在配置中填写相关配置
import redis
...
````
class Config(object):
    """工程配置信息"""
    ...
    # redis配置
    REDIS_HOST = "127.0.0.1"
    REDIS_PORT = 6379

app.config.from_object(Config)
db = SQLAlchemy(app)
redis_store = redis.StrictRedis(host=Config.REDIS_HOST, port=Config.REDIS_PORT)
运行测试
````

CSRF
包含请求体的请求都需要开启CSRF
````
from flask_wtf.csrf import CSRFProtect
...
app.config.from_object(Config)
...
CSRFProtect(app)
CSRFProtect只做验证工作，cookie中的 csrf_token 和表单中的 csrf_token 需要我们自己实现
````
Session
利用 flask-session扩展，将 session 数据保存到 Redis 中
from flask_session import Session
...
```class Config(object):
    """工程配置信息"""
    SECRET_KEY = "EjpNVSNQTyGi1VvWECj9TvC/+kq3oujee2kTfQUs8yCM6xX9Yjq52v54g+HVoknA"
    ...
    # flask_session的配置信息
    SESSION_TYPE = "redis" # 指定 session 保存到 redis 中
    SESSION_USE_SIGNER = True # 让 cookie 中的 session_id 被加密签名处理
    SESSION_REDIS = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT) # 使用 redis 的实例
    PERMANENT_SESSION_LIFETIME = 86400 # session 的有效期，单位是秒

app.config.from_object(Config)
...
Session(app)
运行测试

文档地址：http://pythonhosted.org/Flask-Session/

Flask-Script与数据库迁移扩展
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand
...
manager = Manager(app)
Migrate(app, db)
manager.add_command('db', MigrateCommand)
...
```

if __name__ == '__main__':
    manager.run()
    
   



----------------------------------------------------------
log日志相关配置：
集成日志到当前项目
在 config.py 文件中在不同的环境的配置下添加日志级别
class Config(object):
    ...

    # 默认日志等级
    LOG_LEVEL = logging.DEBUG


class ProductionConfig(Config):
    """生产模式下的配置"""
    LOG_LEVEL = logging.ERROR
在 info 目录下的 init.py 文件中添加日志配置的相关方法
def setup_log(config_name):
    """配置日志"""

    # 设置日志的记录等级
    logging.basicConfig(level=config[config_name].LOG_LEVEL)  # 调试debug级
    # 创建日志记录器，指明日志保存的路径、每个日志文件的最大大小、保存的日志文件个数上限
    file_log_handler = RotatingFileHandler("logs/log", maxBytes=1024 * 1024 * 100, backupCount=10)
    # 创建日志记录的格式 日志等级 输入日志信息的文件名 行数 日志信息
    formatter = logging.Formatter('%(levelname)s %(filename)s:%(lineno)d %(message)s')
    # 为刚创建的日志记录器设置日志记录格式
    file_log_handler.setFormatter(formatter)
    # 为全局的日志工具对象（flask app使用的）添加日志记录器
    logging.getLogger().addHandler(file_log_handler)
在 create_app 方法中调用上一步创建的方法，并传入 config_name
def create_app(config_name):
    ...

    # 配置项目日志
    setup_log(config_name)
    app = Flask(__name__)
    ...
在项目根目录下创建日志目录文件夹 logs：


运行项目，当前项目日志已输出到 logs 的目录下自动创建的 log 文件中

在 logs 文件夹下创建 .gitkeep 文件，以便能将 logs 文件夹添加到远程仓库，并在 .gitignore 文件中添加忽略提交生成的日志文件
logs/log*
在 Flask框架 中，其自己对 Python 的 logging 进行了封装，在 Flask 应用程序中，可以以如下方式进行输出 log:

current_app.logger.debug('debug')
current_app.logger.error('error')