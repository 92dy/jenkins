## Necessary plug-ins

### 默认的参数化构建

-  字符参数

![](./images/char_parameter.png)

- 文本参数

![](./images/txt_parameter.png)

### Extended Choice Parameter-`requires manual installation`

![](./images/extend_parameter.png)
![](./images/extend_parameter_res.png)
-------------------------------------------------------------------------------

![](./images/property_parameter.png)

![](./images/property_parameter_file.png)

![](./images/property_parameter_file_res.png)

### Git Parameter-`requires manual installation`

- **1、Add Global User 凭据**

![](./images/gitlab_user_root.png)

> 说明这里添加 `gitlab` 账号的用户名和密码,不是`gitlab`服务器的用户名和密码

- **2、Jenkins 使用root 用户运行** 

```
sed -i 's@JENKINS_USER="jenkins"@JENKINS_USER="root"@g /etc/sysconfig/jenkins && /etc/init.d/jenkins restart 
```
- **3、参数化构建过程**

![](./images/git_parameter_1.png)
![](./images/git_parameter_2.png)
![](./images/git_parameter_res.png)

