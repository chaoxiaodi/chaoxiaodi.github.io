---
layout: post
title: "oracle cloud oci python sdk auth practice"
subtitle: 'oci 认证最佳实践(个人认为)'
author: "chaoxiaodi"
header-style: text
tags:
  - oracle cloud
  - oci
  - auth
  - python
  - credentials
---

# 前言

在接触oracle cloud之后

按照上篇aws认证的思路

继续输出oci在认证使用过程中更好的方式

oci的认证没有aws的iam一样提供那么多的功能

只能按照文档用个大概

以后oci进行大的升级的时候可能会来再次更新这篇文章


# 代码

简单的应用代码就不单独贴出来了

可以直接在三段代码里面提取出来直接使用

同样也是支持传递类似aksk的信息

支持通过配置文件获取权限

支持直接在instance绑定Principal

    #!/usr/bin/env python

    import oci

    class OciApi:

        def __init__(self, config: dict = None, profile_name: str = 'DEFAULT'):
            auth_flag = False

            try:
                if config:
                    oci.config.validate_config(config)
                    self.tenancy_id = config['tenancy']
                    auth_flag = True
                    self.config = config
                    self.client_kwargs = {}
            except Exception as e:
                self.log.warning(e)

            try:
                if not auth_flag:
                    default_config = oci.config.from_file(profile_name=profile_name)
                    oci.config.validate_config(default_config)
                    self.config = default_config
                    self.tenancy_id = default_config['tenancy']

                    auth_flag = True
                    self.client_kwargs = {}
            except Exception as e:
                self.log.warning(e)

            try:
                if not auth_flag:
                    custom_strategy = oci.retry.RetryStrategyBuilder().add_max_attempts(1).get_retry_strategy()
                    signer_kwargs = {'retry_strategy': custom_strategy}
                    signer = oci.auth.signers.InstancePrincipalsSecurityTokenSigner(timeout=1,**signer_kwargs)
                    self.client_kwargs = {
                        'signer': signer
                    }
                    self.tenancy_id = signer.tenancy_id

                    auth_flag = True
                    self.config = {}

            except Exception as e:
                self.log.warning(e)

            if not auth_flag:
                raise Exception('auth failed')


            self.identity = oci.identity.IdentityClient(self.config, **self.client_kwargs)



    if __name__ == '__main__':
        pass



# 参考
[OCI SDK Authentication Methods](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_authentication_methods.htm)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
