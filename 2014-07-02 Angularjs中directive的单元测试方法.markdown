翻译自：[Unit Testing an AngularJS Directive](http://blog.revolunet.com/blog/2013/12/05/unit-testing-angularjs-directive/)

在这篇文章中，我将详述如何给上周开发的[stepper directive](http://blog.revolunet.com/blog/2013/11www/28/create-resusable-angularjs-input-component/)做单元测试的过程。下周会讲到如何使用Github和Bower进行组件分离。

单元测试是一种测试项目中最小单元代码的艺术，是使程序思路清晰的基础。一旦所有的测试通过，零散的单元组合在一起也会运行的很好，因为这些单元的行为已经被独立的验证过了。

单元测试能够**避免代码出现回归性BUG**，**提高代码的质量和可维护性**，**使代码在代码库中是可信赖的**，从而提高团队合作的质量，使重构变得简单和快乐: )

单元测试的另一个用处是当发现了一个新的BUG，你可以为这个BUG写一个单元测试，当你修改了代码，使这个测试可以PASS了的时候，就说明这个BUG已经被修复了。

AngularJS最好的小伙伴儿[KarmaJS test runner](http://karma-runner.github.io/)（一个能够在浏览器中运行测试同时生成结果日志的Node.js server）还有 [Jasmine](http://pivotal.github.io/jasmine/)（定义了测试和断言的语法的库）。使用Grunt-karma将karma集成在经典且繁重的grunt 工作流中，然后在浏览器中运行测试。这里值得注意的是，karma可以将测试运行在远程的云浏览器中，比如[SauceLabs](http://saucelabs.com/)和[BrowserStack](http://www.browserstack.com/)。

**AngularJS是将是经过了严密地测试的，所以赶紧给自己点个赞，现在就开始写测试吧！**

------------

术语：

在进行下一步之前有一些术语需要说明：

*  `spec`: 想要测试的代码的说明，包括一个或多个测试条件。spec应该覆盖所有预期行为。
*  `test suite`: 一组测试的集合，定义在`Jasmine`提供的`describe`语句块中，语句块是可以嵌套的。
*  `test`： 测试说明，写在`Jasmin`提供的`it`语句块中，以一个或者多个期望值结束（译者按：也就是说，一个it语句块中，一定要有一个以上的期望值）。
*  `actual`: 在期望中要被测试的值。
*  `expected value`: 针对测试出的真实值做比较的期望值。（原文：this is the value you test the actual value against.）
*  `matcher`: 一个返回值为Boolean类型的函数，用于比较真实值跟期望值。结果返回给jasmine，比如`toEqual`,`toBeGreatherThan`,`toHaveBeenCalledWith`... 你也可以定义自己的matcher。
*  `expectation`: 使用expect函数测试一个值，得到它的返回值，`expectation`是与一个得到期望值的`matcher函数`链接的。（原文：Use the expect function to test a value, called the actual. It is chained with a matcher function, which takes the expected value.）
*  `mock`:  一种「stubbed」（不会翻译）服务，你可以制造一些假数据或方法来替代程序真正运行时所产生的数据。

这有一个spec文件的例子：

```javascript   

// a test suite (group of tests)
//一组测试
describe('sample component test', function() {
    // a single test
    //单独的测试
    it('ensure addition is correct', function() {
        // sample expectation
        // 简单的期望
        expect(1+1).toEqual(2);
        //                  `--- the expected value (2) 期望值是2
        //             `--- the matcher method (equality) toEqual方法就是matcher函数
        //       `-- the actual value (2) 真实值是2
    });
    // another test
    // 另一个测试
    it('ensure substraction is correct', function() {
        expect(1-1).toEqual(0);
    });
});
```

------------

###测试环境搭建###

将grunt-karma添加到你项目的依赖中

```
npm install grunt-karma --save -dev
```

创建一个karma-unit.js文件

[这里是一个karma-unit文件的例子](https://github.com/revolunet/angular-stepper/blob/master/karma-unit.js)
这个文件定义了如下内容：
*  将要被加载到浏览器进行测试的JS文件。通常情况下，不仅项目用的库和项目本身的文件需要包含在内，你所要测试的文件和mock文件也要在这里加载。
*  想将测试运行在哪款浏览器中。
*  怎样接收到测试结果，是命令行里还是在浏览器中...？
*  可选插件。

以下是files这一项的例子：
```javascript
files: [
  "http://code.angularjs.org/1.2.1/angular.js",       <-- angular sourc
  "http://code.angularjs.org/1.2.1/angular-mocks.js", <-- angular mocks & test utils
  "src/angular-stepper.js",                           <-- our component source code
  "src/angular-stepper.spec.js"                       <-- our component test suite
]
```

注:这里可以添加jquery在里面，如果需要它帮助你编写测试代码(更强大的选择器,CSS测试,尺寸计算…)

将karma grunt tasks添加到Gruntfile.js中
```javascript
karma: {
    unit: {
        configFile: 'karma-unit.js',
        // run karma in the background
        background: true,
        // which browsers to run the tests on
        browsers: ['Chrome', 'Firefox']
    }
}
```

然后创建 `angular-stepper.spec.js`文件，将上面写的简单的测试代码粘贴进来。这时就可以轻松运行`grunt karma`任务去观察测试在浏览器中运行并且在命令行中生成测试报告。

```
....
Chrome 33.0.1712 (Mac OS X 10.9.0): Executed 2 of 2 SUCCESS (1.65 secs / 0.004 secs)
Firefox 25.0.0 (Mac OS X 10.9): Executed 2 of 2 SUCCESS (2.085 secs / 0.006 secs)
TOTAL: 4 SUCCESS
```
上面有四个点，每个点都代表一个成功的测试，这时可以看到，两个测试分别运行在我们配置的两个浏览器中了。
哦也~

那么接下来，让我们写一些真正的测试代码吧: )

---------------------

###给directive编写单元测试###

为组件所编写的一组单元测试，又叫做spec的东西，不仅应该覆盖所要测试的组件的所有预期行为，还要将边缘情况覆盖到（比如不合法的输入、服务器的异常状况）。

下面展示的`angular-stepper`组件的测试集的精华部分，[完整版点这里](https://github.com/revolunet/angular-stepper/blob/master/src/angular-stepper.spec.js)。对这样一个组件的测试非常简单，不需要假数据。唯一比较有技巧性的是，将的`directive`包含在了一个form表单下，这样能够在使用ngModelController和更新表单验证正确性的情况下正确的运行测试。(注：此处的内容需要读angular-stepper那个组件的文件才能懂为何要将directive包含在form表单中，如果不想深入了解，可以忽略这句。原文：The only tricky thing is that we wrap our directive inside a form to be able to test that it plays well with ngModelController and updates form validity correctly.)

```javascript

