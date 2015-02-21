###branch:source 

jekyll源码目录

###branch:master

生成的静态页面目录

###自动提交脚本

在source分支下编辑文档后,执行update.sh即可自动把源码和生成的静态页文档提交到Github

update.h
	
	 #!/bin/bash
	 git checkout source
	 jekyll build
	 git add -A
	 git commit -m "update source"
	 cp -r _site /tmp/
	 git checkout master
	 rm -r ./*
	 cp -r /tmp/_site/* ./
	 git add -A
	 git commit -m "deploy blog"
	 git push origin master
	 git checkout source
	 echo "deploy succeed"
	 git push origin source
	 echo "push source"
