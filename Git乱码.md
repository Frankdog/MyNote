# Git 中文乱码

##ls 显示乱码
  编辑 Git\etc\git-completion.bash 
  添加 alias ls="ls --show-control-chars" 

##git add 显示乱码
  `git config --global core.quotepath false`
##git commit/log 
  `i18n.commitencoding=utf-8`
  `i18n.logoutputencoding=utf-8`
