---
title: Inheriting methods' docstrings in Python
categories: 
  - [Python, motosport]
tags: 
  - Python
---

This is a variation on Paul McGuire's DocStringInheritor metaclass.

1. It inherits a parent member's docstring if the child member's docstring is empty.
2. It inherits a parent class docstring if the child class docstring is empty.
3. It can inherit the docstring from any class in any of the base classes's MROs, just like regular attribute inheritance.
4. Unlike with a class decorator, the metaclass is inherited, so you only need to set the metaclass once in some top-level base class, and docstring inheritance will occur throughout your OOP hierarchy.

```python
import unittest
import sys

class DocStringInheritor(type):
    """
    A variation on
    http://groups.google.com/group/comp.lang.python/msg/26f7b4fcb4d66c95
    by Paul McGuire
    """
    def __new__(meta, name, bases, clsdict):
        if not('__doc__' in clsdict and clsdict['__doc__']):
            for mro_cls in (mro_cls for base in bases for mro_cls in base.mro()):
                doc=mro_cls.__doc__
                if doc:
                    clsdict['__doc__']=doc
                    break
        for attr, attribute in clsdict.items():
            if not attribute.__doc__:
                for mro_cls in (mro_cls for base in bases for mro_cls in base.mro()
                                if hasattr(mro_cls, attr)):
                    doc=getattr(getattr(mro_cls,attr),'__doc__')
                    if doc:
                        if isinstance(attribute, property):
                            clsdict[attr] = property(attribute.fget, attribute.fset, 
                                                     attribute.fdel, doc)
                        else:
                            attribute.__doc__ = doc
                        break
        return type.__new__(meta, name, bases, clsdict)



class Test(unittest.TestCase):

    def test_null(self):
        class Foo(object):

            def frobnicate(self): pass

        class Bar(Foo, metaclass=DocStringInheritor):
            pass

        self.assertEqual(Bar.__doc__, object.__doc__)
        self.assertEqual(Bar().__doc__, object.__doc__)
        self.assertEqual(Bar.frobnicate.__doc__, None)

    def test_inherit_from_parent(self):
        class Foo(object):
            'Foo'

            def frobnicate(self):
                'Frobnicate this gonk.'
        class Bar(Foo, metaclass=DocStringInheritor):
            pass
        self.assertEqual(Foo.__doc__, 'Foo')
        self.assertEqual(Foo().__doc__, 'Foo')
        self.assertEqual(Bar.__doc__, 'Foo')
        self.assertEqual(Bar().__doc__, 'Foo')
        self.assertEqual(Bar.frobnicate.__doc__, 'Frobnicate this gonk.')

    def test_inherit_from_mro(self):
        class Foo(object):
            'Foo'

            def frobnicate(self):
                'Frobnicate this gonk.'
        class Bar(Foo):
            pass

        class Baz(Bar, metaclass=DocStringInheritor):
            pass

        self.assertEqual(Baz.__doc__, 'Foo')
        self.assertEqual(Baz().__doc__, 'Foo')
        self.assertEqual(Baz.frobnicate.__doc__, 'Frobnicate this gonk.')

    def test_inherit_metaclass_(self):
        class Foo(object):
            'Foo'

            def frobnicate(self):
                'Frobnicate this gonk.'
        class Bar(Foo, metaclass=DocStringInheritor):
            pass

        class Baz(Bar):
            pass
        self.assertEqual(Baz.__doc__, 'Foo')
        self.assertEqual(Baz().__doc__, 'Foo')
        self.assertEqual(Baz.frobnicate.__doc__, 'Frobnicate this gonk.')

    def test_property(self):
        class Foo(object):
            @property
            def frobnicate(self): 
                'Frobnicate this gonk.'
        class Bar(Foo, metaclass=DocStringInheritor):
            @property
            def frobnicate(self): pass

        self.assertEqual(Bar.frobnicate.__doc__, 'Frobnicate this gonk.')


if __name__ == '__main__':
    sys.argv.insert(1, '--verbose')
    unittest.main(argv=sys.argv)
```

---

This version supports multiple inheritance and copying the documentation from a base's base by using __mro__ instead of __bases__.

```python
def fix_docs(cls):
    """
    This will copy all the missing documentation for methods from the parent classes.

    :param type cls: class to fix up.
    :return type: the fixed class.
    """
    for name, func in vars(cls).items():
        if isinstance(func, types.FunctionType) and not func.__doc__:
            for parent in cls.__bases__:
                parfunc = getattr(parent, name, None)
                if parfunc and getattr(parfunc, '__doc__', None):
                    func.__doc__ = parfunc.__doc__
                    break
        elif isinstance(func, property) and not func.fget.__doc__:
            for parent in cls.__bases__:
                parprop = getattr(parent, name, None)
                if parprop and getattr(parprop.fget, '__doc__', None):
                    newprop = property(fget=func.fget,
                                       fset=func.fset,
                                       fdel=func.fdel,
                                       parprop.fget.__doc__)
                    setattr(cls, name, newprop)
                    break

    return cls
```

Test:
```python
import pytest


class X(object):

    def please_implement(self):
        """
        I have a very thorough documentation
        :return:
        """
        raise NotImplementedError

    @property
    def speed(self):
        """
        Current speed in knots/hour.
        :return:
        """
        return 0

    @speed.setter
    def speed(self, value):
        """

        :param value:
        :return:
        """
        pass


class SpecialX(X):

    def please_implement(self):
        return True

    @property
    def speed(self):
        return 10

    @speed.setter
    def speed(self, value):
        self.sp = value


class VerySpecial(X):

    def speed(self):
        """
        The fastest speed in knots/hour.
        :return: 100
        """
        return 100

    def please_implement(self):
        """
        I have my own words!
        :return bool: Always false.
        """
        return False

    def not_inherited(self):
        """
        Look at all these words!
        :return:
        """


class A(object):

    def please_implement(self):
        """
        This doc is not used because X is resolved first in the MRO.
        :return:
        """
        pass


class B(A):
    pass


class HasNoWords(SpecialX, B):
    def please_implement(self):
        return True

    @property
    def speed(self):
        return 10

    @speed.setter
    def speed(self, value):
        self.sp = value


def test_class_does_not_inhirit_works():
    fix_docs(X)


@pytest.mark.parametrize('clazz', [
    SpecialX,
    HasNoWords
])
def test_property_and_method_inherit(clazz):
    x = fix_docs(clazz)
    assert x.please_implement.__doc__ == """
        I have a very thorough documentation
        :return:
        """

    assert x.speed.__doc__ == """
        Current speed in knots/hour.
        :return:
        """


def test_inherited_class_with_own_doc_is_not_overwritten():
    x = fix_docs(VerySpecial)
    assert x.please_implement.__doc__ == """
        I have my own words!
        :return bool: Always false.
        """

    assert x.speed.__doc__ == """
        The fastest speed in knots/hour.
        :return: 100
        """
```

---

