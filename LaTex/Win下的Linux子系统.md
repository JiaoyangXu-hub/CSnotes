# win下的Linux子系统

打开powershell，键入
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```


后续可在https://zhuanlan.zhihu.com/p/258563812找见

使用体验不错，安装不附带GUN，但该方案是原生环境，因此运行比较流畅。但由于是采用虚拟环境，因此可能比装双系统性能稍低，且其存储空间固定于启动盘中，会增大启动盘的存储负担