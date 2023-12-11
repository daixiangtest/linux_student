#!/bin/bash
#安装最新稳定版 1Panel
#检查系统架构
osCheck=$(uname -a)
if [[ $osCheck =~ 'x86_64' ]]; then
    architecture="amd64"
elif [[ $osCheck =~ 'arm64' ]] || [[ $osCheck =~ 'aarch64' ]]; then
    architecture="arm64"
elif [[ $osCheck =~ 'armv7l' ]]; then
    architecture="armv7"
elif [[ $osCheck =~ 'ppc64le' ]]; then
    architecture="ppc64le"
elif [[ $osCheck =~ 's390x' ]]; then
    architecture="s390x"
else
    #不支持的架构，打印消息并退出
    echo "暂不支持的系统架构，请参阅官方文档，选择受支持的系统。"
    exit 1
fi

#如果未指定，默认将安装模式设置为 "stable"
if [[ ! ${INSTALL_MODE} ]]; then
    INSTALL_MODE="stable"
else
    # 验证安装模式
​    if [[ ${INSTALL_MODE} != "dev" && ${INSTALL_MODE} != "stable" ]]; then
        # 无效的安装模式，打印消息并退出
​        echo "请输入正确的安装模式（dev 或 stable）"
​        exit 1
​    fi
fi

#获取最新版本
VERSION=$(curl -s https://resource.fit2cloud.com/1panel/package/${INSTALL_MODE}/latest)
HASH_FILE_URL="https://resource.fit2cloud.com/1panel/package/${INSTALL_MODE}/${VERSION}/release/checksums.txt"

#检查是否成功获取版本
if [[ "x${VERSION}" == "x" ]]; then
    # 获取最新版本失败，打印消息并退出
    echo "获取最新版本失败，请稍候重试"
    exit 1
fi

#定义安装包细节
package_file_name="1panel-${VERSION}-linux-${architecture}.tar.gz"
package_download_url="https://resource.fit2cloud.com/1panel/package/${INSTALL_MODE}/${VERSION}/release/${package_file_name}"
expected_hash=$(curl -s "$HASH_FILE_URL" | grep "$package_file_name" | awk '{print $1}')

#检查本地是否已存在相同版本的安装包
if [ -f ${package_file_name} ]; then
    #如果哈希值匹配，跳过下载并继续安装
    actual_hash=$(sha256sum "$package_file_name" | awk '{print $1}')
    if [[ "$expected_hash" == "$actual_hash" ]]; then
        #安装包存在且哈希值匹配，跳过下载并安装
        echo "安装包已存在，跳过下载"
        rm -rf 1panel-${VERSION}-linux-${architecture}
        tar zxvf ${package_file_name}
        cd 1panel-${VERSION}-linux-${architecture}
        /bin/bash install.sh
        exit 0
    else
        #哈希值不匹配，重新下载安装包
        echo "已存在安装包，但是哈希值不一致，开始重新下载"
        rm -f ${package_file_name}
    fi
fi

#如果安装包不存在或哈希值不匹配，下载安装包
echo "开始下载 1Panel ${VERSION} 版本在线安装包"
echo "安装包下载地址： ${package_download_url}"

#下载安装包并记录安装日志
curl -LOk -o ${package_file_name} ${package_download_url}
curl -sfL https://resource.fit2cloud.com/installation-log.sh | sh -s 1p install ${VERSION}

#检查下载是否成功
if [ ! -f ${package_file_name} ]; then
    #下载安装包失败，打印消息并退出
    echo "下载安装包失败，请稍候重试。"
    exit 1
fi

#解压下载的安装包
tar zxvf ${package_file_name}

#检查解压是否成功
if [ $? != 0 ]; then
    # 解压安装包失败，打印消息并退出
    echo "下载安装包失败，请稍候重试。"
    rm -f ${package_file_name}
    exit 1
fi

#进入解压后的目录并运行安装脚本
cd 1panel-${VERSION}-linux-${architecture}
/bin/bash install.sh