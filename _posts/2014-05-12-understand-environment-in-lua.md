---
layout: post
title: "理解lua中的_G"
date:   2014-05-12
categories: Game-Dev
---

我觉得对一门语言的理解在于能否理解建立在思想之上的基础。也就是我们常说的“语言细节”，既是最基础的方面，但又是很容易忽略的地方。别人常说，当你使用一门语言时，那只是工具，当你理解一门语言时，那是思维和思想。那就得出：使用一门语言简单，造一个工具却很难。

lua是一门易于使用的语言，从接触lua这一两个月以来，最内心的感受是使用简单，想要理解它着实要花一番功夫。lua中有“环境”这个概念，所有的变量是基于“全局”的，究竟什么是lua的“环境”，lua的“环境”是什么？

lua中所有没有```local```修饰的变量和函数都是全局的，这些全局变量都放在```_G```这个变量中，这个变量是环境全局，也就是说整个lua运行环境都认识这个变量。

    home = "hubei";
    print(home);
    print(_G.home);
    print(_G["home"]);

这三种方式都正常输出```hubei```，lua在编译lua文件，把没有```local```修饰的变量默认放在_G这个table中，这个变量存储了当前lua运行中的所有全局变量，可以直接通过```_G.XX```访问。但是这个不是我今天的主题，今天的主题是```require```。

文件lua1.lua  

    function Fun1()
    	print("Fun1");
    end

文件lua2.lua  

    require("lua1");
    Fun1();

正常输出```Fun1```。对于这个过程，我是比较好奇的。```require```是加载模块所用的函数，它自动将模块加载入运行环境，也就是说当```require```之后，```Fun1```实际上是存在于```_G```变量中的。当调用```Fun1();```时，lua环境会去寻找这个函数，首先会在```_G```中找到这个函数，然后调用之。同样，可以用```_G.Fun1()```得到同样的效果。

require机制实际上是先按照```LUA_PATH```路径查找模块，如果没有就查找模块同名文件，这里就加载了lua1.lua文件。而```require```是将文件作为全局加载的，返回一个是否加载成功的boolean值。显然这种方式在实际用途中还是有一定局限性的，一旦脚本文件很多的时，那么```_G```将会有非常多的变量，我们的原则是尽量在```_G```添加需要的东西，其余的东西不需要，简化脚本的加载。

可以先试想一下理想中的脚本加载方式。lua中到处都充斥着table的概念，那么实际上可以把一个脚本里的内容完全当作一个table，然后按照正常的方式去访问table里的东西。假如我们在lua2.lua文件中调用Fun1，可以这样子：

    lua1.Fun1();

每个脚本文件享有自己的```_G```，而不是lua环境的全局```_G```，而当需要```_G```时，我们可以添加进去我们自己想要的东西：

    _G.Fun1 = function ()
    end
    -- 或者
    function Fun1()
    end

    rawset(_G, "Fun1", Fun1);

lua 5.1之后引入模块的概念，可以把一个脚本文件当作一个模块加载入环境，而环境享有自己的全局变量。  
文件lua1.lua  

    module("lua1", package.seeall);
    
    Home = "hubei";
    function Fun1()
    	print("Fun1");
    end
    
    rawset(_G, "Fun1", Fun1);

文件lua2.lua  

    local lua1 = require("lua1");
    lua1.Fun1();
    _G.Fun1();
    _G["Fun1"]();

如果在lua1.lua文件中不加上```rawset(_G, "Fun1", Fun1);```在lua2.lua利用```_G```调用时会出现找不到值的错误，也就是说在```module```之后，lua1.lua中的所有变量和函数不会在加入```_G```变量中了，而是加入了模块```lua1```中了。这样就明白游戏脚本里对于```Import```的设计了。