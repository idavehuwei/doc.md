

nginx


proxy_connect_timeout 5;
proxy_read_timeout 60;
proxy_send_timeout  5;
proxy_buffer_size 16k;
proxy_buffers  4 64k;
proxy_busy_buffers_size 128k;
proxy_temp_file_write_size 128k;
proxy_temp_path /home/cache/temp;
#临时文件目录
proxy_cache_path /home/cache/path levels=1:2 keys_zone=cache_one:5m inactive=7d max_size=1g;
#5m为内存占用，1g为最大硬盘占用，cache_one为缓存区名称，如果需要修改对应修改。

server{
    listen 80;
    server_name example.com www.example.com;
    #绑定的域名
    index index.php;
    #默认首页
    access_log off;
    #off 关闭日志
    
    location / {
        proxy_cache_key "$scheme://$host$request_uri";
        #缓存key规则，用于自动清除缓存。
        proxy_cache cache_one;
        #缓存区名称，与前面定义的相同
        proxy_cache_valid 200 304 3h;
        proxy_cache_valid 301 3d;
        proxy_cache_valid any 10s;
        #200 304状态缓存3小时
        301状态缓存3天
        其他状态缓存（如502 404）10秒
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #向后端传递访客ip
        proxy_set_header Referer http://example.com;
        #强制定义Referer，程序验证判断会用到
        proxy_set_header Host $host;
        #定义主机头
        proxy_pass http://1.2.3.4;
        #指定后端ip，可以加端口
        #proxy_cache_use_stale invalid_header error timeout http_502;
        #当后端出现错误、超时、502状态时启用过期缓存，慎用。
    }
}