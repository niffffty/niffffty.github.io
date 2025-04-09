报错：使用gentime导出svg格式文件时编码报错
原因：文件中有中文，默认为utf-8编码，应该为GB2312
解决：用vscode打开svg文件，修改编码为GB2312