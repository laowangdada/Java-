#### 模块化体系

![Image](..\static\NGINX模块化体系.png)

core分两大部分 http和mail 为其他模块提供环境

event module：处理事件 epoll事件机制

phase handle：处理客户端请求

output filter：过滤（处理）内容

upstream：反向代理模块，转发请求

load balance：负载均衡模块

extend module：第三方模块

在nginx安装包中可以查看

![Image](..\static\安装模块.png)

auto：自动检测操作系统和编译脚本

changes：历史版本号和说明 

conf：配置，安装后将此目录中文件拷贝到conf文件夹中作为配置

configure：用作编译的配置

contrib：与工具包相关，提供一些工具

html：提供一些静态界面

LICENSE：版本说明

man：帮助手则

objs：安装的第三方模块之后会在次出现

src：nginx源码

![Image](..\static\nginxSrc目录.png)