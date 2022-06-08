# pyinstaller 打包文件
```bash
pyinstaller -F -w (-i icofile) 文件名.py
```

解释一下该命令：
1、-w 表示在打包好程序后，双击.exe文件不会不会不会（重要的事情说三遍）出现黑色的命令窗口。（如果你的程序有print等输出命令，则输出的内容就在此命令窗口中显示）
2、小括号中的内容是可以省略的，-i 表示给.exe文件一个图标，icofile表示图标的文件名，图标的图片格式为.ico格式的。不懂.ico图片的可以百度一下。打包时如果需要给.exe加入图标，将图标文件放到和.py文件同一目录下。
3、文件名.py 这里的文件名是你要打包的.py文件的名称。



# 引用moviepy 后用pyinstaller打包出错
参考：
- https://blog.csdn.net/J_____Q/article/details/120123232

- https://blog.csdn.net/LaoYuanPython/article/details/105346224?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163085302716780255215097%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163085302716780255215097&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduend~default-1-105346224.pc_v2_rank_blog_default&utm_term=AttributeError:%20module%20%27moviepy.audio.fx.all%27%20has%20no%20attribute%20%27audio_fadein%27&spm=1018.2226.3001.4450

- https://www.cnblogs.com/rustygear/p/13584250.html

moviepy下的子包使用的是一种动态加载模块的模式加载包下的模块的
pyinstaller对这种模式不能处理，所以需要我们手动处理一下：
将 audio > fx > all > _ init _.py 和 video > fx > all > _ init _.py改为静态加载


这段代码可以实现动态加载：
```python
import pkgutil

import moviepy.audio.fx as fx

__all__ = [name for _, name, _ in pkgutil.iter_modules(
    fx.__path__) if name != "all"]

for name in __all__:
    # exec("from ..%s import %s" % (name, name))
    print("from  moviepy.audio.fx import %s" % (name))
```
运行结果：
`
```
from  moviepy.audio.fx import audio_fadein
from  moviepy.audio.fx import audio_fadeout
from  moviepy.audio.fx import audio_left_right
from  moviepy.audio.fx import audio_loop
from  moviepy.audio.fx import audio_normalize
from  moviepy.audio.fx import volumex
```
`
其中，发现了这种奇妙的用法，可以快速找到列表中的中间项。
感觉这样实现超简单，要是用c语可能很麻烦，在这里记录下。

```python
mylist=[('123','albl','qwe'),'wqe','asd','123']
__all__ = [name for _, name, _ in mylist if name != "all"]
print(__all__)
```
运行结果
`
['albl', 'q', 's', '2']
`


# Python 线程相关
## 线程的开始、暂停、结束
```python
class MyThread(threading.Thread):
    def __init__(self):
        super().__init__()
        self.__flag = threading.Event()     # 用于暂停线程的标识
        self.__flag.set()       # 设置为True
        self.__running = threading.Event()      # 用于停止线程的标识
        self.__running.set()      # 将running设置为True
        self.setDaemon(True)  #主线程运行结束时不对这个子线程进行检查而直接退出，同时所有daemon值为True的子线程将随主线程一起结束，而不论是否运行完成。

    
    def pause(self):
        self.__flag.clear()     # 设置为False, 让线程阻塞

    def resume(self):
        self.__flag.set()    # 设置为True, 让线程停止阻塞

    def stop(self):
        self.__flag.set()       # 将线程从暂停状态恢复, 如何已经暂停的话
        self.__running.clear()        # 设置为False    

class VideoProcessor(QObject,MyThread):
    ImageSignal=pyqtSignal(np.ndarray)
    def __init__(self,state) -> None: 
        super(VideoProcessor, self).__init__()       
        self.state=state
        self.cap = cv2.VideoCapture("D:/2.flv")
        # self.cap = cv2.VideoCapture(0)
        self.userid=self.state.userid
        self.useridbytes=self.state.userid.encode()
        self.meetidbytes=self.state.meetingid.encode()
        self.start()
    #region run2
    def run(self): 
        while self._MyThread__running.isSet():
            self._MyThread__flag.wait()      # 为True时立即返回, 为False时阻塞直到内部的标识位为True后返回   
            success, frame=self.cap.read()#读取视频流 
            if  success:             
                show = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)   
                self.ImageSignal.emit(show)
                encodeshow=VideoConvert.JpegCompress(show)
                imbyt=encodeshow.tobytes()#全传
                self.state.audiovideoconn.Send(DataType.image_c_to_s,[self.useridbytes,self.meetidbytes,imbyt])  
            else: break    
            time.sleep(0.04)
        self.cap.release()
    #endregion
```

