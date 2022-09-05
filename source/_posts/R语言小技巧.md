---
title: R语言小技巧
date: 2021-05-20 11:54:50
tags: "R"
categories: 
- ["R"]
- ["devel"]
---

# R 语言使用记录

## list

- 使用"names()"方法查看list中的成员
- 列表的一个元素也可以称为列表的一个“变量”， 单个列表元素必须用两重方括号格式访问
- 列表的单个元素也可以用$格式访问
- 列表的单个元素也可以用$格式访问

## packages

- 已加载的包 :: (.packages())
- 卸除加载包 :: detach("package:name")
- 删除安装包 :: remove.packages(c(p1, p2, ...), lib = file.path("path", "to", "library"))

# Lua 

## Lua中的面向对象
对象由属性和方法组成。LUA中最基本的结构是table，所以需要用table来描述对象的属性。
lua 中的 function 可以用来表示方法。那么LUA中的类可以通过 table + function 模拟出来。
至于继承，可以通过 metetable 模拟出来（不推荐用，只模拟最基本的对象大部分时间够用了）。

使用点号(.)来访问类的属性，使用冒号 : 来访问类的成员函数。

    -- 元类
    Shape = {area = 0}
    
    -- 基础类方法 new
    function Shape:new (o,side)
      o = o or {}
      setmetatable(o, self)
      self.__index = self
      side = side or 0
      self.area = side*side;
      return o
    end
    
    -- 基础类方法 printArea
    function Shape:printArea ()
      print("面积为 ",self.area)
    end
    
    -- 创建对象
    myshape = Shape:new(nil,10)
    
    myshape:printArea()

# python

## load python module or class dynamically

### Using the __import__ Magic Method
	
    ########################################################################
    class DynamicImporter:
	""""""

        #----------------------------------------------------------------------
        def __init__(self, module_name, class_name):
	        """Constructor"""
	        module = __import__(module_name)
	        my_class = getattr(module, class_name)
	        instance = my_class()
	        print instance
	                
	if __name__ == "__main__":
	    DynamicImporter("decimal", "Context")
	    
        
### Using Python’s imp Module

    import imp
    import sys
    #----------------------------------------------------------------------
    def dynamic_importer(name, class_name):
        """
        Dynamically imports modules / classes
        """
        try:
            fp, pathname, description = imp.find_module(name)
        except ImportError:
            print "unable to locate module: " + name
            return (None, None)
        
        try:
            example_package = imp.load_module(name, fp, pathname, description)
        except Exception, e:
            print e
            
        try:
            myclass = imp.load_module("%s.%s" % (name, class_name), fp, pathname, description)
            print myclass
        except Exception, e:
            print e
            
        return example_package, myclass
        
    if __name__ == "__main__":
        module, modClass = dynamic_importer("decimal", "Context")
