# 分布式爬虫——QQ音乐 #

## 方案设计： ##

- 遍历26个字母分类和1个特殊符号#的歌手列表
- 遍历每个字母分类的歌手总页数
- 遍历每个字母分类的每页歌手信息
- 遍历每个歌手的歌曲页数

## 模块划分 ##

- 歌曲下载
- 歌手信息和歌曲信息
- 字母分类下的歌手列表
- 全站歌手列表

## 数据存储 ##

- 数据库选择mysql，使用sqlalchemy模块实现数据入库
- 入库数据有：
	- 主键：song_id
	- 歌名：song_name
	- 所属专辑：song_album
	- 时长：song_interval
	- 歌曲mid：song_songmid
	- 歌手姓名：song_singer
- 为了避免mysql数据库连接数不足，可先更改mysql默认的最大连接数：
	- 进入mysql
	- 执行sql语句：

			# 查看最大连接数
			show global variables like "max_connections";
			# 修改最大连接数
			set global variables max_connections=200;
  
## 分布式策略 ##

- 使用concurrent.futures模块
- 多进程：
	- 爬取全站歌曲信息是按照A~Z和符号#依次爬取，用循环分27个进程一一对应分类进行爬取
- 多线程
	- 由于爬虫大部分代码是IO密集型，在每个进程下可用多线程提高每个进程的运行效率
	- 在循环每个分类歌手总页数时，将不同页数的歌手交给不同线程去处理即可，并使每个线程处理的页数尽量相近。
- 分布式时注意事项：
	- 全局变量不能放在if __name__ == '__main__'中，因为使用多进程时新开的进程无法在该block内读取数据
	- 使用SQLalchemy入库最好重新创建一个数据库连接，避免多个线程、进程使用同一个连接而出现错误