// the describe keyword is used to define a test suite (group of tests)
describe('rnStepper directive', function() {

    // we declare some global vars to be used in the tests
    var elm,        // our directive jqLite element
        scope;      // the scope where our directive is inserted

    // load the modules we want to test 在跑测试之前将要测试的模块引入进来
    beforeEach(module('revolunet.stepper'));

    // before each test, creates a new fresh scope 
    // the inject function interest is to make use of the angularJS 
    // dependency injection to get some other services in our test  inject方法的作用是利用angularJS的依赖注入将所需要的服务注入进去
    // here we need $rootScope to create a new scope 需要用$rootScope新建一个scope
    beforeEach(inject(function($rootScope, $compile) {
        scope = $rootScope.$new();
        scope.testModel = 42;
    }));

    function compileDirective(tpl) {
        // function to compile a fresh directive with the given template, or a default one
        // compile the tpl with the $rootScope created above
        // wrap our directive inside a form to be able to test
        // that our form integration works well (via ngModelController)
        // our directive instance is then put in the global 'elm' variable for further tests
        if (!tpl) tpl = '<div rn-stepper ng-model="testModel"></div></form>';
        tpl = '<form name="form">' + tpl + '</form>'; //原文最后一个标签是</tpl>感觉是笔误。
        // inject allows you to use AngularJS dependency injection
        // to retrieve and use other services
        inject(function($compile) {
            var form = $compile(tpl)(scope);
            elm = form.find('div');
        });
        // $digest is necessary to finalize the directive generation
        //$digest 方法对于生成指令是必要的。
        scope.$digest();
    }

    describe('initialisation', function() {
        // before each test in this block, generates a fresh directive
        beforeEach(function() {
            compileDirective();
        });
        // a single test example, check the produced DOM
        it('should produce 2 buttons and a div', function() {
            expect(elm.find('button').length).toEqual(2);
            expect(elm.find('div').length).toEqual(1);
        });
        it('should check validity on init', function() {
            expect(scope.form.$valid).toBeTruthy();
        });
    });

    it('should update form validity initialy', function() {
        // test with a min attribute that is out of bounds
        // first set the min value
        scope.testMin = 45;
        // then produce our directive using it
        compileDirective('<div rn-stepper min="testMin" ng-model="testModel"></div>');
        // this should impact the form validity
        expect(scope.form.$valid).toBeFalsy();
    });

    it('decrease button should be disabled when min reached', function() {
        // test the initial button status
        compileDirective('<div rn-stepper min="40" ng-model="testModel"></div>');
        expect(elm.find('button').attr('disabled')).not.toBeDefined();
        // update the scope model value
        scope.testModel = 40;
        // force model change propagation
        scope.$digest();
        // validate it has updated the button status
        expect(elm.find('button').attr('disabled')).toEqual('disabled');
    });
    // and many others...
});

