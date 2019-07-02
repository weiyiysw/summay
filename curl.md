# curl

~~~shell
# $url 替换为真实URL即可。

# 当我们不加任何选项时，默认发送GET请求获取链接内容
curl $url

# I只显示HTTP头， -i会显示HTTP头以及内容
curl -I $url

#  -v 选项，--verbose，指定该选项后，可以跟踪URL的连接信息。我们可以根据这个选项看看curl是怎么工作的。
# -i 是 -v的子集
curl -v $url

# 链接保存到文件
# 1. 使用Linux系统的 > 重定向符号
curl $url > index.html
# 2. 使用curl自带的 -o/-O 选项
# -o（小写）: 结果会保存到命令行提供的文件名
curl -o index.html $url
# -O（大写）：结果会以URL中的文件名作为保存输出的文件名
curl -O $url
# 3. 可以使用 -o/-O 选项同时指定多个链接

# 使用-L跟随链接重定向，有时候我们会遇到链接重定向了，这时，添加-L参数就会像浏览器一样，自带跳转
curl -L $url

# -A 自定义User-Agent，可以伪装成不同的设备或浏览器进行访问
# $uer-Agent，注意，这个需要使用双引号
curl -A "$user-Agent" $url

# -H 自定义Header，当访问一个URL需要传递特定header时使用，可直接传递cookie
curl -H "$header" $url

# -c 保存cookie，使用curl访问页面时，默认不保存cookie。
# 当我们需要保存cookie时，可使用此功能
# -c 后面加上要保存的文件名
curl -c "$filename" $url

# -b 读取cookie
# -c 把cookie保存到文件里，当我们再次使用时，需要读取出来
curl -b "$filename" $url

# -d 发送POST请求
# 我们经常需要登录页面，此时通常都是POST请求。
# -d 用于指定发送的数据， -X 用于指定发送数据的方式，如果省略-X，默认为POST方式
curl -d "username=tom&passwd=123456" -X POST $url

# 也可以发送数据时，强制使用GET方法
curl -d "data" -X GET $url

#
~~~

