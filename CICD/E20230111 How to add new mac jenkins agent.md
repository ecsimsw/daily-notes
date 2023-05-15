# How to add new mac jenkins agent

회사에 남는 맥을 빌드 노드로 사용할 수 있도록 

## Install JDK

#### 1. In intel mac 
```
brew update
brew tap adoptopenjdk/openjdk
brew install --cask adoptopenjdk11
```

#### 2. In apple silicon mac 

Apple Silicon mac을 사용하는데 간헐적으로 연결이 종료된다며 아래를 따라본다. 

```
If you installed jdk with brew/adoptopenjdk in apple silicon mac, the agent will be intermittently turned off.
In the "normal" SSH connection, this should return 0,

sysctl sysctl.proc_translated

when executed as println "sysctl sysctl.proc_translated".execute().text in Jenkins, you will see a 1
```


#### Tip) Unintall jdk on mac

```
sudo rm -fr /Library/Java/JavaVirtualMachines/jdk-{version}.jdk/
sudo rm -fr /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin
sudo rm -fr /Library/PreferencePanes/JavaControlPanel.prefPane
```

```
Install native jdk

brew install --cask zulu
```


### Jenkins cofiguration → Add new node

![image](https://user-images.githubusercontent.com/46060746/211698603-bd4210c2-32d0-4044-9aa4-666458a75304.png)

Run from agent command line:

1. agent.jar 다운
2. nohup java -jar agent.jar -secret <SECRET> -workDir <WORK_DIR> &


### TIP) Disable mac sleep mode

#### disable
```
sudo pmset -c disablesleep 
```
  
#### able
```
sudo pmset -c disablesleep 0
```
  
