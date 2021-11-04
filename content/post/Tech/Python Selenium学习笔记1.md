---
title: "Python Selenium学习笔记"
date: 2016-09-26
tags: ["Python","Selenium","UnitTest"]
draft: false
---


## 如何运行单元测试时只运行一个浏览器，而不是每个TestCase都打开一个浏览器
具体实现方法：
```
class BaseUnitTest(unittest.TestCase):
    # --实例属性区--
    """是否开启TestCase级别的单驱动模式,默认False
    开启时, 一个TestCase只会打开一个浏览器,也就是method之间是公用浏览器,
    而不是每个测试Method都打开一个浏览器
    """
    SINGLE_DRIVER = False

    # --类方法区--
    @classmethod
    def setUpClass(cls):
        cls.driver = None

    @classmethod
    def tearDownClass(cls):
        if cls.SINGLE_DRIVER:
            BaseUnitTest.__quit_driver(cls.driver)
    
    # --静态方法区--
    @staticmethod
    def __new_driver():
        driver = browser(env.BROWSER, env.DRIVER_PATH)
        driver.maximize_window()
        return driver

    @staticmethod
    def __quit_driver(driver):
        if env.BROWSER.upper() == 'PHANTOMJS':
            # 这里是为了能够真正的退出
            driver.service.process.send_signal(signal.SIGTERM)
        driver.quit()
        
     def setUp(self):
        if self.SINGLE_DRIVER:
            if self.__class__.driver is None:
                self.driver = BaseUnitTest.__new_driver()
                self.__class__.SINGLE_DRIVER = self.SINGLE_DRIVER
                self.__class__.driver = self.driver
            else:
                self.driver = self.__class__.driver
        else:
            self.driver = BaseUnitTest.__new_driver()

    def tearDown(self):
        if not self.SINGLE_DRIVER:
            BaseUnitTest.__quit_driver(self.driver)
```

## 如何用多线程运行Selenium测试
> * `threading.Thread`线程方式,这种方式需要注意的是多个线程之间的相互影响，导致共享值的变化，下面是实现方式（这种有问题，就是不知道合适的时机和方式去关闭driver）：

