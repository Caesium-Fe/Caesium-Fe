OTA Daily Report Tool

Report.py 的 4243行开始整理出summary数据





在使用cx_oracle组件时，需要环境中有instancetclient包中所有的.dll文件，将这些dll文件放在python.exe同级目录下就可以了。





字典在进行copy时，只是深复制字典本身，但是字典中的对象任然是引用，所以如果字典中出现新对象，则需要使用 new_dict = copy.deepcopy(old_dict)
