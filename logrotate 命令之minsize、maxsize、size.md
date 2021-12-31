# logrotate 命令之minsize、maxsize、size

`logrotate` 是一个日志管理程序，用来把旧的日志文件删除（备份），并创建新的日志文件，这个过程称为"转储"。

此处重点解释核心参数：`hourly`、`minsize`、`maxsize`、`size`、`rotate`

## 1. 滚动周期

hourly表示滚动周期为1小时，即 一个小时内只对日志文件进行一次滚动操作，不管日志文件的大小如何。

意思是说：如果在0:00--0:59之内，任意执行一次logrorate命令就可以使demo.txt发生滚动并压缩，但是同一小时内第二次执行logrotate命令就没有作用，因为你将滚动周期设置成了hourly，一个小时内只滚动一次，当logrorate发现当前一小时内已经发生了一次，就不会再次滚动了。只有等到下一个滚动周期1:00--1:59之内，再次执行logrorate命令才会触发滚动。同理适用daily、weekly、monthly、yearly。


## 2. maxsize

```bash
/var/log/demo.txt
{
        rotate 10
        hourly
        maxsize 100k
        missingok
        notifempty
        delaycompress
        compress
        postrotate
                /usr/lib/rsyslog/rsyslog-rotate
        endscript
}
```



由于将maxsize设置成了100k，当执行logrorate指令时，只要demo.txt的大小超过100k，即使时间没到下一个滚动周期内，也会发生滚动。那么同一小时内0:00--0:59就会发生多次滚动。换句话说，如果demo.txt的大小一直没有达到maxsize，那么一个滚动周期就只会发生一次滚动，即当前滚动周期内第一次执行logrorate指令就会触发滚动。以后的59分钟都不会发生滚动。maxsize可以使滚动提前发生，再下一个滚动周期到来之前发生多次滚动。

总结一句话：每满足maxsize就滚动一次，不满足则滚动1次，（每个滚动周期内，n次或1次）



## 3. minsize

minsize表示，执行logrorate命令时，只要当demo.txt文件大小超过100k，才会发生滚动，但是一个滚动周期内只会发生一次滚动。比如logrotate还是每分钟执行一次，在0:33时，demo.txt的大小超过了100k，此时logrotate命令就会使日志发生滚动。然后在当前滚动周期后面的时间里，即使demo.txt大小再次超过100k，也不会再发生滚动，即minsize不能使滚动提前发生。如果demo.txt的大小一直没有达到minsize，那么这个滚动周期内是不会触发滚动的。即如果0：00--0：59之每次执行logrotate指令时，demo.txt的大小都小于100k，那么demo.txt是不会发生滚动的。


总结一句话：满足minsize则滚动一次，不满足则滚动0次，(每个滚动周期内，1次或0次)



## 4. size



首先明确一点：size参数跟滚动周期参数：hourly、daily、monthly、yearly完全是互斥的。即是说，一旦设置了size参数，滚动周期参数就自动无效了，只要每次执行logrotate指令时，demo.txt文件大小超过size，就会触发一次滚动，没有滚动周期一说了。

总结一句话：满足size就滚动一次，（没有，滚动周期内，n次）


## 5. rotate

首先明确一点：这个参数一定要设置，如果不设置则默认为1 ， 那么只会保存一个滚动压缩的文件和一个正在输出日志文件demo.txt。

如果设置了rotate 5 ，那么只会保留5个滚动后的归档文件，时间比较旧归档文件就会被删除。注：永远只会保留5个归档文件，而不是每个滚动周期只保留5个归档文件。即每次执行logrotate指令时，会自动判断当期总的归档文件个数是否超过了5，如果超过了，再滚动完成后就会自动删除最老的一个。如果是rotate 0 的话则不会有任何归档文件保存下来。

## 6. maxage

自动删除掉超过maxage指定天数的归档文件。



## 参考链接

[logrotate 轮转核心参数](https://blog.csdn.net/sinat_36358653/article/details/107390349)

[man logrotate](https://linux.die.net/man/8/logrotate)