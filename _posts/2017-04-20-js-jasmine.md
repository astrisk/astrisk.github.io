---
title:  "JAVASCRIPT AUTO TEST-JASMINE"
date:   2017-04-20 16:27:03
categories: JAVASCRIPT
---

**简介**

Jasmine是一款针对javaScript代码的单元测试框架。它不依赖于其他框架；不依赖于DOM; BDD式语法，方便书写测试代码。

第一个简单的测试：

{% highlight c %}

describe("A suite", function() {
  it("contains spec with an expectation", function() {
    expect(true).toBe(true);
  });
});

{% endhighlight %}

**Unit、Jasmine基本概念对应关系**
- 测试套件Suites: describe
- 测试用例 TestCase：it
- 测试断言 Assert：expect
- Setup: beforeEach
- teardown: afterEach

**匹配函数: to***(arg)**

- 内置匹配函数:
	- toBe() 
	- toBeDefined() 
	- toBeUndefined() 
	- toBeNull() 
	- toBeTruthy() 
	- toBeFalsy() 
	- toEqual() 
	- toBeLessThan() 
	- toBeGreaterThan() 
	- toContain() 
	- toBeCloseTo() 
	- toHaveBeenCalled() 
	- toHaveBeenCalledWith() 
	- toMatch() 
- 自定义匹配函数
	- 示例：http://jasmine.github.io/2.0/custom_matcher.html
**跳过和挂起测试**
- disable suites ：xdescribe
- disable testCase：xit
	- Xdescribe 和xit的测试，运行期间将被skip,它们也不会被显示在运行结果中。
- Pending specs: pending()
	- Pending 的spec 不会运行，但是测试名会在运行结果中显示为pending
- Pending specs
	- can be declared 'xit'
	- can be declared with 'it' but without a function
	- can be declared by calling 'pending' in the spec body
- Spies
	- Spy函数具有双重功能：做为测试的stub和跟踪函数的调用
	- and.callThrough
	- and.returnValue
	- and.callFake
	- and.throwError
	- and.stub
	- calls.any()
	- calls.count()
	- calls.argsFor(index):
	- calls.all():
	- calls.mostRecent():
	- calls.first()
**Mock Timeout**
- Install function: jasmine.clock().install();
- clock is ticked forward: jasmine.clock().tick(milliseconds)
- Asynchronous Support
	- Calls to beforeEach, it, and afterEach can take an optional single argument that should be called when the async work is complete.
	- This spec will not start until the done function is called in the call to beforeEach above. And this spec will not complete until its done is called.
	- Jasmine默认异步调用超时为5S，可以通过设置jasmine.DEFAULT_TIMEOUT_INTERVAL为自定义超时时间

{% highlight c %}

describe("Asynchronous specs", function() {
  var value;
beforeEach(function(done) {
    setTimeout(function() {
      value = 0;
      done();
    }, 1);
  });
it("should support async execution of test preparation and expectations", function(done) {
    value++;
    expect(value).toBeGreaterThan(0);
    done();
  });

{% endhighlight %}

**info**
- http://jasmine.github.io/2.0/introduction.html
