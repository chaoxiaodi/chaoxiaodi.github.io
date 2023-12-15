---
layout: post
title: "alibabacloud python sdk auth practice"
subtitle: 'alibabacloud 认证最佳实践(个人认为)'
author: "chaoxiaodi"
header-style: text
tags:
  - alibabacloud
  - auth
  - python
  - Credentials
---

# 前言

继续记录一篇关于云厂商sdk认证的文章

按照官方文档说明

阿里云同样支持多种凭证管理

[阿里云凭证管理文档](https://help.aliyun.com/zh/sdk/developer-reference/manage-python-access-credentials?spm=a2c4g.11186623.0.0.363f3cefatNGgP)

按照与前两篇逻辑尽量贴近统一的原则

还是按照之前的结构进行认证

不过文章中提到的环境变量是没有使用的

直接上代码



# 代码

    class AlicloudApi:
    
        def __init__(self,
            access_key_id: str='',
            access_key_secret: str='',
            account: str='',
            profile: str='default',
            role_name: str='role_name'
        ):

            self.cred = None

            auth_util.client_type = profile

            try:
                cred = CredClient()
                if cred:
                    self.cred = cred
                log.info('this time use default credential')
            except CredentialException as ce:
                config = CredConfig(
                    type='ecs_ram_role'
                )
                cred = CredClient(config=config)
                if cred:
                    self.cred = cred
                log.info('this time use ecs ram role credential')
            except Exception as e:
                tb = traceback.format_exc()
                log.error('get default cerdential error %s\n traceback: %s' % (e, tb))

            try:
                if access_key_id and access_key_secret:
                    config = CredConfig(
                        type='access_key',
                        access_key_id=access_key_id,
                        access_key_secret=access_key_secret
                    )
                    cred = CredClient(config)
                    if cred:
                        self.cred = cred
            except Exception as e:
                tb = traceback.format_exc()
                log.error('get cerdential with ak error %s\n traceback: %s' % (e, tb))

            
            try:
                if account:
                    cred = self.__assume_role(account, role_name)

                    config = CredConfig(
                        type='sts',
                        access_key_id=cred.access_key_id,
                        access_key_secret=cred.access_key_secret,
                        security_token=cred.security_token
                    )
                    self.cred = CredClient(config)

            except Exception as e:
                tb = traceback.format_exc()
                log.error('assume role error %s\n traceback: %s' % (e, tb))

            print(self.cred, self.cred.get_type(), self.cred.get_access_key_id(), self.cred.get_access_key_secret(), self.cred.get_security_token())


        def __assume_role(self, account, role_name):
            endpoint = 'sts.cn-beijing.aliyuncs.com'
            config = self.__make_config(endpoint)
            client = Sts20150401Client(config)
            assume_role_request = sts_20150401_models.AssumeRoleRequest(
                role_arn='acs:ram::%s:role/%s' % (account, role_name),
                role_session_name='to-%s' % account
            )
            runtime = util_models.RuntimeOptions()
            cred = client.assume_role_with_options(assume_role_request, runtime)
            return cred.body.credentials

## 调用方法

    q = AlicloudApi()
    key = ''
    sec = ''
    # 下面是切换账号的一些方法 进行assume role的操作在有account的参数时执行
    # 前提是必须进行了授权 a账号有切换到b账号的权限 同时b账号允许a账号进行切换
    # 但是腾讯云的角色切换这部分功能还并不是很完善
    q = AlicloudApi(key=key, secret=sec)
    q = AlicloudApi(account='xxxxx')
    q = AlicloudApi(account='xxxxx', role='test')
    q = AlicloudApi(account='xxxxx', key=key, secret=sec, role='test')

# 参考
[阿里云凭证管理文档](https://help.aliyun.com/zh/sdk/developer-reference/manage-python-access-credentials?spm=a2c4g.11186623.0.0.363f3cefatNGgP)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
