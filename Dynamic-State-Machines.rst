动态状态机
============

原文： `Dynamic State Machines <http://harkablog.com/dynamic-state-machines.html>`_

译者： `youngsterxyf <http://xiayf.blogspot.com/>`_

昨天，我和同事 `Jeff <http://jeffelmore.org/>`_ 以及 Randy一起讨论Randy正在做的工作---在Django中实现一个有限状态机，然后我想起之前利用Python的动态类型实现过状态机。

在静态语言中，对象与其类型绑定，不能改变，那么要实现基于状态的行为就需要辅助性的类，通过在类之间的切换来实现不同状态的不同行为。Python的动态特征很自然地使这事简单得多。

使用动态类型实现状态
---------------------

Python中，一个对象的类型简单地决定于它的 ``__class__`` 属性的值。方便，但又令人相当震惊的是，这个属性是可变的。这意味着你可以通过简单地将一个新的类类型赋给 ``__class__`` 属性，就能改变一个对象的基本类型。

这种模式非常强大，也很容易被滥用，但是它能清晰高效地表达正在发生的事情，并且直接匹配我们的问题域。概念上，当一个对象改变状态，就成了一个不同类型的对象。实现上，当对象要改变状态，我们就改变对象的类型。如果我们想让不同的状态具有一些相同的行为，那么使用的类型可以是一个共同基类的子类，但这个是完全可选的。Python并不在意我们的状态是否相关。

在底层，当你试图访问某个对象的属性，Python首先查看对象自己是否有这个属性。如果有，就直接使用这个属性，如果没有，就检查对象所属的类，然后是超类，一直往上解析直到基类， ``object`` 。这个解释有些简单化了，忽略了Python赋予程序员的能力---使用 ``__getattr__`` 和 ``__getattribute__`` 来重载属性访问，但它对我们的目标来说够用了。

当对象改变状态时，它仍是那个对象，没有改变，具备以前所具备的属性，但是现在任何查找对象所属类的操作都会去查找新赋给对象的类类型，而不是在第一次创建对象时所使用的类类型。

.. image:: http://harkablog.com/static/images/morpho_menelaus.jpg

蝴蝶的生命周期
----------------

举例来说，我们来看看一个实现蝴蝶生命周期的状态机：

::

    class Egg(object):
        def __init__(self, species):
            self.species = species

        def hatch(self):
            self.__class__ = Caterpillar

    class Caterpillar(object):
        legs = 16
        def crawl(self): pass
        def eat(self): pass
        def pupate(self):
            self.__class__ = Pupa

    class Pupa(object):
        def emerge(self):
            self.__class__ = Butterfly

    class Butterfly(object):
        legs = 6
        def fly(self): pass
        def eat(self): pass
        def reproduce(self):
            return Egg(self.species)

对象生命开始时是一个 ``卵(Egg)`` ，孵化而成为一只 ``毛虫(Caterpillar)`` ，而后成为一只 ``蛹(Pupa)`` ，最后成为一只 ``蝴蝶(Butterfly)`` ，可以繁殖，创造新的 ``卵`` 。

实践过程看起来是这样的：

::

    >>> import buffterfly
    >>> critter = buffterfly.Egg('Morpho menelaus')
    >>> id(critter), type(critter), critter.species
    (10376583L, <class 'butterfly.Egg'>, 'Morpho menelaus')
    >>> hasattr(critter, 'legs')
    False


