# Git

## git remote

### q1：如何解决git clone速度慢的问题？

答：转接到gitee，把github远程库上的地址复制到gitee上，通过从URL导入github远程库地址，可以达到git clone速度快的目的，但是这时候push的时候还是gitee的地址，我们就需要通过**git remote set-url origin [github远程库地址]**来修改远程库的地址，从而使push的时候也能推到github远程库上

## git log

### q1：查看git分支树

答：git log --oneline --graph --decorate --all