# function
## `rm`? just wait...
```
function rm(){
    array=("${(@s/ /)@}")
    for i in ${array[@]}; do

        mv $i ~/.Trash/${i##*/}"-"`date +"%Y_%m_%d-%H_%M_%S"`

    done
}
```

## 解压任意压缩文件
```
function x {
 if [ -z "$1" ]; then
    # display usage if no parameters given
    echo "Usage: extract <path/file_name>.<zip|rar|bz2|gz|tar|tbz2|tgz|Z|7z|xz|ex|tar.bz2|tar.gz|tar.xz>"
    echo "       extract <path/file_name_1.ext> [path/file_name_2.ext] [path/file_name_3.ext]"
 else
    for n in "$@"
    do
      if [ -f "$n" ] ; then
          case "${n%,}" in
            *.cbt|*.tar.bz2|*.tar.gz|*.tar.xz|*.tbz2|*.tgz|*.txz|*.tar) 
                         tar xvf "$n"       ;;
            *.lzma)      unlzma ./"$n"      ;;
            *.bz2)       bunzip2 ./"$n"     ;;
            *.cbr|*.rar)       unrar x -ad ./"$n" ;;
            *.gz)        gunzip ./"$n"      ;;
            *.cbz|*.epub|*.zip)       unzip ./"$n"       ;;
            *.z)         uncompress ./"$n"  ;;
            *.7z|*.arj|*.cab|*.cb7|*.chm|*.deb|*.dmg|*.iso|*.lzh|*.msi|*.pkg|*.rpm|*.udf|*.wim|*.xar)
                         7z x ./"$n"        ;;
            *.xz)        unxz ./"$n"        ;;
            *.exe)       cabextract ./"$n"  ;;
            *.cpio)      cpio -id < ./"$n"  ;;
            *.cba|*.ace)      unace x ./"$n"      ;;
            *)
                         echo "extract: '$n' - unknown archive method"
                         return 1
                         ;;
          esac
      else
          echo "'$n' - file does not exist"
          return 1
      fi
    done
fi
}
```

## 修改自定义 ss 规则
`mac only`

需要安装 `ShadowsocksX-NG-R8`

```
function addssrule(){
    
    filename=~/.ShadowsocksX-NG/user-rule.txt
    old_md5=$(cat $filename | md5)
    vim $filename

    new_md5=$(cat $filename | md5)

    if [[ $old_md5 != $new_md5 ]]; then
        app_name='ShadowsocksX-NG-R8'
        echo '[!] User rules was \033[33mchanged\033[0m, restart '$app_name' ...'

        echo -n '  [-] Killing '$app_name': '
        pkill ShadowsocksX-NG
        result=$(ps aux | grep -v grep | grep ShadowsocksX-NG-R8)

        if [ $result ]; then
            echo '\033[31mfailed\033[0m'
        else
            echo '\033[32msuccess\033[0m'
            sleep 0.5
            echo -n '  [-] Starting '$app_name': '
            open /Applications/ShadowsocksX-NG-R8.app
            sleep 1
            result=$(ps aux | grep -v grep | grep ShadowsocksX-NG-R8)
            if [ $result ]; then
                echo '\033[32msuccess\033[0m'
            else
                echo '\033[31mfailed\033[0m'
            fi
        fi

    else
        echo '[*] User rules was \033[32munchanged\033[0m'
    fi

    echo '[*] Bye!'
}
```

## 带颜色的 log

```

function plog(){
    if [ $# -ne 3 ]; then
        plog ERROR red "Usage: plog LEVEL COLOR MESSAGE"
        return 1
    fi
    level=[$1]
    color=$2
    message=$3
    timestamp=[$(date "+%Y-%m-%d %H:%M:%S")]
    case ${color} in
        black|k|黑|30)
            level="\033[30m${level}\033[0m";;
        red|r|红|31)
            level="\033[31m${level}\033[0m";;
        green|g|绿|32)
            level="\033[32m${level}\033[0m";;
        yellow|y|黄|33)
            level="\033[33m${level}\033[0m";;
        blue|b|蓝|34)
            level="\033[34m${level}\033[0m";;
        purple|p|紫|35)
            level="\033[35m${level}\033[0m";;
        cyan|c|青|36)
            level="\033[36m${level}\033[0m";;
        white|w|白|37)
            level="\033[37m${level}\033[0m";;
        *)
            plog ERROR red "Unknown param color ${color}"
            return 1;;
    esac
    echo ${timestamp} ${level} ${message}
}
```

# alias
## 终端走 http 代理，配合 ss 客户端使用
`alias proxy="http_proxy=http://127.0.0.1:7890/ https_proxy=http://127.0.0.1:7890/ "`

## clear terminal
`mac only`

`alias c="printf '\e]50;ClearScrollback\a'"`