```

 
一些需要注意的点：
 
在要被测试的scope中，一个directive需要被compiled(译者注：也就是上面代码中的`$compile(tpl)(scope);`这句话在做的事情）。
一个非隔离scope可以通过element.scope()方法访问到。
一个隔离的scope可以通过element.isolateScope()方法访问到。

####为啥在改变一个Model的值的时候需要调用scope.$digest()方法？####

在一个真正的angular应用中，\$digest方法是angular通过各种事件（click,inputs,requests...)的反应自动调用的。自动化测试不是以真实的用户事件为基础的，所以需要手动的调用\$digest方法（$digest方法负责更新所有数据绑定）。

--------------------------------

###额外福利 \#1: 实时测试###

多亏了grunt，当文件改动的时候，可以自动的进行测试。

如果你想在代码有任何改动的时候都进行一次测试，只要将一段代码加入到grunt的watch任务中就行。
```javascript
js: {
    files: ['src/*.js'],
    tasks: ['karma:unit:run', 'build']
},
```
你也可以将grunt的默认任务设置成这样：

```javascript
grunt.registerTask('default', ['karma:unit', 'connect', 'watch']);
```

设置完后，运行grunt，就可以实时的在内置的server中跑测试了。


----------------------------------

###额外福利  \#2:添加测试覆盖率报告 ###

作为开发者，我们希望以靠谱的数据作为依据，也希望持续的改进自己的代码。"coverage"指的是你的测试代码的覆盖率，它可以提供给你一些指标和详细的信息，无痛的增加代码的覆盖率。

下面是一个简易的覆盖率报告：

![覆盖率报告](http://blog.revolunet.com/images/coverage-example.png)


我们可以详细的看到每个文件夹的每个文件的代码是否被测试覆盖。归功于grunt+karma的集成，这个报告是实时更新的。我们可以在每一个文件中一行一行的检查哪一块代码没有被测试。这样能使测试变得更加的简单。
####100% test coverage 不代表你的代码就没有BUG了，但它代表这代码质量的提高！####

karma+grunt的集成特别的简单，karma有一套「插件」系统，它允许通过配置karma-unit.js文件来外挂`fantastic Istanbul 代码覆盖率检测工具`。只要配置一下文件，妈妈就再也不用担心我的代码覆盖率了。

####Add coverage to karma####

```javascript
# add the necessary node_modules
npm install karma-coverage --save-dev
```

现在将新的设置更新到kamar的配置文件中

```javascript
// here we specify which of the files we want to appear in the coverage report
preprocessors: {
    'src/angular-stepper.js': ['coverage']
},
// add the coverage plugin
plugins: [ 'karma-jasmine', 'karma-firefox-launcher', 'karma-chrome-launcher', 'karma-coverage'],
// add coverage to reporters
reporters: ['dots', 'coverage'],
// tell karma how you want the coverage results
coverageReporter: {
  type : 'html',
  // where to store the report
  dir : 'coverage/'
}
```


更多覆盖率的设置请看这里：[https://github.com/karma-runner/karma-coverage](https://github.com/karma-runner/karma-coverage)