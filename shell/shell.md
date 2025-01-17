<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [推荐实践](#%E6%8E%A8%E8%8D%90%E5%AE%9E%E8%B7%B5)
- [基础操作](#%E5%9F%BA%E7%A1%80%E6%93%8D%E4%BD%9C)
    - [jq，操作 json 文件](#jq%E6%93%8D%E4%BD%9C-json-%E6%96%87%E4%BB%B6)
    - [年月日](#%E5%B9%B4%E6%9C%88%E6%97%A5)
    - [添加多行到文件](#%E6%B7%BB%E5%8A%A0%E5%A4%9A%E8%A1%8C%E5%88%B0%E6%96%87%E4%BB%B6)
    - [多行赋值变量](#%E5%A4%9A%E8%A1%8C%E8%B5%8B%E5%80%BC%E5%8F%98%E9%87%8F)
    - [软连接](#%E8%BD%AF%E8%BF%9E%E6%8E%A5)
    - [当前目录](#%E5%BD%93%E5%89%8D%E7%9B%AE%E5%BD%95)
    - [获取脚本所在项目根目录](#%E8%8E%B7%E5%8F%96%E8%84%9A%E6%9C%AC%E6%89%80%E5%9C%A8%E9%A1%B9%E7%9B%AE%E6%A0%B9%E7%9B%AE%E5%BD%95)
- [远程执行命令](#%E8%BF%9C%E7%A8%8B%E6%89%A7%E8%A1%8C%E5%91%BD%E4%BB%A4)
- [有趣命令](#%E6%9C%89%E8%B6%A3%E5%91%BD%E4%BB%A4)
    - [判断程序是否安装](#%E5%88%A4%E6%96%AD%E7%A8%8B%E5%BA%8F%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85)
    - [使用管理员权限运行](#%E4%BD%BF%E7%94%A8%E7%AE%A1%E7%90%86%E5%91%98%E6%9D%83%E9%99%90%E8%BF%90%E8%A1%8C)
    - [提示输入值操作](#%E6%8F%90%E7%A4%BA%E8%BE%93%E5%85%A5%E5%80%BC%E6%93%8D%E4%BD%9C)
    - [判断数据是否包含](#%E5%88%A4%E6%96%AD%E6%95%B0%E6%8D%AE%E6%98%AF%E5%90%A6%E5%8C%85%E5%90%AB)
    - [遍历 json](#%E9%81%8D%E5%8E%86-json)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 推荐实践

```shell
#!/bin/bash
# -e 设置定义变量没有使用报错
# -o pipefail 可以让管道命令报错了，可以让脚本也退出
set -eo pipefail

# 复杂命令赋值并且 export,需要分隔开，可以让脚本感知复杂命令报错
AWS_ASSUME_ROLE=$(jq -er .assume_role < a.file")
export AWS_ASSUME_ROLE

# 或者 bash -eo pipefail script.sh
```

可以使用 shellcheck 检查脚本有没有不好的实践。



## 基础操作

#### jq，操作 json 文件

```shell
yum install epel-release -y
yum update -y
yum install jq -y
```

#### 年月日

```shell
# 2022-04-03-13-32-02
date +"%Y-%m-%d-%H-%M-%S"
# 获取当前日期   2020-07-18 14:35:06
NowTime=$(/bin/date +%Y-%m-%d' '%H:%M:%S)
echo ${NowTime}

# 返回时间 2020-08-05-10-28-29
file_time() {
    echo $(date +%Y-%m-%d-%H-%M-%S)
}

# 返回时间戳,1970 到现在
file_timestamp() {
    echo $(date +%s)
}
```

#### 添加多行到文件

```bash
#!/bin/bash
# 自定义文本 END OF FILE 
echo "abcd" >a.txt

# <<EOF 定义文件的结束符
cat >a.txt <<EOF
AA111
BB222
CC333
EOF
```

#### 多行赋值变量

```shell
## 添加单行或多行内容到文本

# <<EOF 定义文件的结束符
AA=$(cat <<EOF
11
22
EOF
)
# 保留换行添加双引号
echo "$AA"
json=$(
    cat <<-END
    {
    "tag_name": "11",
    "tag_name1": "11"
    }
END
)
echo $json
```

#### 软连接

```shell
# 创建硬链接
ln 源文件 目标文件

# 创建软连接
ln -s 源文件 目标文件
```

#### 获取执行的脚本所在目录

```shell
#!/bin/bash
set -eo pipefail
SCRIPT_FILE_DIR="$(dirname "$(readlink -f "${BASH_SOURCE:-$0}")")"
echo ${SCRIPT_FILE_DIR}
```



```shell
PROJECT_HOME_DIR=$(
  cd "${SCRIPT_FILE_DIR}/.."
  pwd
)
```





## 远程执行命令

```shell
# 双引号，必须有。如果不加双引号，第二个ls命令在本地执行
# 分号，两个命令之间用分号隔开
ssh local_centos7 'cd $HOME ; echo "111">aaa.txt'
```

多行复杂命令

```shell
#!/bin/bash
set -eo pipefail
ssh local_centos7 >/dev/null 2>&1 <<EOF
cd \$HOME
touch abcdefg1.txt
exit
EOF
echo done!
```

下载远程命令执行

```shell
ssh local_centos7 "curl -fsSL -O https://raw.githubusercontent.com/zhangpanqin/fly-circleci/HEAD/not_set_eo_pipefail.sh ; /bin/bash ./not_set_eo_pipefail.sh ; rm ./not_set_eo_pipefail.sh"
```

## 有趣命令

#### 判断程序是否安装

```shell
if [[ "$(command -v brew)" ]]; then
    echo "安装"
else
    echo "没散装"
fi
```

#### 使用管理员权限运行

```shell
CURRENT_USER=$(whoami)
if [[ $CURRENT_USER != "root" ]]; then
echo "请使用 sudo 运行"
exit 2
fi
```

#### 提示输入值操作

```shell
read -r -p "Do you want to install ? (Y/n) " yn
case $yn in
[nN])
  exit 0
  ;;
*)
  ;;
esac
```

#### 判断数据是否包含

```shell
SUPPORT_ENVIRONMENT=(
sandbox
prod
)

if echo "${SUPPORT_ENVIRONMENT[*]}" | grep -q -w "${ENVIRONMENT}"; then
echo "111"
else
exit 1
fi
```

#### 遍历 json

```shell
JSON_TEST=$(
cat <<EOF
{
  "items": [
    {
      "name": "xiaoming",
      "age": 1
    }
  ]
}
EOF
)

for item in $(jq -er '.items[]| tostring' <<<"${JSON_TEST}"); do
    name=$(jq -er '.name' <<<"${item}")
    age=$(jq -er '.age' <<<"${item}")
    echo "name is ${name}, age is ${age}"
done
```
