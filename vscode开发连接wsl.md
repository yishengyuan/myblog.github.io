
连接wsl  CTRL+SHIFT+P 调出命令面板，键入 WSL
![[Pasted image 20251031074439.png]]

选择文件夹
![[Pasted image 20251031074423.png]]
##  访问windows C和D  
```
cd /mnt/d
```

![[Pasted image 20251031075141.png]]

## 复制文件夹到目录文件夹(复制之前先压缩传输更快)

```
cp -r /mnt/d/hardhat-test.zip  .
```


## 切换到root 
``` 
sudo su - root
```

## 打开文件夹
![[Pasted image 20251031080344.png]]


安装yarn环境 和node环境 参考ubuntu命令

