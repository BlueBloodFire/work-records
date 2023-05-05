本人初入职场，经验尚浅，故借此记录来写下本人在技术工作上的点点滴滴，希望这能帮到自己。

2023年
4月3号-5月5号
1.了解Umi-OCR项目，查看底层代码，熟悉其实现流程
[项目链接](https://github.com/hiroi-sora/Umi-OCR)
2.实现该项目对文件夹的递归读取//win_main.py第976行
```
isRecursiveSearch = Config.get("isRecursiveSearch")
        for path in paths:  # 遍历拖入的所有路径
            if os.path.isdir(path):  # 若是目录
                if isRecursiveSearch:  # 需要递归子文件夹
                    for subDir, dirs, subFiles in os.walk(path):
                        for s in subFiles:
                            addImage(subDir+"\\"+s)
                else:  # 非递归，只搜索子文件夹一层
                    subFiles = os.listdir(path)  # 遍历子文件
                    for s in subFiles:
                        addImage(path+"\\"+s)  # 添加
            elif os.path.isfile(path):  # 若是文件：
                addImage(path)  # 直接添加
```
3.实现该项目对图片输出成java后辍文件//ocr/output_java.py,ocr/output_separate_java.py
```
其输出模块与原有的output_txt.py,output_separate_txt.py一致
```
```
合并java输出
    def __init__(self):
        outputDir = Config.get('outputFilePath')  # 输出路径（文件夹）
        outputDirDir = os.path.dirname(outputDir)
        outputName = Config.get('outputFileName')  # 文件名
        self.outputPath = f'{outputDirDir}/{outputName}.java'  # 输出路径
        self.isDebug = Config.get('isDebug')  # 是否输出调试
        # 创建输出文件
        try:
            if os.path.exists(self.outputPath):  # 文件存在
                os.remove(self.outputPath)  # 删除文件
            open(self.outputPath, 'w').close()  # 创建文件
        except FileNotFoundError:
            raise Exception(f'创建java文件失败。请检查以下地址是否正确。\n{self.outputPath}')
        except Exception as e:
            raise Exception(
                f'创建java文件失败。文件地址：\n{self.outputPath}\n\n错误信息：\n{e}')
    
        def img(self, textBlockList, imgInfo, numData, textDebug):
        # 输出图片结果
        # 标题和debug信息
        textDebug = f'```\n{textDebug}```\n' if self.isDebug and textDebug else ''
        textOut = f"\n{textDebug}"
        # 正文
        for tb in textBlockList:
            if tb['text']:
                textOut += f"{tb['text']}\n"
        self.print(textOut+'\n')
```
```
独立java输出
    def img(self, textBlockList, imgInfo, numData, textDebug):
        '''输出图片结果'''
        # 收集ocr文字
        ocrText = ''
        for tb in textBlockList:
            ocrText += tb['text'] + '\n'
        # 输出到图片同名java文件
        path = os.path.splitext(imgInfo['path'])[0]+'.java'
        try:
            with open(path, 'w', encoding='utf-8') as f:  # 写入本地文件
                f.write(ocrText)
        except FileNotFoundError:
            raise Exception(f'创建java文件失败。请检查以下地址是否正确。\n{path}')
        except Exception as e:
            raise Exception(
                f'创建java文件失败。文件地址：\n{path}\n\n错误信息：\n{e}')
```
4.修改//config.py
```
添加output_java、output_separate_java的输出判断
'isOutputJava': {
        'default': True,
        'isSave': True,
        'isTK': True,
    },
    'isOutputSeparateJava': {
        'default': False,
        'isSave': True,
        'isTK': True,
    },
```
5.添加java后辍文件的输出可视化模块//win_main.py
```
wid = ttk.Checkbutton(
                    fr1, variable=Config.getTK('isOutputJava'), text='合并.java文件')
                self.balloon.bind(wid, f'所有识别文本输出到同一个java文件')
                wid.grid(column=0, row=1, sticky='w')
                self.lockWidget.append(wid)
wid = ttk.Checkbutton(
                    fr1, variable=Config.getTK('isOutputSeparateJava'), text='独立.java文件')
                self.balloon.bind(wid, f'每张图片的文本输出到同名的单独java文件')
                wid.grid(column=2, row=1, sticky='w')
                self.lockWidget.append(wid)
```
6.添加输出java后辍文件模块的路径//msn_batch_paths.py
```
from ocr.output_java import OutputJava
from ocr.output_separate_java import OutputSeparateJava

        if Config.get("isOutputSeparateJava"): #输出到单独java
            self.outputList.append(OutputSeparateJava())
        if Config.get("isOutputJava"): #输出到java
            self.outputList.append((OutputJava()))
```
7.最终效果
<center class="half">
<img src=软件界面.jpg width=330/>
<img src=软件设置.png width=330/>
</center>

8.添加命令行输出//umiocr.ahk
```
pipe_name := "\\.\pipe\umiocr" ; 命名管道名称
run_name := "Umi-OCR 文字识别.exe" ; 启动程序名称

; 将启动参数数组 转为字符串，每组参数用双引号括起来。
args := ""
for index, value in A_Args
{
    args .= """" . value . """ "
}

; 检查命名管道，若存在则通过管道传指令
if FileExist(pipe_name)
{
    Sleep, 30 ; 等待一段时间让服务端重启管道
    FileEncoding UTF-8 ; 设置文件写入的编码类型为UTF8
    pipe := FileOpen(pipe_name, "w")
    if (ErrorLevel) {
        MsgBox, 16, Error, 打开命名管道%pipe_name%失败。
        return
    }
    pipe.Write(args) ; 向管道写入指令
}
; 若不存在则启动Umi-OCR软件，通过启动参数传指令
else
{
    if !FileExist(run_name)
    {
        MsgBox 程序%run_name%不存在。请将该命令行入口放在Umi-OCR文件夹下。
        return
    }
    Run, %run_name% %args%
}
```
9.至此，该项目的工作需求已完成


