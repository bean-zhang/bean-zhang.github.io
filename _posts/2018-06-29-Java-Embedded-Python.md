---
layout: post
permalink: /blog/2018/06/29/Java-Embedded-Python.html
category: software
tags: [software, python, java, scala, machine learning]
title: Integrate Python into Java or Scala Stack using JEP(Java Embedded Python)
---

本博客主要讲解如何使用JEP(Java Embedded Python)库将Python代码集成到Java或Scala代码中。

> 曾经人类联合起来兴建希望能够通往天堂的高塔；为了阻止人类的计划，上帝让人类说不同的语言，使人类相互之间不能沟通，通天塔计划因此失败，人类自此各散东西。 ——《圣经·旧约·创世纪》第11章

可见不同语言之间的交流存在着巨大的障碍。同样的不同编程语言之间的集成令程序员们颇为头疼。

## 为什么要在Java/Scala中集成Python

> 不忘初心，方得始终。 ——《华严经》

Java是一门面向对象的编程语言，其已占据[TIOBE Index](https://www.tiobe.com/tiobe-index/)榜首多年，可见其流行程度。很多公司的企业级应用都是用Java开发的。

Scala是一门多范式的编程语言，集成了面向对象编程和函数式编程的各种特性，其基于JVM，可与Java进行互操作。目前非常流行的专为大规模数据处理而设计的计算引擎Apache Spark便是用Scala开发的。

然而在Java和Scala中机器学习的库相对匮乏，Spark MLlib / deeplearning4j等 只支持部分机器学习算法。

在机器学习领域，Python的生态大放异彩，无论是经典机器学习算法库scikit-learn还是深度学习框架Tensorflow/Theano/Keras/PyTorch等等，成为了数据科学家和算法工程师们得心应手的工具。

面对现状，就出现了一个需求：将Java/Scala高性能强类型语言系统和Python丰富的第三方库相集成。

> 以终为始 ——《高效人士的七个习惯》

集成的目的是：在用Java/Scala开发的业务流中能够调用Python编写的机器学习模型，完成在线近实时的模型预测功能。

工作流可能有：

- 手动训练模型=>手动放置模型=>手动更新模型
- 发起一个训练请求=>离线训练=>训练完成=>存储并更新模型
- 发起一个预测请求=>载入模型在线预测=>预测完成直接返回结果

需要考虑的问题：Java/Scala<=>Python

- 训练 + 预测/评分
- Spark DataFrame <=> Pandas DataFrame (scikit-learn)
- 数据量比较大时的性能问题

## Java/Scala中集成Python的方式

### Jython + JyNI

Jython项目是用Java将Python语言重新实现，虽然其包含了Python的大部分模块，但它缺乏对C语言扩展模块的支持。然而Python机器学习相关的第三方库基本上都采用了C语言扩展，以便提升执行速度。

JyNI(Jython Native Interface)是为了使Jython能够使用CPython扩展(例如NumPy/SciPy)而设计的兼容层。然而到目前为止，JyNI还不支持整个CPython的API，所以还不能通过JyNI来使用Python的各种C语言扩展。

另外，从Jython官网可以看到，最新的发布版是来自2015年5月的Jython 2.7.0，之后便没有再更新。而JyNI一直到2017年9月还在更新发布。

- [Jython Project](http://www.jython.org/)
- [JyNI(Jython Native Interface)](https://www.jyni.org/)

### [JEP(Java Embedded Python)](https://github.com/ninia/jep)

JEP(Java Embedded Python)通过调用JNI(Java Native Interface)和CPython的API在JVM中启动Python解释器来实现Java调用Python，其本身在多线程环境中是线程安全的。

但由于部分CPython扩展本身设计不适合在子解释器里运行，比如使用了全局静态变量，会导致JVM崩溃，因此在多线程或多解释器环境中容易出错，需要进行限制。

其基本工作过程是：

- 当JVM启动时，会初始化一个顶层解释器，用于启动和关闭Python；
- 当在Java代码中创建一个Jep实例时，顶层解释器会为这个Jep实例创建一个子解释器；
- 子解释器会一直留存在内存中，直到其对应的Jep实例被关闭（jep.close()）；
- 顶层解释器会一直留存在JVM中直到JVM被销毁；

### Thrift

### REST API


## JEP(Java Embeded Python)

出于开发工程量的考虑，我们暂时选择了JEP开源库来实现Java调用Python。

### 依赖

JEP依赖的Java版本和Python版本

- Java >= 1.7
- Python 2.7, 3.3, 3.4, 3.5, 3.6 or 3.7

在同样是Java 1.8的环境中，测试了不同Python版本(2.7.10/3.6.3)+不同Jep版本(3.7.1/3.8.0rc)+不同操作系统(macOS/CentOS)的运行情况，个人比较推荐采用 Python 3.6.3 + Jep 3.8.0rc 方案。

本文使用Python包

- [numpy](http://www.numpy.org/)
- [pandas](https://pandas.pydata.org/)
- [scipy](https://www.scipy.org/)
- [scikit-learn](http://scikit-learn.org/stable/index.html)

可通过[pipenv](https://docs.pipenv.org/)或`pip`来安装Python包，例如

    pip install numpy pandas scipy scikit-learn

### 在macOS中运行

通过从官网下载.dmg文件或直接通过homebrew安装Java

    $ java -version
    $ export JAVA_HOME="$(/usr/libexec/java_home -v 1.8)" # add this `export` to ~/.bash_profile

安装[pyenv](https://github.com/pyenv/pyenv)和Python 3.6.3

    $ brew update
    $ brew install pyenv

    $ pyenv update
    $ pyenv versions
    $ pyenv install 3.6.3
    $ cd git_repo # get into git repository directory
    $ pyenv local 3.6.3
    $ pyenv versions
    $ pyenv which python
    $ pyenv which pip

在Python 3.6.3中安装所需Python包

    $ pip list
    $ pip install numpy pandas scipy scikit-learn

从源码安装[jep](https://github.com/ninia/jep)到Python 3.6.3中

    $ git clone https://github.com/ninia/jep.git
    $ cd jep
    $ git checkout v3.8rc   # version 3.8 rc tag
    $ python setup.py test  # All tests should pass or skip, no failure
    $ python setup.py install

拷贝jep的.jar和.so文件至`./lib`文件夹中

    $ cd git_repo # get into git repository directory
    $ mkdir lib
    $ cp -rp ~/.pyenv/versions/3.6.3/lib/python3.6/site-packages/jep/{jep-3.8.0.jar,jep.cpython-36m-darwin.so,libjep.jnilib} ./lib/

安装SBT: [Installing sbt on Mac](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Mac.html) 并运行

    $ brew install sbt
    $ cd git_repo
    $ export PYTHONHOME=~/.pyenv/versions/3.6.3/
    $ sbt run

### 在CentOS中运行

安装Java: [How To Install Java on CentOS and Fedora](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora)

    $ sudo yum install java
    $ java -version
    $ export JAVA_HOME=/usr/java/jdk1.8.0_171/  # add this `export` to ~/.bashrc

安装Python pip及所需开发工具

    $ sudo yum install epel-release
    $ sudo yum install python-pip
    $ sudo yum install python-devel
    $ sudo yum groupinstall "Development Tools"
    $ sudo yum install gcc gcc-c++ make patch openssl-devel zlib-devel readline-devel sqlite-devel bzip2-devel

通过[pyenv-installer](https://github.com/pyenv/pyenv-installer)安装[pyenv](https://github.com/pyenv/pyenv)和Python 3.6.3

    $ curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
    $ pyenv update
    $ pyenv versions
    $ PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.6.3 # compile with -fPIC
    $ cd git_repo # get into git repository directory
    $ pyenv local 3.6.3
    $ pyenv which python
    $ pyenv which pip

在Python 3.6.3中安装所需Python包

    $ pip list
    $ pip install numpy pandas scipy scikit-learn

从源码安装[jep](https://github.com/ninia/jep)到Python 3.6.3中

    $ git clone https://github.com/ninia/jep.git
    $ cd jep
    $ git checkout v3.8rc   # version 3.8 rc tag
    $ python setup.py test  # All tests should pass or skip, no failure
    $ python setup.py install

拷贝jep的.jar和.so文件至`./lib`文件夹中

    $ cd git_repo # get into git repository directory
    $ mkdir lib
    $ cp -rp ~/.pyenv/versions/3.6.3/lib/python3.6/site-packages/jep/{jep-3.8.0.jar,jep.cpython-36m-x86_64-linux-gnu.so,libjep.so} ./lib/

安装SBT: [Installing sbt on Linux](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Linux.html)并运行

    $ curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
    $ sudo yum install sbt
    $ cd git_repo
    $ export PYTHONHOME=~/.pyenv/versions/3.6.3/
    $ sbt run

### 在Ubuntu中运行

安装Java: [How To Install Java with Apt-Get on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)

    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update
    $ sudo apt-get install oracle-java8-installer
    $ sudo update-alternatives --config java
    $ export JAVA_HOME="/usr/lib/jvm/java-8-oracle"

安装Python pip和所需Python包

    $ sudo apt-get update
    $ sudo apt-get python-pip
    $ pip install --upgrade pip
    $ pip list
    $ pip install numpy pandas scipy scikit-learn
    
从源码安装[jep](https://github.com/ninia/jep)及拷贝jep的.jar和.so文件至`./lib`文件夹中与CentOS操作系统中类似。

安装SBT: [Installing sbt on Linux](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Linux.html)并运行

    $ echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
    $ sudo apt-get update
    $ sudo apt-get install sbt
    $ cd git_repo
    $ sbt run

## 参考资料

- [Integrating Python into Scala Stack](https://sushant-hiray.me/posts/python-in-scala-stack/)
- [How to integrate python into a scala stack to build realtime predictive models](https://www.slideshare.net/JerryChou4/how-to-integrate-python-into-a-scala-stack-to-build-realtime-predictive-models-v2-nomanuscript)
