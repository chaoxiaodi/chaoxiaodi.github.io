---
layout: post
title: "tencentcloud python sdk auth practice"
subtitle: 'tencentcloud 认证最佳实践(个人认为)'
author: "chaoxiaodi"
header-style: text
tags:
  - tencentcloud
  - boto3
  - auth
  - python
  - Credentials
---

# 前言

继续记录一篇关于云厂商sdk认证的文章

按照官方文档说明

腾讯云同样支持多种凭证管理

[腾讯云凭证管理文档](https://cloud.tencent.com/document/sdk/python?from=20423&from_column=20423#6d1c6674-37d1-431c-908b-2e27bec41331)

按照与前两篇逻辑尽量贴近统一的原则

还是按照之前的结构进行认证

不过文章中提到的环境变量是没有使用的

直接上代码



# 代码

    class QcloudApi:

        def __init__(self, account='',key=None, secret=None, profile='', role='relo_name'):
            cred = None
            self.secret_id = None
            self.secret_key = None

            if profile:
                cred = credential.ProfileCredential().get_credential()

            if not cred:
                cred = credential.CVMRoleCredential().get_credential()

            if key and secret:
                cred = credential.Credential(key, secret)

            if cred is None:
                raise Exception('必须提供profile或者secret密钥信息')

            self.cred = cred

            if account:
                self.__assume_role('ap-beijing', account, role)

        def __assume_role(self, region, account, role='role_name'):
            endpoint = 'sts.tencentcloudapi.com'
            try:
                client_profile = self.__generate_client_profile(endpoint)

                client = sts_client.StsClient(self.cred, region, client_profile)
                req = sts_models.AssumeRoleRequest()
                params = {
                    "RoleArn": "qcs::cam::uin/%s:roleName/%s" % (account, role),
                    "RoleSessionName": "to-%s" % account
                }
                req.from_json_string(json.dumps(params))

                # 返回的resp是一个AssumeRoleResponse的实例，与请求对象对应
                resp = client.AssumeRole(req)

                self.secret_id = resp.Credentials.TmpSecretId
                self.secret_key = resp.Credentials.TmpSecretKey
                token = resp.Credentials.Token
                self.cred = credential.Credential(self.secret_id, self.secret_key, token)
            except Exception as e:
                print(e)

## 调用方法

    q = QcloudApi()
    key = ''
    sec = ''
    # 下面是切换账号的一些方法 进行assume role的操作在有account的参数时执行
    # 前提是必须进行了授权 a账号有切换到b账号的权限 同时b账号允许a账号进行切换
    # 但是腾讯云的角色切换这部分功能还并不是很完善
    q = QcloudApi(key=key, secret=sec)
    q = QcloudApi(account='xxxxxx')
    q = QcloudApi(account='xxxxxx', role='test')
    q = QcloudApi(account='xxxxxx', key=key, secret=sec, role='test')

# 参考
[腾讯云凭证管理文档](https://cloud.tencent.com/document/sdk/python?from=20423&from_column=20423#6d1c6674-37d1-431c-908b-2e27bec41331)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
