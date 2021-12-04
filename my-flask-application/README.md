# my-flask-application

https://www.serverless.com/blog/flask-python-rest-api-serverless-lambda-dynamodb

## メモ

### Local development configuration with Serverless offline plugin

`sls dynamodb start` で以下のようなエラーが発生する。

```
Dec 04, 2021 11:02:22 PM com.almworks.sqlite4java.Internal log
SEVERE: [sqlite] SQLiteQueue[]: stopped abnormally, reincarnation is not possible for in-memory database
Dec 04, 2021 11:02:24 PM com.almworks.sqlite4java.Internal log
WARNING: [sqlite] cannot open DB[3]: com.almworks.sqlite4java.SQLiteException: [-91] cannot load library: java.lang.UnsatisfiedLinkError: /Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib: dlopen(/Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib, 0x0001): tried: '/Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib' (fat file, but missing compatible architecture (have 'i386,x86_64', need 'arm64e')), '/usr/local/lib/libsqlite4java-osx.dylib' (no such file), '/usr/lib/libsqlite4java-osx.dylib' (no such file)
Dec 04, 2021 11:02:24 PM com.almworks.sqlite4java.Internal log
SEVERE: [sqlite] SQLiteQueue[]: error running job queue
com.almworks.sqlite4java.SQLiteException: [-91] cannot load library: java.lang.UnsatisfiedLinkError: /Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib: dlopen(/Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib, 0x0001): tried: '/Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib' (fat file, but missing compatible architecture (have 'i386,x86_64', need 'arm64e')), '/usr/local/lib/libsqlite4java-osx.dylib' (no such file), '/usr/lib/libsqlite4java-osx.dylib' (no such file)
        at com.almworks.sqlite4java.SQLite.loadLibrary(SQLite.java:97)
        at com.almworks.sqlite4java.SQLiteConnection.open0(SQLiteConnection.java:1441)
        at com.almworks.sqlite4java.SQLiteConnection.open(SQLiteConnection.java:282)
        at com.almworks.sqlite4java.SQLiteConnection.open(SQLiteConnection.java:293)
        at com.almworks.sqlite4java.SQLiteQueue.openConnection(SQLiteQueue.java:464)
        at com.almworks.sqlite4java.SQLiteQueue.queueFunction(SQLiteQueue.java:641)
        at com.almworks.sqlite4java.SQLiteQueue.runQueue(SQLiteQueue.java:623)
        at com.almworks.sqlite4java.SQLiteQueue.access$000(SQLiteQueue.java:77)
        at com.almworks.sqlite4java.SQLiteQueue$1.run(SQLiteQueue.java:205)
        at java.base/java.lang.Thread.run(Thread.java:833)
Caused by: java.lang.UnsatisfiedLinkError: /Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib: dlopen(/Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib, 0x0001): tried: '/Users/gky360/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/.dynamodb/DynamoDBLocal_lib/libsqlite4java-osx.dylib' (fat file, but missing compatible architecture (have 'i386,x86_64', need 'arm64e')), '/usr/local/lib/libsqlite4java-osx.dylib' (no such file), '/usr/lib/libsqlite4java-osx.dylib' (no such file)
        at java.base/jdk.internal.loader.NativeLibraries.load(Native Method)
        at java.base/jdk.internal.loader.NativeLibraries$NativeLibraryImpl.open(NativeLibraries.java:384)
        at java.base/jdk.internal.loader.NativeLibraries.loadLibrary(NativeLibraries.java:228)
        at java.base/jdk.internal.loader.NativeLibraries.loadLibrary(NativeLibraries.java:170)
        at java.base/jdk.internal.loader.NativeLibraries.findFromPaths(NativeLibraries.java:311)
        at java.base/jdk.internal.loader.NativeLibraries.loadLibrary(NativeLibraries.java:283)
        at java.base/java.lang.ClassLoader.loadLibrary(ClassLoader.java:2422)
        at java.base/java.lang.Runtime.loadLibrary0(Runtime.java:818)
        at java.base/java.lang.System.loadLibrary(System.java:1989)
        at com.almworks.sqlite4java.Internal.tryLoadFromSystemPath(Internal.java:352)
        at com.almworks.sqlite4java.Internal.loadLibraryX(Internal.java:124)
        at com.almworks.sqlite4java.SQLite.loadLibrary(SQLite.java:95)
        ... 9 more
```

M1 Mac ではそのままでは動かない模様なので、 `sls dynamodb start` 代わりに docker image を使う。
config を少し修正する。

```yaml
custom:
  ...
  dynamodb:
    stages:
      - dev
    start:
      port: '8000' # the port of our Dynamo docker container
      noStart: true
      migrate: true
```

docker container を起動する。

```
docker run -p 8000:8000 amazon/dynamodb-local
```

docker container を走らせた状態で `sls dynamodb start` する。

しかしながら、以下のようなエラーでアプリケーションから dynamodb local にアクセスできない。

```
Traceback (most recent call last):
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/flask/app.py", line 2091, in __call__
    return self.wsgi_app(environ, start_response)
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/flask/app.py", line 2076, in wsgi_app
    response = self.handle_exception(e)
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/flask/app.py", line 2073, in wsgi_app
    response = self.full_dispatch_request()
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/flask/app.py", line 1518, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/flask/app.py", line 1516, in full_dispatch_request
    rv = self.dispatch_request()
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/flask/app.py", line 1502, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**req.view_args)
  File "/Users/yinagaki/dev/src/github.com/gky360/learn-serverless-framework/my-flask-application/app.py", line 28, in get_user
    resp = client.get_item(
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/botocore/client.py", line 391, in _api_call
    return self._make_api_call(operation_name, kwargs)
  File "/Users/yinagaki/.pyenv/versions/3.8.12/envs/my-flask-application/lib/python3.8/site-packages/botocore/client.py", line 719, in _make_api_call
    raise error_class(parsed_response, operation_name)
botocore.errorfactory.ResourceNotFoundException: An error occurred (ResourceNotFoundException) when calling the GetItem operation: Cannot do operations on a non-existent table
```
