有时候日志文件太大了，vim根本打不开，想找点东西更是不可能了

这时候可以用split命令将文件分割成小部分，处理小文见就ok了

```
split --bytes 500M --numeric-suffixes --suffix-length=2 passport.log passport.
```

`--bytes`选项表示一个文件的大小
`--numeric-suffixes`表示以数字作为拆分文件的后缀
`--suffix-length=2`表示后缀长度为2，不够的会补前缀0
`passport.log`是要拆分的文件
`passport.`是拆分文件的前缀

假如passport.log有1.8G
拆分之后就会有
passport.01
passport.02
passport.03
passport.04
