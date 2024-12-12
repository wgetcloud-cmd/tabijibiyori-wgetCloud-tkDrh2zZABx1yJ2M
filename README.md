
# c\#上位机:示波器功能


好久没有更新了，因为最近主要学习了如何用c\#去做一个示波器功能，这里的示波器主要是用于单片机的调试。下面，我主要分享一下我做示波器的一些心得：


我这里示波器是用winform做的，了解到有很多开源的曲线控件，比如：chart,Oxyplot,scottplot,hslcontrols等，当然还有一些收费的曲线控件，这里就不一一说了。同时，自己画一个曲线图也是可以的。


最开始学习别人的示波器制作，如下：


![image-20241211221100395](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211221103791-1145083562.png)
这个示波器的原理：


1\.首先定义在示波器界面代码里或者自己创建一个缓存数据的类，在定义静态的队列用来缓存数据，这里四个通道，则是4个队列



```
 public static volatile Queue<int> quedata1 = new Queue<int>(dataCountMax);//通道1数据
 public static volatile Queue<int> quedata2 = new Queue<int>(10001);
 public static volatile Queue<int> quedata3 = new Queue<int>(10001);
 public static volatile Queue<int> quedata4 = new Queue<int>(10001);

```

2\.在接收数据的地方先出一个数据，再进一个数据



```
 FrmScope.quedata1.Dequeue();
 FrmScope.quedata1.Enqueue(comData.data1);
 FrmScope.quedata2.Dequeue();
 FrmScope.quedata2.Enqueue(comData.data2);
 FrmScope.quedata3.Dequeue();
 FrmScope.uedata3.Enqueue(comData.data3);
 FrmScope.quedata4.Dequeue();
 FrmScope.quedata4.Enqueue(comData.data4);

```

3\.打开示波器界面时，先将队列填满



```
for (int i = 0; i < dataCountMax; i++) 
{
    if (quedata1.Count < dataCountMax)
    {
        quedata1.Enqueue(0);
        quedata2.Enqueue(0);
        quedata3.Enqueue(0);
        quedata4.Enqueue(0);
 
    }

}

```

4\.将队列转化为数组赋值给自己画好的控件进行定时刷新，只要保证每个数据自己的采样时间间隔一样，波形应该就很完美



```
if (ScopeRun)
{
    QueToArray(quedata1, arrScope1);
    QueToArray(quedata2, arrScope2);
    QueToArray(quedata3, arrScope3);
    QueToArray(quedata4, arrScope4);
}

Invalidate();//刷新显示 

```

由于我自己做的一个调试软件通道特别多，而且没有固定的通道，需要做到随意添加曲线的功能，并且还由于美观原因就没有采用这种做法。于是，我去学习了一下scottplot和Oxyplot控件，学习发现，对于动态曲线，Oxyplot控件显示的更好且操作更方便，Scottplot在静态曲线上则更有优势，数据可达百万级别。经过甄别，我选择了Oxyplot控件。


示波器大致界面（这里我自己做的模拟数据方便展示）：


![image-20241211225047579](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211225048817-1959300592.png)
思路如下：


1\.首先我们先和上方一样，肯定要定义一个数据缓存的队列。不过，我们这里先定义了一个曲线实体类，用来存储数据，定义了一个公共类来缓存曲线以及索引曲线的名字，队列采用的是阻塞队列（生产者与消费者问题）。


![image-20241211225133617](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211225134407-1612472811.png)


![image-20241211225145556](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211225146545-1080238033.png)


2\.模拟一下生产数据，存入队列


![image-20241211225421020](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211225422320-153011290.png)


3\.在另一个线程里面不断去索引所添加的曲线，自己可以封装一下添加和删除曲线的方法，这里就不展示了。


![image-20241211225611805](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211225612942-1399487439.png)


就这样，示波器功能就完成了。我采用阻塞队列这种方式去显示实时波形，不知道还有没有别的方法去实现，有好的见解可以评论区留言。


当然，上面做的过程中还有一些错误得必坑：


1\.我存储的时候不存储时间可以不？我觉得实际过程中如果横坐标为点的个数的话，那你曲线采样的时间则是消费者的延时时间，特别是在出现故障再次恢复曲线时会出现时间间隔导致波形问题。


2\.遍历曲线之前要先转化为list，不要在foreach里面去tolist，当你添加曲线时偶尔会报错。


![image-20241211230306994](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211230307997-1027064411.png)


3\.消费者的时间可以往短了放，因为你横坐标为时间，不会影响实际的波形。


4\.就是多线程里this.Invok的问题了，千万不要为了省事情，while循环里面先套个Invoke再说，这样是会耗时间的，只要针对你需要委托的地方用就行，别乱用。


![image-20241211230512228](https://img2023.cnblogs.com/blog/3267782/202412/3267782-20241211230513052-1653199168.png)


  * [c\#上位机:示波器功能](#c%E4%B8%8A%E4%BD%8D%E6%9C%BA%E7%A4%BA%E6%B3%A2%E5%99%A8%E5%8A%9F%E8%83%BD)

   ![](https://github.com/cnblogs_com/yizhangheka/1695816/o_230526064744_ZVArHsuH5w_0365de1b.png)    - **本文作者：** [磊磊吖](https://github.com)
 - **本文链接：** [https://github.com/zhuiyine/p/18601179](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com):[slower加速器官网](https://chundaotian.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