## 进程同步互斥
参考：https://www.cnblogs.com/zyk01/p/11375772.html
### 一、锁机制:  multiprocess.Lock
使用互斥锁维护执行顺序：

```python
# 使用锁机制维护执行顺序
 
from multiprocessing import Process, Lock
from os import getpid
from time import sleep
from random import random
 
 
def work(lock, i):
    lock.acquire()    # ⚠️开锁进门
    print("%s: %s is running" %(i, getpid()))
    sleep(random())
    print("%s: %s is done" %(i, getpid()))
    lock.release()    # ⚠️出门，归还钥匙
 
if __name__ == '__main__':
    lock = Lock()    # ⚠️实例化一把锁，一把钥匙
 
    for i in range(5):
        p = Process(target=work, args=(lock, i))
        p.start()
```

上面这种情况虽然使用加锁的形式实现了顺序的执行，
却使得进程变成了串行执行，这样确实会浪费些时间，但是保证了数据的安全.

使用锁机制可以保证多个进程想要修改同一块数据时，在同一时间点只能有一个进程进行修改，即串行的修改，牺牲速度而保证安全性.

虽然可以用文件共享数据实现进程间通讯，但问题是：1. 效率低（共享数据基于文件，而文件是硬盘上的数据）；2. 需要自己加锁处理。因此我们最好找寻一种解决方案能够兼顾：1. 效率高（多个进程共享一块内存的数据）；2. 帮我们处理好锁的问题。这就是mutiprocessing模块为我们提供的基于消息的IPC通信机制：队列和管道

队列和管道都是将数据存放于内存中，而队列又是基于（管道+锁）实现的，可以让我们从复杂的问题中解脱出来，我们应该尽量避免使用共享数据，尽可能使用消息传递和队列，避免处理复杂的同步和锁问题，而且在进程数目增多时，往往可以获得更好的可扩展性.

### 二、信号量：multiprocessing.Semaphore

1. 信号量Semaphore允许一定数量的线程在同一时间点更改同一块数据，很形象的例子：厕所里有3个坑，同时可以3个人蹲，如果来了第4个人就要在外面等待，直到某个人出来了才能进去.

2.信号量同步基于内部计数器，每调用一次acquire()，计数器-1，每调用一次release()，计数器+1，当计数器为0时，acquire()调用被阻塞。这是迪克斯彻（Dijkstra）信号量概念P()和V()的Python实现，信号同步机制适用于访问像服务器这样的有限资源.

3.注意：信号量与进程池的概念很像，要注意区分，信号量涉及到加锁的概念.
方法
- obj = Semaphore(4)：实例化一把锁，4把钥匙，钥匙可以指定（最小0，最大未知）

- obj.acquire()：加锁，锁住下方的语句，直到遇到obj.release()方可解锁

- obj.release()：释放，释放obj.acquire()和obj.release()的语句数据，此方法如果写在加锁之前，便多了一把钥匙

基本用法
```python
# 一把锁，多把钥匙
 
from multiprocessing import Semaphore
 
s = Semaphore(2)    # 实例化一把锁，配2把钥匙
 
s.release()     # 可以先释放钥匙，变成3把钥匙
s.release()     # 再释放一把钥匙，现在变成了4把钥匙
 
s.acquire()
print(1)
 
s.acquire()
print(2)
 
# 加上先释放的钥匙，我门总共有4把钥匙
s.acquire() # 顺利执行
print(3)
 
s.acquire() # 顺利执行
print(4)
 
# 钥匙全部被占用
s.acquire() # 此处阻塞住，等待钥匙的释放
print(5)    # 不会被打印
```
### 三、事件机制：multiprocessing.Event
Python线程的事件用于主线程控制其它线程的执行，事件主要提供了三个方法：set、wait，clear

事件处理机制：全局定义了一个“Flag”，如果“Flag”值为False，程序执行wait()方法会被阻塞；
如果“Flag”值为True，程序执行wait()方法便不会被阻塞.
方法
- obj.is_set()：默认值为False，事件是通过此方法的bool值去标示wait()的阻塞状态

- obj.set()：将is_set()的bool值改为True

- obj.clear()：将is_set()的bool值改为False

- obj.wait()：is_set()的值为False时阻塞，否则不阻塞

