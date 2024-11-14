---
title: "一次calibre打开epub文件出现错误的记录"
date: 2024-11-14T15:02:29+08:00
draft: false
---

在使用calibre试图打开某些epub文件时，会显示如下的错误。
```
calibre, version 7.21.0
错误： 无法加载书籍: 无法打开位于 Calibre 书库\Ai Le Li _Kui En\Wang Zhe Yi Shi (23)\Wang Zhe Yi Shi - Ai Le Li _Kui En.epub 的书籍，点击“显示详情”获取更多信息。

Failed to convert book: Calibre 书库\Ai Le Li _Kui En\Wang Zhe Yi Shi (23)\Wang Zhe Yi Shi - Ai Le Li _Kui En.epub with error:
InputFormatPlugin: EPUB Input running
on C:\Users\xieyuhuai\Calibre 书库\Ai Le Li _Kui En\Wang Zhe Yi Shi (23)\Wang Zhe Yi Shi - Ai Le Li _Kui En.epub
Found HTML cover OPS/coverpage.html
Failed to run pipe worker with command: from calibre.srv.render_book import viewer_main; viewer_main()
Traceback (most recent call last):
  File "runpy.py", line 198, in _run_module_as_main
  File "runpy.py", line 88, in _run_code
  File "site.py", line 83, in <module>
  File "site.py", line 78, in main
  File "site.py", line 50, in run_entry_point
  File "calibre\utils\ipc\worker.py", line 196, in main
  File "<string>", line 1, in <module>
  File "calibre\srv\render_book.py", line 847, in viewer_main
  File "calibre\srv\render_book.py", line 838, in render_for_viewer
  File "calibre\srv\render_book.py", line 816, in render
  File "calibre\srv\render_book.py", line 618, in process_exploded_book
calibre.srv.render_book.Spineless: Book is empty, no content in spine
```
解决方法：
1. 右键对应项，点击“编辑书籍”  
如果可以点开，说明整体格式没有问题
2. 找到container.xml，里面应该有一个opf文件的路径：  
full-path="OPS/fb.ops"
3. 找到这个文件并打开它：  
找到\<package\>这一项，看看里面有没有一个“mlns”的key，如果有，将其更改为“xmlns”，然后保存并退出编辑器。
4. 查看该epub是否可以正常阅读。