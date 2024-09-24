# 2024年选课系统更新，此项目已经无法使用，并且暂无更新计划。


[Download](https://github.com/g10guang/WHUT_Courses_System/raw/master/release/WHUT1.0.zip)

如有发现任何问题，可以邮件联系我 [g10guang@foxmail.com](g10guang@foxmail.com)，或者 github 上提交 Issue。

*Please feel free to do it.*

# 项目描述

## 开发初衷

武汉理工大学每次选课都会崩溃，不必同学们整天候着浏览器等数据刷新出来，然后又不幸地发现登陆超时或者服务器已经崩溃了，开发这个项目等于更大地增大了选课网站的压力，但是我还是希望能够能够为部分同学提供帮助。

## 用户阅读

### 主要界面

+ **登陆界面**

![登陆界面](https://github.com/g10guang/WHUT_Courses_System/blob/master/doc/img/登陆界面.png)

选项说明：

- [ ] 获取课程信息

选课是否需要请求选课系统获取用户课程信息，每个用户课程信息不完全一样，比如不同专业拥有不同的专业课。

- [x] 获取课程信息

如选择获取课程信息，务必在抢课前进行，比如第一轮选课时间，在抢课流量峰值时，不能够确保能够爬取到课程信息。如果认为爬取信息错误，请在网络环境比较好的时候重新爬取课程信息。

- [ ] 验证

是否验证学号与密码是否匹配，在抢课时可以把该选项取消，但是务必确认学号与密码匹配（否则我们怎么登陆选课系统呢？）

如果勾选了`获取课程信息`，那么默认会验证学号与密码是否匹配

- [x] 验证

我们会与教务处做一个模拟登陆，将验证学号与密码的任务交给教务处完成。

![显示密码](https://github.com/g10guang/WHUT_Courses_System/blob/master/assert/show.png) 用于 `****` 显示密码与明文显示密码的切换，如确保学号与密码匹配可以取消`验证`选项。


<button>More info</button>

打开浏览器，浏览用户使用指南

<button>Start up</button>

开始按钮


+ **查看课程信息界面**

![查看课程信息界面](https://github.com/g10guang/WHUT_Courses_System/blob/master/doc/img/查看课程信息界面.png)

<button>返回</button>

返回`登陆页面`，可以在多个账号之前切换

<button>发起抢课</button>

会把选中的课程加入到抢课队列中，系统会自动模拟登陆状态，不断地向选课系统发包，直到成功抢到课程为止。

<button>查看任务列表</button>

跳转到`查看选课任务界面`，查看选课任务的状态


+ **查看选课任务界面**

![查看选课任务界面](https://github.com/g10guang/WHUT_Courses_System/blob/master/doc/img/查看选课任务界面.png)

`状态` 描述了每个选课任务当前状态，需要注意的是需要该页面不会动态刷新，如果需要刷新数据显示，需要点击 <button>返回</button> 返回到`查看课程信息界面`，再点击<button>查看任务列表</button>得到最新数据。

## 二次开发者阅读

以下是武汉理工大学选课系统截图：

![武汉理工选课系统](https://github.com/g10guang/WHUT_Courses_System/blob/master/doc/img/选课系统截图.png)

如果读者学校的选课系统类似，那么**欢迎**把被仓库改造为读者学校的抢课软件

### 技术方案

#### TkInter GUI

界面是使用了Python内置模块TkInter完成，TkInter是一个跨平台模块（Windows / MacOS / Linux等主流操作系统皆可使用）

#### 爬虫

本系统初次方案是使用了 Scrapy 爬虫框架去爬取教务处系统信息，然后将课程信息保存到`data/学号`目录下，Scrapy 爬虫代码存放于 WHUT 目录下。【*已弃用*】

但是由于打包失败，后转为`requests + BeautifulSoup`去抓取选课系统中的信息。

**requests**：用于模拟发送 http 请求，用于与选课系统交互

**BeautifulSoup**：用于解析选课系统网站的 html 文档，分析 html 文档树，提取需要的信息，比如链接和课程信息

> 爬虫问题：当初选用 scrapy 框架是底层使用 Twisted 异步框架，爬虫是 IO 密集，瓶颈是系统响应速度和网络传输速度。现在使用了 `requests + BeautifulSoup` 方案，只是使用了单线程、非异步方式，整个过程非常慢，大概耗时 1~2min，后期有待优化。

#### 抢课逻辑

开启一个新的进程完成选课逻辑，每添加一个新的选课任务，就不断在该进程中创建一个新的线程去完成抢课任务。

> 使用新进程的原因：python 有 GIL 锁，在同一进程中，同一时间只有一个线程能够处于运行状态。PS：Jython 等解决方案可以克服 GIL 锁发挥多线程力量，但是权衡后还是决定使用多进程解决方案。

使用`requests`进行不断模拟选课系统登陆，然后不断地发送选课的 http 包，分析选课系统返回数据，判断是否选课成功。

#### 选课系统反馈信息分析

选课系统会返回 json 或者是 html 页面，选课系统设计很烂，无论什么情况都会返回 200 状态码。

以下情况下反馈 json：

+ 登陆超时

```json
{"message":"登录超时，请重新登录！","statusCode":"300"}
```

+ 课程重复，不能选已选课程

```json
{"message":"课程重复，不能选已选课程","statusCode":"300"}
```

+ 目前不在选课时间，不能选课

```json
{"message":"目前不在选课时间，不能选课","statusCode":"300"}
```

+ 该课程与已选课程上课时间冲突

```json
{"message":"该课程与已选课程上课时间冲突","statusCode":"300"}
```

+ 该门课程容量不足，选课失败

```json
{"message":"该门课程容量不足，选课失败","statusCode":"300"}
```

+ 你所选的课程的课程性质已超出了限制的可选门数，不能选择此课程性质的课程！

```json
{"message":"你所选的课程的课程性质已超出了限制的可选门数，不能选择此课程性质的课程！","statusCode":"300"}
```

以下情况返回 html 页面：

+ 选课成功

> 由于只有成功选课时才会返回 html 页面，其他情况下返回 json，我们采用的方案是尝试 json.loads(response.text) 解析，如果解析失败，认为就是返回了 html 页面，否则解析为了则认为选课失败，通过`message`来判断具体是属于哪一种情况


除了成功选课外，其他情况会不断地发送选课请求，直到选课成功。

#### 配置信息

配置处于 `app/__init__.py` 与 `app/spider/__init__.py` 文件中，尽可能减少硬编码。
