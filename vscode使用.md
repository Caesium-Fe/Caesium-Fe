<h1>VSCode使用指南</h1>

## 1、如何使用python？

​	安装python插件

## 2、如何进行Debug？

​	首先确定python插件的版本能够支持当前python版本进行debug， 其次需要配置 launch.json 文件(这个文件在点击运行调试按钮后会需要你点击配置)

"configurations"中添加多个字典代表着添加多个测试实例，可以并行debug

## 3、如何使用调试控制台？

​	同pycharm的evaluate一致使用，换行需要使用 shift+Enter。

## 4、运行时找不到自定义的模块？

```
1.首先我们先用 sys.path 来查看当前运行环境中是否存在项目路径；
2.如果有查看是否名字有误，如果没有则需要添加相应的路径；
3. 一、可以直接添加当前项目路径：sys.path.append('项目路径')；
	二、或者可以添加相对路径：sys.path.append(os.path.join('项目上一层路径', '项目名称'))
	  三、需要在项目的.vscode中新建setting.json文件，并在文件中写入：{"python.analysis.extraPaths":["模块所在的文件夹名称"]}
4.如果嫌第三点的做法太过于啰嗦，可以选择在.vscode文件夹下的launch.json文件中的 configrations 中新增: "env": {"PYTHONPATH": "${workspaceRoot}"},
并在setting.json文件中新增: "terminal.integrated.env.windows":{"PYTHONPATH":"${workspaceFolder};${env:PYTHONPATH}"}
5.最终按 F5进行继续调试，如果还报找不到模块就重启一遍vscode。
```

## 5、
