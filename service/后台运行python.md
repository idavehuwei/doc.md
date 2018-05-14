#python程序在linux后台运行

由于在shell脚本中直接使用&符号和python程序有冲突,不能正常的进入后台执行python(多番尝试写在python xxx.py &,后面的&总是不能正常识别.系统centos7).所以需要用到几种不同的方式来启动python

- nohup

```
nohup python run_server.py 
```

- ()

```
(python run_server.py)
```

- screen 推荐的方式

```
screen python run_server.py
```


脚本中使用

```
screen -dmS "server1"
screen -x -S "jms" -p 0 -X stuff "python /opt/server1/run_server.py all\n"

screen_name=$"server2"     
screen -dmS $screen_name  
cmd=$"python /opt/server2/run_server.py";
screen -x -S $screen_name -p 0 -X stuff "$cmd"
screen -x -S $screen_name -p 0 -X stuff $'\n'  

```
```
#!/bin/bash
#  echo "/bin/bash /opt/startjms.sh" and chmod +x /etc/rc.d/rc.local 

# env 
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
PATH=$PATH:$HOME/bin
export PATH
source /opt/py3/bin/activate


#open screen terminal
screen_name1="server1"
screen -dmS $screen_name1

screen_name2="server2"
screen -dmS $screen_name2

#trans cmd to screen terminal
screen -x -S "jms" -p 0 -X stuff "python /opt/$screen_name1/run_server.py all\n"

cmd="python /opt/$screen_name2/run_server.py"
screen -x -S $screen_name2 -p 0 -X stuff "$cmd"
screen -x -S $screen_name2 -p 0 -X stuff $'\n'
```




jms startscript

```
#!/bin/bash
#  echo "source /opt/py3/bin/activate" /root/.bash_profile
#  echo "/bin/bash /opt/startjms.sh" and chmod +x /etc/rc.d/rc.local 

# env 
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
PATH=$PATH:$HOME/bin
export PATH
source /opt/py3/bin/activate


screen -dmS "jms"
screen -dmS "coco"

sleep 3

screen -x -S "jms" -p 0 -X stuff "python /opt/jumpserver/run_server.py all\n"

sleep 3

screen -x -S "coco" -p 0 -X stuff "python /opt/coco/run_server.py all\n"
```