# GitHub Token使用

GitHub上传代码使用Token的方式进行校验，因此现在需要使用Token的方式来鉴权，否则无法通过Git push更新代码



# 生成token

首先需要在GitHub网站上生成token。路径：

```shell
settings -> developer settings -> personal access tokens -> generate new token
```

一般生成的token只会在生成的时候以明文显示，所以需要将Token保存到一个可靠的地方，避免后续忘记，如果忘记只能重新生成一个。

另外Token一般有有效期，默认最长的90天，也可以定制，但是GitHub不建议。所以Token到期后就会鉴权失败，出现类似以下的错误

```shell
lenovo@lenovo-ThinkStation-K:~/github_upload/android-doc$ git push
gh auth git-credential: "erase" operation not supported
remote: Invalid username or password.
fatal: 'https://github.com/applematrix/android-doc.git/' 鉴权失败
```

此时再次重新生成一个Token即可。

# 本地鉴权使用Token

执行**gh auth login**命令，在第三步：

How would you like to authenticate GitHub CLI

选择粘贴Token的方式，将前面生成的token贴上即可

```shell
lenovo@lenovo-ThinkStation-K:~/github_upload$ gh auth login
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations? HTTPS
? How would you like to authenticate GitHub CLI? Paste an authentication token
Tip: you can generate a Personal Access Token here https://github.com/settings/tokens
The minimum required scopes are 'repo', 'read:org', 'workflow'.
? Paste your authentication token: ****************************************
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as applematrix
```