```python
# coding=utf-8

import unittest
import threading
from driver import browser
import env
import signal
import utils
import dataUtils
import sys
from time import ctime


class BaseUnitTest(unittest.TestCase):
    # --实例属性区--
    """是否开启TestCase级别的单驱动模式,默认False
    开启时, 一个TestCase只会打开一个浏览器,也就是method之间是公用浏览器,
    而不是每个测试Method都打开一个浏览器
    """
    SINGLE_DRIVER = False

    # --类方法区--
    @classmethod
    def setUpClass(cls):
        env.thread_local.SINGLE_DRIVER = cls.SINGLE_DRIVER

    @classmethod
    def tearDownClass(cls, is_thread=False):
        if is_thread and env.thread_local.SINGLE_DRIVER:
            BaseUnitTest.__quit_driver(env.thread_local.driver)

    # --静态方法区--
    @staticmethod
    def __new_driver():
        driver = browser(env.BROWSER, env.DRIVER_PATH)
        driver.maximize_window()
        return driver

    @staticmethod
    def __quit_driver(driver):
        if env.BROWSER.upper() == 'PHANTOMJS':
            # 这里是为了能够真正的退出
            driver.service.process.send_signal(signal.SIGTERM)
        driver.quit()

    # --实例方法区--
    # 1.重写父类区
    def __init__(self, methodName='runTest'):
        ''' overide the TestCase __init__ Method
            so need to recall the TestCase __init__ method
        '''
        super(BaseUnitTest, self).__init__(methodName)
        self.__testMethodName = methodName

    def setUp(self):
        if env.thread_local.SINGLE_DRIVER:
            if env.thread_local.driver is None:
                env.thread_local.driver = BaseUnitTest.__new_driver()
            self.driver = env.thread_local.driver
        else:
            self.driver = BaseUnitTest.__new_driver()

    def tearDown(self):
        if not env.thread_local.SINGLE_DRIVER:
            BaseUnitTest.__quit_driver(self.driver)

    # 2.自定义方法区
    def assertEqualUnicode(self, a, b):
        a = dataUtils.decodeToUnicode(a)
        b = dataUtils.decodeToUnicode(b)
        self.assertEqual(a, b)

    # 3.私有方法区
    def __exc_info(self):
        ''' copy from TestCase ,aim to overide the TestCase __call__ method
        '''
        exctype, excvalue, tb = sys.exc_info()
        if sys.platform[:4] == 'java':  # tracebacks look different in Jython
            return (exctype, excvalue, tb)
        newtb = tb.tb_next
        if newtb is None:
            return (exctype, excvalue, tb)
        return (exctype, excvalue, newtb)

    def __call__(self, result=None):
        ''' overide the TestCase __call__ method
            aim to add insert_img_decorator to the test_case method
        '''
        if result is None:
            result = self.defaultTestResult()
        result.startTest(self)
        testMethod = getattr(self, self.__testMethodName)
        newMethod = utils.insert_img_decorator(
            self.__testMethodName)(testMethod)
        try:
            try:
                self.setUp()
            except KeyboardInterrupt:
                raise
            except:
                result.addError(self, self.__exc_info())
                return

            ok = 0
            try:
                newMethod(self)
                ok = 1
            except self.failureException, e:
                result.addFailure(self, self.__exc_info())
            except KeyboardInterrupt:
                raise
            except:
                result.addError(self, self.__exc_info())

            try:
                self.tearDown()
            except KeyboardInterrupt:
                raise
            except:
                result.addError(self, self.__exc_info())
                ok = 0
            if ok:
                result.addSuccess(self)
        finally:
            result.stopTest(self)
```

> * `multiprocessing`进程方式，这种方式可以避免线程上的值共享被共同访问时的问题，但是需要注意其他问题，就是测试结果如何共享的问题，实现代码如上

## unittest执行过程分析-TestLoader
**TestLoader.discover**发现的结果如下：
```
<unittest.suite.TestSuite
    tests=[
        <unittest.suite.TestSuite
            tests=[
                <unittest.suite.TestSuite tests=[]>,
                <unittest.suite.TestSuite
                    tests=[
                        <exam_manager_test.ExamManageTest testMethod=test_a_add_exam>,
                        <exam_manager_test.ExamManageTest testMethod=test_b_search_exam>
                        ]
                >
            ]
        >,
        <unittest.suite.TestSuite
            tests=[
                <unittest.suite.TestSuite tests=[]>,
                <unittest.suite.TestSuite
                    tests=[
                        <exam_manager_1_test.ExamManageTest testMethod=test_c_add_exam>,
                        <exam_manager_1_test.ExamManageTest testMethod=test_d_search_exam>
                        ]
                >
            ]
        >
    ]
>
```
数据知道了，那具体流程图如下：
```flow
st=>start: Start
suite=>operation: TestSuite Run
teardownclass=>operation: TestCase TearDownPreClass
setupmodule=>operation: TestCase SetUpModule
setupclass=>operation: TestCase SetUpClass
for=>operation: Iterator TestSuite
cond=>condition: Is TestSuite?
case=>operation: TestCase Run
setup=>operation: TestCase SetUp
teardown=>operation: TestCase TearDown
islastcase=>condition: Is Last TestCase?
teardownmodule=>operation: TestCase TearDownModule
teardownclass2=>operation: TestCase TearDownClass
end=>end: End

st->suite->for->cond
cond(yes,right)->suite(right)
cond(no)->teardownclass->setupmodule->setupclass
setupclass->setup->case->teardown->islastcase
islastcase(no,right)->for(left)
islastcase(yes)->teardownclass2->teardownmodule->end
```





