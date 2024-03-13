OTA Daily Report Tool

Report.py 的 4243行开始整理出summary数据





在使用cx_oracle组件时，需要环境中有instancetclient包中所有的.dll文件，将这些dll文件放在python.exe同级目录下就可以了。





字典在进行copy时，只是深复制字典本身，但是字典中的对象任然是引用，所以如果字典中出现新对象，则需要使用 new_dict = copy.deepcopy(old_dict)



# 重构

## UI

```

_is_vswr 判断是否为vswr， 走特殊处理逻辑，报告名称，  对应Data source

_delete_retest 判断是否删除重测数据 对应Keep Re-test Data
_if_fa 判断是否为fa 对应Keep Re-test Data保留機台第一遍測試數據後12H（默認，可設置）內的數據, 超過的數據也不需要在報告頁面中出現

_t_distance 中间时间跨度时间 对应Keep FA time

_isNA， _isNA_ether， _isNA_sota， _isNA_mars， _isNA_aoa 判断是否删除NA limit数据

_data_type 为 por还是doe数据 对应 Data Type

is_apple_pass 判断是否需要apple pass 对应 Apple Pass

_auto_select_item 需要打开一份excel，给用户填写，用于不走keyword逻辑， 逻辑在 _get_item_data 中

_if_consum 判断是否需要config summary， 对应Config-Summary

_if_del_format 是否需要删除删除数据样式， 对应Remove data style

_top10 用于summary页展示失败和重测数据个数 对应Fail item amount

_if_slim 用来处理数据时走特殊处理（dataprocess， _process_keyword_info）对应 Slimming data

_sota_pass_fail_mode 各工站算pass和fail的依照（items还是status） _if_items在选择items为True别的为False        _empty_Fail默认为False
_mars_pass_fail_mode
_other_pass_fail_mode

_omnia_pass_fail_mode 当ota数据分工站时（工站数不为1），_if_items在选择items为True别的为False        _empty_Fail：在_if_items为True时需要根据_omnia_empty_value选择PASS为False，否则为True，在_if_items为False时，为False
_omnia_empty_value 当ota数据不分工站时（工站个数为1），_if_items默认为True   _empty_Fail根据选择PASS为False，否则为True

当_delete_retest为True时，_if_items若为True，则删除重复SN数据并删除item下数据全为空， 若为False，则删除item下数据全为空

_same_sn 对重复SN有关（从多个csv中找出共有的SN）， 对应 Only Report Same SN
```

## Keyword

- ItemSelection

  ```
  ItemSelection  根据Map sheet中得到real station名称，对应station中的
  检索：select_item.py的get_select_sheet()
  ```

- Map

  ```
  get_map_station方法里用于根据csv中的station名称来获取特定station名称，如果没有map这个sheet，则直接用csv中的station名称
  read_station_map方法也有使用
  ```

- SelectConfig

  ```
  get_select_config_data方法读取 _select_user_care_configs方法中需要保留的config
  ```

- DeleteConfig

  ```
  get_select_config_data方法读取 _select_user_care_configs方法中需要移除的config
  ```

- station order

  

- splitData

  ```
  _get_split_data_keyword方法里获取split data规则，存到_split_data_keyword(字典)
  ```

- Delete SN

  ```
  _get_delete_sn_keyword方法中获取一列SN数据，用来移除SN行数据
  ```

- Limits Keyword

  ```
  _get_limits_keyword方法中获取keyword中Limits Keyword的数据，存到_limits_keyeord(列表)， 用到change_limits方法中
  ```

- Omnia Summary SKU mapping

  ```
  分离出原始SN的product 与version和sheet中填写一致的数据，成一个sheet
  ```

- Omnia Split staton

  ```
  同splitData逻辑
  ```

- REDSIG-OTA-LAT

  

- combine csv

  

- AntGroup

  

- Limits Keyword

  ```
  change_limits方法里用到这个sheet，去修改特定item的limit
  ```

- Report name

  ```
  Report name  报告名称  检索：get_report_name
  ```

  

## sn文件

```
_is_sn bool值，取决于是否有sn文件

_get_all_sn_config_unitno方法：根据SN.xlxs 获取sn,config, unit


```

## Auto Select

```
item_excel_path  item select文件的路径，用于_get_item_data方法
```



对DailyReport进行需求整理，做好记录

（整理keyword、ui、sn.xlsx、bandsequence.xlsx影响范围，整理报表数据的生成逻辑、标色逻辑）

## bandsequence.xlsx

```
1. 读取文档
bandsequence.py， xlsx三个sheet:tx,rx,bandgroup
2. 主要操作
	2.1 tx sheet读取band的这一列，保存在变量self.band_tx中，band_tx_dict是self.band_tx值作为key，index作为value的dict。rx sheet相同
	2.2 没有传bandsequence.xlsx的路径时，有一个默认的band_tx，在defalut_keyword()中
	2.3 bandgroup sheet保存在self.bandorder、self.band_dict里，bandorder是第一行的band值，band_dict = {'tx':{band类型1800-2000M:[一列的band值B17]}}
	2.4 main_band_label()中，把tx和rx的每个band对应的band type对应保存在self.band_label_dict中
3. 影响范围
sort_item() 方法中tx， rx排序会用到	ReportData.py by_band()
```





## SUMMARY页

```
set_summary_data方法 获取summary数据的方法
```

## CONFIG SUMMARY页

```

```

## OMNIA SUMMARY页

```

```



```
_get_non_aab_fail逻辑：拿删除重复sn数据后的数据与原始数据进行匹配，找到符合fail状态的相同sn，并且重复的sn对应的station—id值具有少于2个的，或者重复sn的个数少于3个的，这个sn会被保存下来。用于后续写入表格样式使用......
```









```
整一个重构逻辑：
1.UI端，数据后端
2.数据后端分为处理部分，写入部分
3.处理部分需要做的：处理数据再返回。（我分为了多层， 调用顺序为：api层调用controller层，controller层调用service层处理，service
层取dao层数据）
```





需要一个得到所有cof list的方法，cof item用 cof_ 开头还是 c_ 





```
fail逻辑
要用failed_unit_list： 所有fail了的sn坐标
判断是否有apple pass列
是：

否：
	判断是否为pdca数据
	是：
		循环遍历failed_unit_list
		   取出每个sn的failing test list值
            判断值的数量
            没有：
                pass
            一个：
                是否按照 if_item
                是：
                    这个fail值是否在所有数据的item中
                    是：
                        拿出这个item的limit2值, sn中属于这个item的值， sn的所有item值
                        判断 isfail 并且 是否apple——pass
                        是：
                        	判断是否这个sn的状态是否为fail
                        	是：
                        		判断是否 empty_fail 并且 所有item值中有空值
                        		是：
                        			dict[fail—item] += 1
                        		否：
                        			判断 check——empty——applepass
                        			是：
                        				applepass
                        			否：
                        				dict[fail—item] += 1
                     	否：
                     		dict[fail—item] += 1
				否：
					判断是否 apple_pass 并且 cof+fail_item 是否在 cof_data中
					是：
						判断check——empty——applepass
						是：
							applepass
						否：
							dict[fail—item] += 1
					否：
						dict[fail—item] += 1
			多个：
				判断是否 if——item：
				是：
					遍历所有failing test items
						拿出所有sn的item值
						判断 不是 apple pass 或者 （ empty_fail 并且 所有item值中有空）
						是：
							dict[fail—item] += 1
						若是 fail item在 所有item中：
							判断是 isfail 或 （empty 并且 所有item值中有空） 或 不是 check empty applepass
							是：
								dict[fail—item] += 1
						否则：
							apple
	否：


```