```python
# 模拟红绿灯
 
from multiprocessing import Process, Event
import time
import random
 
 
def Tra(e):
    print("\033[32m绿灯亮\033[0m")
    e.set()
 
    while 1:
        if e.is_set():
            time.sleep(3)
            print("\033[31m红灯亮\033[0m")
            e.clear()
        else:
            time.sleep(3)
            print("\033[32m绿灯亮\033[0m")
            e.set()
 
 
def Car(e, i):
    e.wait()
    print("第%s辆小汽车过去了" % i)
 
 
if __name__ == '__main__':
    e = Event()
 
    tra = Process(target=Tra, args=(e,))
    tra.start()
 
    for i in range(100):    # 模拟一百辆小汽车
        time.sleep(0.5)
        car = Process(target=Car, args=(e, i))
        car.start()
```


## 线程池
参考：https://www.jb51.net/article/221703.htm
### 1、为什么要使用线程池呢？
因为线程执行完任务之后就会被系统销毁，下次再执行任务的时候再进行创建。这种方式在逻辑上没有啥问题。但是系统启动一个新线程的成本是比较高，因为其中涉及与操作系统的交互，操作系统需要给新线程分配资源。打个比方吧！就像软件公司招聘员工干活一样。当有活干时，就招聘一个外包人员干活。当活干完之后就把这个人员辞退掉。你说在这过程中所耗费的时间成本和沟通成本是不是很大。那么公司一般的做法是：当项目立项时就确定需要几名开发人员，然后将这些人员配齐。然后这些人员就常驻在项目组，有活就干，没活就摸鱼。线程池也是同样的道理。线程池可以定义最大线程数，这些线程有任务就执行任务，没任务就进入线程池中歇着。
### 2、线程池怎么用呢？
线程池的基类是concurrent.futures模块中的Executor类，而Executor类提供了两个子类，即ThreadPoolExecutor类和ProcessPoolExecutor类。其中ThreadPoolExecutor用于创建线程池，而ProcessPoolExecutor用于创建进程池。
Exectuor 提供了如下常用方法：
- submit(fn, \*args, \*\*kwargs)：将 fn 函数提交给线程池。\*args 代表传给 fn 函数的参数，\*kwargs 代表以关键字参数的形式为 fn 函数传入参数。
- map(func, \*iterables, timeout=None, chunksize=1)：该函数类似于全局函数 map(func, \*iterables)，只是该函数将会启动多个线程，以异步方式立即对 iterables 执行 map 处理。
- shutdown(wait=True)：关闭线程池。

程序将 task 函数提交（submit）给线程池后，submit 方法会返回一个 Future 对象，Future 类主要用于获取线程任务函数的返回值。由于线程任务会在新线程中以异步方式执行，因此，线程执行的函数相当于一个“将来完成”的任务，所以 Python 使用 Future 来代表。

Future 提供了如下方法：
- cancel()：取消该 Future 代表的线程任务。如果该任务正在执行，不可取消，则该方法返回 False；否则，程序会取消该任务，并返回 True。

- cancelled()：返回 Future 代表的线程任务是否被成功取消。
- running()：如果该 Future 代表的线程任务正在执行、不可被取消，该方法返回 True。
- done()：如果该 Funture 代表的线程任务被成功取消或执行完成，则该方法返回 True。
- result(timeout=None)：获取该 Future 代表的线程任务最后返回的结果。如果 Future 代表的线程任务还未完成，该方法将会阻塞当前线程，其中 timeout 参数指定最多阻塞多少秒。
- exception(timeout=None)：获取该 Future 代表的线程任务所引发的异常。如果该任务成功完成，没有异常，则该方法返回 None。
- add_done_callback(fn)：为该 Future 代表的线程任务注册一个“回调函数”，当该任务成功完成时，程序会自动触发该 fn 函数。

在用完一个线程池后，应该调用该线程池的 shutdown() 方法，该方法将启动线程池的关闭序列。调用 shutdown() 方法后的线程池不再接收新任务，但会将以前所有的已提交任务执行完成。当线程池中的所有任务都执行完成后，该线程池中的所有线程都会死亡。

如果通过非阻塞的方法来获取线程任务最后的返回结果呢？这里就需要使用线程的回调函数来获取线程的返回结果。

```python
from concurrent.futures import ThreadPoolExecutor
import threading
import time


def async_add(max):
    sum = 0
    for i in range(max):
        sum = sum + i
    time.sleep(1)
    print(threading.current_thread().name + "执行求和操作求得的和是=" + str(sum))
    return sum


with ThreadPoolExecutor(max_workers=2) as pool:
    # 向线程池提交一个task
    future1 = pool.submit(async_add, 20)
    future2 = pool.submit(async_add, 50)


    # 定义获取结果的函数
    def get_result(future):
        print(threading.current_thread().name + '运行结果：' + str(future.result()))


    # 查看future1代表的任务返回的结果
    future1.add_done_callback(get_result)
    # 查看future2代表的任务的返回结果
    future2.add_done_callback(get_result)
    print('------------主线程执行结束----')

```

