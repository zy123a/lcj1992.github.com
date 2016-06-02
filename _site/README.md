foolchild
=================

博客地址：[foolchild](http://foolchild.cn)

### 博客类型

1. `api`　旨在实用,高频的，在地铁上没事看看，毕竟一些东西是需要记忆的,如[vi](/2015/12/27/vi), [sed](/2015/12/26/sed), [awk](/2015/12/25/awk),[idea快捷键](2015/11/25/ideaShortCut)

2. `外文翻译`　自己英语水平不够;　自己走一遍比单纯转个连接要强; 之前的认知可能存在偏差，自己的博客自己可以改，不断改认识不断深化，如[jvm　internal](/2015/09/03/jvm_internal),[tcp状态机](/2015/12/25/tcpFSM)

3.  `别人的or笔记`　见上2,3,如[文件系统x问](/2016/01/07/fileSysQA),[并发基础](/2015/11/25concurrent)

4.  `自己做的实验，源码研究` 每个程序员都要有research能力　[cpu高了](/2015/12/17/cpuHigh) ,[磁盘满了](/2015/12/17/diskFull)　

关于2,3　

### 写博注意点

1. toc link中的字母要全小写。
2. post title categories后必须有个空格
3. 标题id命名以word1_word2形式命名
4. 如果git pages支持table
使用redcarpet，需在_config.xml中添加如下配置(<http://stackoverflow.com/questions/16099153/table-not-render-when-use-redcarpet-in-jekyll-github-pages>)
markdown: redcarpet
redcarpet:
  extensions: ["no_intra_emphasis", "fenced_code_blocks", "autolink", "tables", "with_toc_data"]
使用kramdown,表格前后不能直接挨着,都多个空行就行
