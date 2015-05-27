---
layout: post
title: angularjs五个最棒的特点
description: 简单的说：angularjs 是全新的，功能强大的客户端技术，提供一种强大的构建动态WEB应用设计 结构框架，通过扩展HTML的语法，让你能更清楚、简洁地构建你的应用组件。它克服HTML在构建应用上的不足而设计的。HTML是一门很好的为静态文本展示设计的声明式语言，但要构建WEB应用的话它就显得乏力了。
category: blog
---
**简单的说：angularjs 是全新的，功能强大的客户端技术，提供一种强大的构建动态WEB应用设计 结构框架，通过扩展HTML的语法，让你能更清楚、简洁地构建你的应用组件。它克服HTML在构建应用上的不足而设计的。HTML是一门很好的为静态文本展示设计的声明式语言，但要构建WEB应用的话它就显得乏力了。**


###特点1： 双向数据绑定
数据绑定是angularjs中最酷，最实用的功能。这将节省你相当数量的代码。典型的Web应用程序可能包含其80%的代码，专门用于遍历、操作和监听DOM。数据绑定使这些代码消失，这样可以专注于你应用程序中的业务实现。

在应用程序中，只要将Model做为唯一对象关注就可以了，进行更新和读取。而数据绑定指令提供将Model到View的映射，这个映射是无缝的，无需应用程序做任何的操作。只要当Model发生变化后，View也会立马变更。反之，当View变更后，Model也会随之更新，这个是双向作用，并且是无缝结合。

传统上，当Model发生变化后，开发者需要手工操作DOM节点和属性来反映出其变化。反之，视图上的任何变化也无法自动的作用于Model上。如图所示：  

![Alt text](http://docs.angularjs.org/img/One_Way_Data_Binding.png)

angularjs 双向数据绑定改变这一切，让操作变化如此简单：

![Alt text](http://docs.angularjs.org/img/Two_Way_Data_Binding.png)

<iframe width="100%" height="300" src="http://jsfiddle.net/chopperkuang/xJzhG/9/embedded/" frameborder="0"> </iframe>

![Alt text](http://devgirl.org/wp-content/uploads/2013/03/concepts-controller.png)

###特点2： 模板
在angularjs中，模板只是普通的HTML，这个HTML模板会被浏览器解析成DOM，并且angularjs会将该DOM进行编译。
  
在angularjs遍历这个DOM模板进行渲染称之为指令，总这，这些指令负责数据-视图的绑定。负责“连续地”更新，不需要模板和数据的重新合并.  

我们需要重要认识到的是，angularjs操作是字符串的模板，但却是使用DOM进行输入，而不是HTML字符串。数据绑定就是DOM结构的转换，而不是字符串的拼接和通过innerHTML进行将视图变化。使用DOM的输入，而不是字符串。  

angularjs 最大的差异在于，还有其他的子框架，允许开发者进行扩展，自定义指令，这样就可以封装成可重用的组件。  

这种最大的优点在于：使设计者和开发者更好的合作，设计者通过HTML切成页面，而开发者就在此基础上进行少量变更就能将数据进行展示，无需太大的工作。  

上述例子中的ng-repeat指令就是将多条数据进行循环展示。  

另外一个好处，就是无需学习新的模板语言。  

###特点3： MVC
angularjs是结合最基本的MVC设计模式构建客户端WEB应用。但不是传统意义上的MVC，而是MVVM（Model - View - ViewModel）。
#### Model
Model就是应用程序中的数据，一些普通的javascript对象，无需继续某类，或者包裹在代理类中，或者通过getter/setter访问。这是非常棒的一种体验，减少了应用程序的复杂度。

#### ViewModel
一个视图模板对象，它提供具体的数据和方法关联到对应的视图上。
viewmodel在angularjs中可以理解为$scope对象。$scope就是一个简单的javascript对象，这是很精简的API。当对象发现变化时，广播和通知相应的善状态。

#### Controller
该Controller负责初始化状态和添加$scope的方法控制行为，但值得一提的是，控制器并不存储状态，在多个控制器状态不共享。
详细的控制器介绍，可以看这篇中文文档：[AngularJS开发指南17：Controller组件](http://www.angularjs.cn/A00C)

#### View

###特点4： 依赖注入
angularjs创建一套依赖注入方式，帮助开发者在构建系统时更易开发，理解和测试。
依赖注入(DI)，你只要"喊"出你的依赖，而不需要去寻找和构建依赖对象。就如 “嗨，我需要一把斧子”，DI就会帮你制造一把斧子并且提供给你。
{% highlight javascript %}
function EditCtrl($scope, $location, $routeParams) {
     // Something clever here...
}
{% endhighlight %}
你也可以构建自己的服务，注入到需要的地方。
{% highlight javascript %}
angular.
    module('MyServiceModule', []).
    factory('notify', ['$window', function (win) {
    	return function (msg) {
        	win.alert(msg);
	    };
	}
]);

function myController($scope, notify) {
    $scope.callNotify = function (msg) {
        notify(msg);
    };
}

{% endhighlight %}


###特点5： 指令
angularjs指令是我非常喜欢的一个功能点。你有没有想过，让您的浏览器会做新的技巧呢？好了，现在就可以了！  
这是最喜欢angularjs其中之一，这也可能是最具有挑战性的。

指令可用来创建我自定义的HTML标签，作为新的自定义组件。它们也可以用来“装饰”元素上的行为，并以有趣的方式进行DOM操作。

下面一个简单的例子，一个监听事件，并且更新Model
{% highlight javascript %}
myModule.directive('myComponent', function(mySharedService) {
    return {
        restrict: 'E',
        controller: function($scope, $attrs, mySharedService) {
            $scope.$on('handleBroadcast', function() {
                $scope.message = 'Directive: ' + mySharedService.message;
            });
        },
        replace: true,
        template: '<input>'
    };
});
{% endhighlight %}
然后，你可以像这样使用指令：
{% highlight html %}
<my-component ng-model="message"></my-component>
{% endhighlight %}
拥有该特征后，你可以将应用程序按不同的组件编写，然后可以随意添加、删除及更新。very cool.


###奖励特点： 路由、过滤器、测试
未完待续。。。


以上我只是简单的介绍，详细可以看[angularjs官网的教程](http://angularjs.org/)，也有不错的[angularjs中文版](http://www.angularjs.cn/)。