运行结果
```
------------主线程执行结束----
ThreadPoolExecutor-0_1执行求和操作求得的和是=1225
ThreadPoolExecutor-0_1运行结果：1225
ThreadPoolExecutor-0_0执行求和操作求得的和是=190
ThreadPoolExecutor-0_0运行结果：190

```





# Login 窗体的内存问题
参考：https://blog.csdn.net/goforwardtostep/article/details/53647146
在正常创建窗口后，我们一般会调用close()方法来关闭窗口，这里我们看一下Q助手中关于close()方法的介绍。

```
bool QWidget::close()
Closes this widget. Returns true if the widget was closed; otherwise returns false.

First it sends the widget a QCloseEvent. The widget is hidden if it accepts the close event. If it ignores the event, nothing happens. The default implementation of QWidget::closeEvent() accepts the close event.

If the widget has the Qt::WA_DeleteOnClose flag, the widget is also deleted. A close events is delivered to the widget no matter if the widget is visible or not.
```

```
调用close()方法后首先它会向widget发送一个关闭事件（QCloseEvent）。如果widget接受了关闭事件（QCloseEvent），窗口将会隐藏（实际上调用hide()）。如果widget不接受关闭事件，那么窗口将什么也不做。默认情况下widget会接受关闭事件,我们可以重写QCloseEvent事件，可以选择接受或者不接受。

如果widget设置了Qt::WA_DeleteOnClose属性，widget将会被释放。不管widget是否可见，关闭事件都会传递给widget。即接收到QCloseEvent事件后，除了调用hide()方法将窗口隐藏，同时会调用deleteLater()方法将窗口释放掉，不会再占用资源。

所以说调用close()并不一定就会将窗口对象销毁。而只有设置了 Qt::WA_DeleteOnClose属性才会删除销毁。如果这个属性没有设置，close()的作用和hide（），setvisible（false）一样，只会隐藏窗口对象而已，并不会销毁该对象。
```

Qt::WA_DeleteOnClose属性在Qt助手中的解释

```
Qt::WA_DeleteOnClose Makes Qt delete this widget when the widget has accepted the close event (see QWidget::closeEvent()).

如果窗口设置了Qt::WA_DeleteOnClose 这个属性，在窗口接受了关闭事件后，Qt会释放这个窗口所占用的资源。
```

所以如果我们在程序中通过 new 的方式创建一个窗口，可以给该窗口设置 Qt::WA_DeleteOnClose属性。这样在关闭这个窗口时Qt能够自动回收该窗口所占用的资源，这样能够及时回收无效的资源，有用利于节约内存空间。
## 问题
如果按照毕设中的方法，表面上调用.close()方法，将LoginWindow关闭了，但是他还占用着内存，影响安全性。周老师原话：==“表面上看着没了，实际上还在，会带来安全性风险，因为登录窗体的数据在内存中一直没释放，攻击者可以通过钩子程序将里面的用户名和密码获取出来”==
可以采用下面这种方法：
```python
if __name__ == '__main__':
    app = QApplication(sys.argv)
    login = LoginForm()
    # login.show()
    if not login.exec_():
        sys.exit(0)

    mainWindow = MainForm()
    mainWindow.clicked.connect(login.handleClicked)
    # login.closeSignal.connect(mainWindow.Start)

    mainWindow.show()
    sys.exit(app.exec_())
```

但是这样就无法传递参数了，于是，修改成下面这种方法，不要忘了要在loginwindow中加上一句:`self.setAttribute(Qt.WA_DeleteOnClose)`用来关闭时释放内存。

```
if __name__=='__main__':
    app=QApplication(sys.argv)
    #底部任务栏图标设置为自己的
    ctypes.windll.shell32.SetCurrentProcessExplicitAppUserModelID("myappid")

    login_window=LoginWindow()
    login_window.setStyleSheet(CommonHelper.readQss())

    mainwin=MainWindow()
    

    login_window.ArgsSignal.connect(mainwin.OnInit)

    if not login_window.exec_():
        sys.exit(0)
    else:
        mainwin.setStyleSheet(CommonHelper.readQss())
        mainwin.show()

    
    sys.exit(app.exec_())
```