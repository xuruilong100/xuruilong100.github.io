---
title: QuantLib 金融计算——自己动手封装 Python 接口（1）
date: 2019-12-16 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: []
tags: [QuantLib]
description: 介绍如何为 QuantLib 封装 Python 接口。
---

> 由于版本问题，代码可能与最新版不兼容。

# QuantLib 金融计算——自己动手封装 Python 接口（1）

## 概述

QuantLib 已经开始在 [PyPi](https://pypi.org/project/QuantLib-Python/) 上发布封装好的 Python 接口，安装和使用非常方便，与普通的包别无二致。并且更新及时，保持对应最新版本的 QuantLib。

官方发布的 Python 接口，其优点是**广度**和**全面**，缺点是**深度不足**。有时候用户需要的功能恰好没有被封装（[《收益率曲线之构建曲线（3）》](https://xuruilong100.github.io/posts/QuantLib-%E9%87%91%E8%9E%8D%E8%AE%A1%E7%AE%97%E6%94%B6%E7%9B%8A%E7%8E%87%E6%9B%B2%E7%BA%BF%E4%B9%8B%E6%9E%84%E5%BB%BA%E6%9B%B2%E7%BA%BF-3/)一文中曾经提到过），希望重新封装接口，添加自己需要的功能；亦或是用户已经在 C++ 源代码层面上扩展或修复了 QuantLib，希望包装[扩展的新功能](https://github.com/xuruilong100/QuantLibEx)，并与官方的 Python 接口联合使用。

无论是上述哪种情况，都需要用户自己动手封装 Python 接口。

## QuantLib 如何封装 Python 接口？

QuantLib 使用 [swig](https://www.swig.org/) 来封装 Python 接口（其他语言的接口也是用 swig 封装的），所以，要动手封装自己的 Python 接口需要了解一点 swig 的用法（看[这里](https://github.com/xuruilong100/SWIG3DocCn)，或[这里](https://xuruilong100.github.io/tags/swig3-%E4%B8%AD%E6%96%87%E6%89%8B%E5%86%8C/)）。

swig 封装 C++ 的流程大体如下：
1. 编写若干“接口文件”（文件扩展名是 `.i` ），告知 swig 如何封装 C++ 源代码；
2. 在接口文件上运行 swig 命令，这会产生一个 `.py` 文件（描述封装好的 Python 接口，包含了若干函数或类的定义），以及一个 `.cpp` 文件（接口背后的计算引擎将由该文件生成）；
3. 编写并运行 `setup.py`，这将编译 `.cpp` 文件，并将编译得到的 `.so` 文件（动态库）与 `.py` 文件绑定起来，贯通表面的 Python 接口和背后的 C++ 计算引擎；
4. 最终得到一个 Python 包（将包含在系统目录）。

不同版本 QuantLib 的 swig 接口文件可以在[这里](https://bintray.com/quantlib/releases/QuantLib-SWIG)获得。所有接口文件可以分为三部分：
1. `quantlib.i` 是最顶端的接口文件，swig 将依据此文件生成接口代码（`.py` 和 `.cpp`）；
2. `ql.i` 是中间层文件，用来汇集其他接口文件；
3. `bonds.i`、`date.i` 等等是封装具体接口的文件。

## 自己封装 Python 接口

了解一点 swig 的原理之后会发现，swig 在封装好的 Python 接口背后隐藏了一个个真实的 C++ 对象，实际的计算任务、类型检查和异常处理等等其实是委托给这些 C++ 对象。

因此可以猜测，将同一段 C++ 代码封装成两个不同的 Python 接口，这两个接口应该可以混用，因为这仅仅是“同一个人穿了不同的衣服”。

下面用实验验证这种想法。

### 封装 `Array` 和 `Matrix` 类

以 QuantLib 中的两个类 `Array` 和 `Matrix` 为例，将它们独立出来，封装成名为 QuantLibEx 的包。具体的接口文件没有必要自己写，直接沿用官方发布的版本（我的名言：*学习，从模仿开始*）。

在官方发布的 swig 接口文件中，`Array` 和 `Matrix` 对应的文件是 `linearalgebra.i`，该文件同时包含（`%include`）了 `common.i` 和 `types.i` 两个文件。

将上述三个文件连同 `quantlib.i`（重命名为 `quantlibex.i`）和 `ql.i` 独立出来，删除掉一些和封装 Python 接口无关的代码，作为构建 QuantLibEx 的接口文件。

在包含这五个接口文件的目录下创建一个 `QuantLibEx` 目录，然后运行 swig 命令，生成必需的 `.py` 和 `.cpp` 文件：

```
swig -c++ -python -outdir QuantLibEx -o QuantLibEx/qlx_wrap.cpp quantlibex.i
```

`QuantLibEx` 目录下将出现两个文件：`QuantLibEx.py` 和 `qlx_wrap.cpp`。为了使 QuantLibEx 成为一个 Python 包，需要添加一个 `__init__.py` 文件（内容见**附录**）。

`QuantLibEx.py` 和 `qlx_wrap.cpp` 准备就绪之后就可以运行事先编写好的 `setup.py` 文件（内容见**附录**），编译 `.cpp` 文件，并打包进 Python 的系统目录。

首先，构建（`build` 命令）QuantLibEx 包：

```
python3 setup.py build
```

```
running build
running build_py
creating build
creating build/lib.linux-x86_64-3.6
creating build/lib.linux-x86_64-3.6/QuantLibEx
copying QuantLibEx/__init__.py -> build/lib.linux-x86_64-3.6/QuantLibEx
copying QuantLibEx/QuantLibEx.py -> build/lib.linux-x86_64-3.6/QuantLibEx
running build_ext
building 'QuantLibEx._QuantLibEx' extension
creating build/temp.linux-x86_64-3.6
creating build/temp.linux-x86_64-3.6/QuantLibEx
x86_64-linux-gnu-gcc -pthread -DNDEBUG -g -fwrapv -O2 -Wall -g -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -fPIC -I/usr/include/ql -I/usr/include/python3.6m -c QuantLibEx/qlx_wrap.cpp -o build/temp.linux-x86_64-3.6/QuantLibEx/qlx_wrap.o
x86_64-linux-gnu-g++ -pthread -shared -Wl,-O1 -Wl,-Bsymbolic-functions -Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-Bsymbolic-functions -Wl,-z,relro -g -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 build/temp.linux-x86_64-3.6/QuantLibEx/qlx_wrap.o -L/usr/lib/ -lQuantLib -o build/lib.linux-x86_64-3.6/QuantLibEx/_QuantLibEx.cpython-36m-x86_64-linux-gnu.so
```

构建成功之后会出现一个 `build` 目录，里面包含了若干文件，包括已经构建好的 QuantLibEx 包。然后，进入安装（`install` 命令）环节，打包进 Python 的系统目录（需要 sudo 权限）。

```
sudo python3 setup.py install
```

```
running install
running build
running build_py
running build_ext
running install_lib
copying build/lib.linux-x86_64-3.6/QuantLibEx/QuantLibEx.py -> /usr/local/lib/python3.6/dist-packages/QuantLibEx
copying build/lib.linux-x86_64-3.6/QuantLibEx/_QuantLibEx.cpython-36m-x86_64-linux-gnu.so -> /usr/local/lib/python3.6/dist-packages/QuantLibEx
byte-compiling /usr/local/lib/python3.6/dist-packages/QuantLibEx/QuantLibEx.py to QuantLibEx.cpython-36.pyc
running install_egg_info
Writing /usr/local/lib/python3.6/dist-packages/QuantLibEx-0.1.egg-info
```

运行 `pip3 list` 就可以看到 QuantLibEx 了。

```
...
QuantLib                1.17
QuantLibEx              0.1
...
```

### QuantLibEx 和官方包混合使用

下面简单验证一下 QuantLibEx（基于 QuantLib-1.15）和官方包（基于 QuantLib-1.17）是否可以混合使用：

```python
import QuantLib as ql
import QuantLibEx as qlx

array = ql.Array(5,0.2)
print(type(array))
print(array)

arrayX = qlx.Array(5,0.3)
print(type(arrayX))
print(arrayX)

print(array + arrayX)
```

```
<class 'QuantLib.QuantLib.Array'>
[ 0.2; 0.2; 0.2; 0.2; 0.2 ]
<class 'QuantLibEx.QuantLibEx.Array'>
[ 0.3; 0.3; 0.3; 0.3; 0.3 ]
[ 0.5; 0.5; 0.5; 0.5; 0.5 ]
```

更复杂一点的例子——二维插值：

```python
xVec = [float(i) for i in range(10)]
yVec = [float(i) for i in range(10)]

m = ql.Matrix(len(xVec), len(yVec))
mX = qlx.Matrix(len(xVec), len(yVec))

for rowIt in range(len(xVec)):
    for colIt in range(len(yVec)):
        m[rowIt][colIt] = scipy.sin(xVec[rowIt]) + scipy.sin(yVec[colIt])
        mX[rowIt][colIt] = scipy.sin(xVec[rowIt]) + scipy.sin(yVec[colIt])

print(type(m))
print(m)

print(type(mX))
print(mX)

bicubIntp = ql.BicubicSpline(
    xVec, yVec, m)

bicubIntpX = ql.BicubicSpline(
    xVec, yVec, mX)

x = 0.5
y = 4.5

print("Analytical Value:  ", scipy.sin(x) + scipy.sin(y))
print("Bicubic Value(base on ql):  ", bicubIntp(x, y))
print("Bicubic Value(base on qlx):  ", bicubIntpX(x, y))
```

```
<class 'QuantLib.QuantLib.Matrix'>
| 0 0.841471 0.909297 0.14112 -0.756802 -0.958924 -0.279415 0.656987 0.989358 0.412118 |
| 0.841471 1.68294 1.75077 0.982591 0.0846685 -0.117453 0.562055 1.49846 1.83083 1.25359 |
| 0.909297 1.75077 1.81859 1.05042 0.152495 -0.0496268 0.629882 1.56628 1.89866 1.32142 |
| 0.14112 0.982591 1.05042 0.28224 -0.615682 -0.817804 -0.138295 0.798107 1.13048 0.553238 |
| -0.756802 0.0846685 0.152495 -0.615682 -1.5136 -1.71573 -1.03622 -0.0998159 0.232556 -0.344684 |
| -0.958924 -0.117453 -0.0496268 -0.817804 -1.71573 -1.91785 -1.23834 -0.301938 0.030434 -0.546806 |
| -0.279415 0.562055 0.629882 -0.138295 -1.03622 -1.23834 -0.558831 0.377571 0.709943 0.132703 |
| 0.656987 1.49846 1.56628 0.798107 -0.0998159 -0.301938 0.377571 1.31397 1.64634 1.06911 |
| 0.989358 1.83083 1.89866 1.13048 0.232556 0.030434 0.709943 1.64634 1.97872 1.40148 |
| 0.412118 1.25359 1.32142 0.553238 -0.344684 -0.546806 0.132703 1.06911 1.40148 0.824237 |

<class 'QuantLibEx.QuantLibEx.Matrix'>
| 0 0.841471 0.909297 0.14112 -0.756802 -0.958924 -0.279415 0.656987 0.989358 0.412118 |
| 0.841471 1.68294 1.75077 0.982591 0.0846685 -0.117453 0.562055 1.49846 1.83083 1.25359 |
| 0.909297 1.75077 1.81859 1.05042 0.152495 -0.0496268 0.629882 1.56628 1.89866 1.32142 |
| 0.14112 0.982591 1.05042 0.28224 -0.615682 -0.817804 -0.138295 0.798107 1.13048 0.553238 |
| -0.756802 0.0846685 0.152495 -0.615682 -1.5136 -1.71573 -1.03622 -0.0998159 0.232556 -0.344684 |
| -0.958924 -0.117453 -0.0496268 -0.817804 -1.71573 -1.91785 -1.23834 -0.301938 0.030434 -0.546806 |
| -0.279415 0.562055 0.629882 -0.138295 -1.03622 -1.23834 -0.558831 0.377571 0.709943 0.132703 |
| 0.656987 1.49846 1.56628 0.798107 -0.0998159 -0.301938 0.377571 1.31397 1.64634 1.06911 |
| 0.989358 1.83083 1.89866 1.13048 0.232556 0.030434 0.709943 1.64634 1.97872 1.40148 |
| 0.412118 1.25359 1.32142 0.553238 -0.344684 -0.546806 0.132703 1.06911 1.40148 0.824237 |

Analytical Value:   -0.498104579060894
Bicubic Value(base on ql):   -0.49656170664824184
Bicubic Value(base on qlx):   -0.49656170664824184
```

到目前为止，一切都能按照预期运行，自定义的封装确实能够和官方发布的包混合使用。不过，类型判定有些**诡异**：

```python
b = array + arrayX
c = arrayX + array

print(type(b))
print(type(c))
```

```
<class 'QuantLibEx.QuantLibEx.Array'>
<class 'QuantLibEx.QuantLibEx.Array'>
```

为什么 `b` 和 `c` 都被判定为 QuantLibEx 中的 `Array`？

![](/img/meme/confuse.jpg)

## 附录：接口文件、`setup.py` 和 `__init__.py`

### `quantlibex.i`

```
%module QuantLibEx

%include exception.i

%exception {
    try {
        $action
    } catch (std::out_of_range& e) {
        SWIG_exception(SWIG_IndexError,const_cast<char*>(e.what()));
    } catch (std::exception& e) {
        SWIG_exception(SWIG_RuntimeError,const_cast<char*>(e.what()));
    } catch (...) {
        SWIG_exception(SWIG_UnknownError,"unknown error");
    }
}

//#if defined(SWIGPYTHON)
%{
#include <ql/version.hpp>
const int    __hexversion__ = QL_HEX_VERSION;
const char* __version__    = QL_VERSION;
%}

const int __hexversion__;
%immutable;
const char* __version__;
%mutable;
//#endif

%include ql.i
```

### `ql.i`

```
//#if defined(SWIGPYTHON)
%{
#ifdef barrier
#undef barrier
#endif
%}
//#endif

%{
#include <ql/quantlib.hpp>

#if QL_HEX_VERSION < 0x011400f0
    #error using an old version of QuantLib, please update
#endif

#ifdef BOOST_MSVC
#ifdef QL_ENABLE_THREAD_SAFE_OBSERVER_PATTERN
#define BOOST_LIB_NAME boost_thread
#include <boost/config/auto_link.hpp>
#undef BOOST_LIB_NAME
#define BOOST_LIB_NAME boost_system
#include <boost/config/auto_link.hpp>
#undef BOOST_LIB_NAME
#endif
#endif

// add here SWIG version check

%}

//#ifdef SWIGPYTHON
%{
#if PY_VERSION_HEX < 0x02010000
    #error Python version 2.1.0 or later is required
#endif
%}
//#endif

// common name mappings

%include common.i
%include linearalgebra.i
%include types.i
```

### `types.i`

```
#ifndef quantlib_types_i
#define quantlib_types_i

%include common.i
%include std_common.i

%{
using QuantLib::Integer;
using QuantLib::BigInteger;
using QuantLib::Natural;
using QuantLib::BigNatural;
using QuantLib::Real;
using QuantLib::Decimal;
using QuantLib::Time;
using QuantLib::Rate;
using QuantLib::Spread;
using QuantLib::DiscountFactor;
using QuantLib::Volatility;
using QuantLib::Probability;
using QuantLib::Size;
%}

typedef int Integer;
typedef long BigInteger;
typedef unsigned int Natural;
typedef unsigned long BigNatural;
typedef double Real;

typedef Real Decimal;
typedef Real Time;
typedef Real Rate;
typedef Real Spread;
typedef Real DiscountFactor;
typedef Real Volatility;
typedef Real Probability;

//#if defined(SWIGPYTHON)
// needed for those using SWIG 1.3.21 in order to compile with VC++6
%typecheck(SWIG_TYPECHECK_INTEGER) std::size_t {
    $1 = (PyInt_Check($input) || PyLong_Check($input)) ? 1 : 0;
}
//#endif

typedef std::size_t Size;

#endif
```

### `common.i`

```
#ifndef quantlib_common_i
#define quantlib_common_i

%include stl.i
%include exception.i

%define QL_TYPECHECK_BOOL       7210    %enddef

%{
// This is necessary to avoid compile failures on
// GCC 4
// see http://svn.boost.org/trac/boost/ticket/1793

#if defined(NDEBUG)
#define BOOST_DISABLE_ASSERTS 1
#endif

#include <boost/algorithm/string/case_conv.hpp>
%}

//#if defined(SWIGPYTHON)
%typemap(in) boost::optional<bool> %{
	if($input == Py_None)
		$1 = boost::none;
	else if ($input == Py_True)
		$1 = true;
	else
		$1 = false;
%}
%typecheck (QL_TYPECHECK_BOOL) boost::optional<bool> {
if (PyBool_Check($input) || Py_None == $input)
	$1 = 1;
else
	$1 = 0;
}
//#endif

%{
// generally useful classes
using QuantLib::Error;
using QuantLib::Handle;
using QuantLib::RelinkableHandle;
%}

namespace boost {

    template <class T>
    class shared_ptr {
      public:
        T* operator->();
        //#if defined(SWIGPYTHON)
        %extend {
            bool __nonzero__() {
                return !!(*self);
            }
            bool __bool__() {
                return !!(*self);
            }
        }
        //#endif
    };

}

template <class T>
class Handle {
  public:
    Handle(const boost::shared_ptr<T>& = boost::shared_ptr<T>());
    boost::shared_ptr<T> operator->();
    //#if defined(SWIGPYTHON)
    %extend {
        bool __nonzero__() {
            return !self->empty();
        }
        bool __bool__() {
            return !self->empty();
        }
    }
    //#endif
};

template <class T>
class RelinkableHandle : public Handle<T> {
  public:
    RelinkableHandle(const boost::shared_ptr<T>& = boost::shared_ptr<T>());
    void linkTo(const boost::shared_ptr<T>&);
};

%define swigr_list_converter(ContainerRType,
                            ContainerCType, ElemCType)
%enddef

%define deprecate_feature(OldName, NewName)
//#if defined(SWIGPYTHON)
%pythoncode %{
def OldName(*args, **kwargs):
    from warnings import warn
    warn('%s is deprecated; use %s' % (OldName.__name__, NewName.__name__))
    return NewName(*args, **kwargs)
%}
//#endif
%enddef

#endif
```

### `linearalgebra.i`

```
#ifndef quantlib_linear_algebra_i
#define quantlib_linear_algebra_i

%include common.i
%include types.i
%include stl.i

%{
using QuantLib::Array;
using QuantLib::Matrix;
%}

%define QL_TYPECHECK_ARRAY       4210    %enddef
%define QL_TYPECHECK_MATRIX      4220    %enddef

//#if defined(SWIGPYTHON)
%{
bool extractArray(PyObject* source, Array* target) {
    if (PyTuple_Check(source) || PyList_Check(source)) {
        Size size = (PyTuple_Check(source) ?
                     PyTuple_Size(source) :
                     PyList_Size(source));
        *target = Array(size);
        for (Size i=0; i<size; i++) {
            PyObject* o = PySequence_GetItem(source,i);
            if (PyFloat_Check(o)) {
                (*target)[i] = PyFloat_AsDouble(o);
                Py_DECREF(o);
            } else if (PyInt_Check(o)) {
                (*target)[i] = Real(PyInt_AsLong(o));
                Py_DECREF(o);
            } else {
                Py_DECREF(o);
                return false;
            }
        }
        return true;
    } else {
        return false;
    }
}
%}

%typemap(in) Array (Array* v) {
    if (extractArray($input,&$1)) {
        ;
    } else {
        SWIG_ConvertPtr($input,(void **) &v, $&1_descriptor,1);
        $1 = *v;
    }
};
%typemap(in) const Array& (Array temp) {
    if (extractArray($input,&temp)) {
        $1 = &temp;
    } else {
        SWIG_ConvertPtr($input,(void **) &$1,$1_descriptor,1);
    }
};
%typecheck(QL_TYPECHECK_ARRAY) Array {
    /* native sequence? */
    if (PyTuple_Check($input) || PyList_Check($input)) {
        Size size = PySequence_Size($input);
        if (size == 0) {
            $1 = 1;
        } else {
            PyObject* o = PySequence_GetItem($input,0);
            if (PyNumber_Check(o))
                $1 = 1;
            else
                $1 = 0;
            Py_DECREF(o);
        }
    } else {
        /* wrapped Array? */
        Array* v;
        if (SWIG_ConvertPtr($input,(void **) &v,
                            $&1_descriptor,0) != -1)
            $1 = 1;
        else
            $1 = 0;
    }
}
%typecheck(QL_TYPECHECK_ARRAY) const Array & {
    /* native sequence? */
    if (PyTuple_Check($input) || PyList_Check($input)) {
        Size size = PySequence_Size($input);
        if (size == 0) {
            $1 = 1;
        } else {
            PyObject* o = PySequence_GetItem($input,0);
            if (PyNumber_Check(o))
                $1 = 1;
            else
                $1 = 0;
            Py_DECREF(o);
        }
    } else {
        /* wrapped Array? */
        Array* v;
        if (SWIG_ConvertPtr($input,(void **) &v,
                            $1_descriptor,0) != -1)
            $1 = 1;
        else
            $1 = 0;
    }
}

%typemap(in) Matrix (Matrix* m) {
    if (PyTuple_Check($input) || PyList_Check($input)) {
        Size rows, cols;
        rows = (PyTuple_Check($input) ?
                PyTuple_Size($input) :
                PyList_Size($input));
        if (rows > 0) {
            // look ahead
            PyObject* o = PySequence_GetItem($input,0);
            if (PyTuple_Check(o) || PyList_Check(o)) {
                cols = (PyTuple_Check(o) ?
                        PyTuple_Size(o) :
                        PyList_Size(o));
                Py_DECREF(o);
            } else {
                PyErr_SetString(PyExc_TypeError, "Matrix expected");
                Py_DECREF(o);
                return NULL;
            }
        } else {
            cols = 0;
        }
        $1 = Matrix(rows,cols);
        for (Size i=0; i<rows; i++) {
            PyObject* o = PySequence_GetItem($input,i);
            if (PyTuple_Check(o) || PyList_Check(o)) {
                Size items = (PyTuple_Check(o) ?
                                        PyTuple_Size(o) :
                                        PyList_Size(o));
                if (items != cols) {
                    PyErr_SetString(PyExc_TypeError,
                        "Matrix must have equal-length rows");
                    Py_DECREF(o);
                    return NULL;
                }
                for (Size j=0; j<cols; j++) {
                    PyObject* d = PySequence_GetItem(o,j);
                    if (PyFloat_Check(d)) {
                        $1[i][j] = PyFloat_AsDouble(d);
                        Py_DECREF(d);
                    } else if (PyInt_Check(d)) {
                        $1[i][j] = Real(PyInt_AsLong(d));
                        Py_DECREF(d);
                    } else {
                        PyErr_SetString(PyExc_TypeError,"doubles expected");
                        Py_DECREF(d);
                        Py_DECREF(o);
                        return NULL;
                    }
                }
                Py_DECREF(o);
            } else {
                PyErr_SetString(PyExc_TypeError, "Matrix expected");
                Py_DECREF(o);
                return NULL;
            }
        }
    } else {
        SWIG_ConvertPtr($input,(void **) &m,$&1_descriptor,1);
        $1 = *m;
    }
};
%typemap(in) const Matrix & (Matrix temp) {
    if (PyTuple_Check($input) || PyList_Check($input)) {
        Size rows, cols;
        rows = (PyTuple_Check($input) ?
                PyTuple_Size($input) :
                PyList_Size($input));
        if (rows > 0) {
            // look ahead
            PyObject* o = PySequence_GetItem($input,0);
            if (PyTuple_Check(o) || PyList_Check(o)) {
                cols = (PyTuple_Check(o) ?
                        PyTuple_Size(o) :
                        PyList_Size(o));
                Py_DECREF(o);
            } else {
                PyErr_SetString(PyExc_TypeError, "Matrix expected");
                Py_DECREF(o);
                return NULL;
            }
        } else {
            cols = 0;
        }

        temp = Matrix(rows,cols);
        for (Size i=0; i<rows; i++) {
            PyObject* o = PySequence_GetItem($input,i);
            if (PyTuple_Check(o) || PyList_Check(o)) {
                Size items = (PyTuple_Check(o) ?
                                        PyTuple_Size(o) :
                                        PyList_Size(o));
                if (items != cols) {
                    PyErr_SetString(PyExc_TypeError,
                        "Matrix must have equal-length rows");
                    Py_DECREF(o);
                    return NULL;
                }
                for (Size j=0; j<cols; j++) {
                    PyObject* d = PySequence_GetItem(o,j);
                    if (PyFloat_Check(d)) {
                        temp[i][j] = PyFloat_AsDouble(d);
                        Py_DECREF(d);
                    } else if (PyInt_Check(d)) {
                        temp[i][j] = Real(PyInt_AsLong(d));
                        Py_DECREF(d);
                    } else {
                        PyErr_SetString(PyExc_TypeError,"doubles expected");
                        Py_DECREF(d);
                        Py_DECREF(o);
                        return NULL;
                    }
                }
                Py_DECREF(o);
            } else {
                PyErr_SetString(PyExc_TypeError, "Matrix expected");
                Py_DECREF(o);
                return NULL;
            }
        }
        $1 = &temp;
    } else {
        SWIG_ConvertPtr($input,(void **) &$1,$1_descriptor,1);
    }
};
%typecheck(QL_TYPECHECK_MATRIX) Matrix {
    /* native sequence? */
    if (PyTuple_Check($input) || PyList_Check($input)) {
        $1 = 1;
    /* wrapped Matrix? */
    } else {
        Matrix* m;
        if (SWIG_ConvertPtr($input,(void **) &m,
                            $&1_descriptor,0) != -1)
            $1 = 1;
        else
            $1 = 0;
    }
}
%typecheck(QL_TYPECHECK_MATRIX) const Matrix & {
    /* native sequence? */
    if (PyTuple_Check($input) || PyList_Check($input)) {
        $1 = 1;
    /* wrapped Matrix? */
    } else {
        Matrix* m;
        if (SWIG_ConvertPtr($input,(void **) &m,
                            $1_descriptor,0) != -1)
            $1 = 1;
        else
            $1 = 0;
    }
}
//#endif

class Array {
    //#if defined(SWIGPYTHON) || defined(SWIGRUBY)
    %rename(__len__)   size;
    //#endif
  public:
    Array();
    Array(Size n, Real fill = 0.0);
    Array(const Array&);
    Size size() const;
    %extend {
        std::string __str__() {
            std::ostringstream out;
            out << *self;
            return out.str();
        }
        //#if defined(SWIGPYTHON) || defined(SWIGRUBY) || defined(SWIGR)
        Array __add__(const Array& a) {
            return Array(*self+a);
        }
        Array __sub__(const Array& a) {
            return Array(*self-a);
        }
        Array __mul__(Real a) {
            return Array(*self*a);
        }
        Real __mul__(const Array& a) {
            return QuantLib::DotProduct(*self,a);
        }
        Array __mul__(const Matrix& a) {
            return *self*a;
        }
        Array __div__(Real a) {
            return Array(*self/a);
        }
        //#endif
        //#if defined(SWIGPYTHON)
        Array __rmul__(Real a) {
            return Array(*self*a);
        }
        Array __getslice__(Integer i, Integer j) {
            Integer size_ = static_cast<Integer>(self->size());
            if (i<0)
                i = size_+i;
            if (j<0)
                j = size_+j;
            i = std::max(0,i);
            j = std::min(size_,j);
            Array tmp(j-i);
            std::copy(self->begin()+i,self->begin()+j,tmp.begin());
            return tmp;
        }
        void __setslice__(Integer i, Integer j, const Array& rhs) {
            Integer size_ = static_cast<Integer>(self->size());
            if (i<0)
                i = size_+i;
            if (j<0)
                j = size_+j;
            i = std::max(0,i);
            j = std::min(size_,j);
            QL_ENSURE(static_cast<Integer>(rhs.size()) == j-i,
                      "arrays are not resizable");
            std::copy(rhs.begin(),rhs.end(),self->begin()+i);
        }
        bool __nonzero__() {
            return (self->size() != 0);
        }
        bool __bool__() {
            return (self->size() != 0);
        }
        //#endif
        //#if defined(SWIGPYTHON) || defined(SWIGRUBY)
        Real __getitem__(Integer i) {
            Integer size_ = static_cast<Integer>(self->size());
            if (i>=0 && i<size_) {
                return (*self)[i];
            } else if (i<0 && -i<=size_) {
                return (*self)[size_+i];
            } else {
                throw std::out_of_range("array index out of range");
            }
        }
        void __setitem__(Integer i, Real x) {
            Integer size_ = static_cast<Integer>(self->size());
            if (i>=0 && i<size_) {
                (*self)[i] = x;
            } else if (i<0 && -i<=size_) {
                (*self)[size_+i] = x;
            } else {
                throw std::out_of_range("array index out of range");
            }
        }
        //#endif
    }
};

// 2-D view

%{
typedef QuantLib::LexicographicalView<Array::iterator> DefaultLexicographicalView;
typedef QuantLib::LexicographicalView<Array::iterator>::y_iterator DefaultLexicographicalViewColumn;
%}

//#if defined(SWIGPYTHON) || defined(SWIGRUBY) || defined(SWIGR)
class DefaultLexicographicalViewColumn {
  private:
    // access control - no constructor exported
    DefaultLexicographicalViewColumn();
  public:
    %extend {
        Real __getitem__(Size i) {
            return (*self)[i];
        }
        void __setitem__(Size i, Real x) {
            (*self)[i] = x;
        }
    }
};
//#endif

%rename(LexicographicalView) DefaultLexicographicalView;
class DefaultLexicographicalView {
  public:
    Size xSize() const;
    Size ySize() const;
    %extend {
        DefaultLexicographicalView(Array& a, Size xSize) {
            return new DefaultLexicographicalView(a.begin(),a.end(),xSize);
        }
        std::string __str__() {
            std::ostringstream s;
            for (Size j=0; j<self->ySize(); j++) {
                s << "\n";
                for (Size i=0; i<self->xSize(); i++) {
                    if (i != 0)
                        s << ",";
                    Array::value_type value = (*self)[i][j];
                    s << value;
                }
            }
            s << "\n";
            return s.str();
        }
        //#if defined(SWIGPYTHON) || defined(SWIGRUBY) || defined(SWIGR)
        DefaultLexicographicalViewColumn __getitem__(Size i) {
            return (*self)[i];
        }
        //#endif
    }
};

%{
typedef QuantLib::Matrix::row_iterator MatrixRow;
using QuantLib::outerProduct;
using QuantLib::transpose;
using QuantLib::SVD;
%}

//#if defined(SWIGPYTHON) || defined(SWIGRUBY)
class MatrixRow {
  private:
    MatrixRow();
  public:
    %extend {
        Real __getitem__(Size i) {
            return (*self)[i];
        }
        void __setitem__(Size i, Real x) {
            (*self)[i] = x;
        }
    }
};
//#endif

class Matrix {
  public:
    Matrix();
    Matrix(Size rows, Size columns, Real fill = 0.0);
    Matrix(const Matrix&);
    Size rows() const;
    Size columns() const;
    %extend {
        std::string __str__() {
            std::ostringstream out;
            out << *self;
            return out.str();
        }
        //#if defined(SWIGPYTHON) || defined(SWIGRUBY)
        Matrix __add__(const Matrix& m) {
            return *self+m;
        }
        Matrix __sub__(const Matrix& m) {
            return *self-m;
        }
        Matrix __mul__(Real x) {
            return *self*x;
        }
        Array __mul__(const Array& x) {
            return *self*x;
        }
        Matrix __mul__(const Matrix& x) {
            return *self*x;
        }
        Matrix __div__(Real x) {
            return *self/x;
        }
        //#endif
        //#if defined(SWIGPYTHON) || defined(SWIGRUBY)
        MatrixRow __getitem__(Size i) {
            return (*self)[i];
        }
        //#endif
        //#if defined(SWIGPYTHON)
        Matrix __rmul__(Real x) {
            return x*(*self);
        }
        Array __rmul__(const Array& x) {
            return x*(*self);
        }
        Matrix __rmul__(const Matrix& x) {
            return x*(*self);
        }
        //#endif
    }
};

// functions

%{
using QuantLib::pseudoSqrt;
using QuantLib::SalvagingAlgorithm;
%}

struct SalvagingAlgorithm {
    //#if defined(SWIGPYTHON)
    %rename(NoAlgorithm) None;
    //#endif
    enum Type { None, Spectral };
};

Matrix transpose(const Matrix& m);
Matrix outerProduct(const Array& v1, const Array& v2);
Matrix pseudoSqrt(const Matrix& m, SalvagingAlgorithm::Type a);

class SVD {
  public:
    SVD(const Matrix&);
    const Matrix& U() const;
    const Matrix& V() const;
    Matrix S() const;
    const Array& singularValues() const;
};

#endif
```

### `setup.py`

```python
"""
setup.py file for QuantLibEx
"""

from distutils.core import setup, Extension

qlx_module = Extension(
    name='QuantLibEx._QuantLibEx',
    sources=['QuantLibEx/qlx_wrap.cpp'],
    include_dirs=['/usr/include/ql'], # QuantLib 头文件所在的目录
    library_dirs=['/usr/lib/'],       # QuantLib 库所在的目录
    libraries=['QuantLib']            # QuantLib 库的名字
    )

setup(
    name        = 'QuantLibEx',
    version     = '0.1',
    author      = "xrl",
    description = "Python bindings for the QuantLibEx library",
    ext_modules = [qlx_module],
    py_modules  = ['QuantLibEx.__init__','QuantLibEx.QuantLibEx'])
```

### `__init__.py`

```python
import sys
if sys.version_info.major >= 3:
    from .QuantLibEx import *
    from .QuantLibEx import _QuantLibEx
else:
    from QuantLibEx import *
    from QuantLibEx import _QuantLibEx
del sys

__author__ = 'xrl'

if hasattr(_QuantLibEx,'__version__'):
    __version__ = _QuantLibEx.__version__
elif hasattr(_QuantLibEx.cvar,'__version__'):
    __version__ = _QuantLibEx.cvar.__version__
else:
    print('Could not find __version__ attribute')

if hasattr(_QuantLibEx,'__hexversion__'):
    __hexversion__ = _QuantLibEx.__hexversion__
elif hasattr(_QuantLibEx.cvar,'__hexversion__'):
    __hexversion__ = _QuantLibEx.cvar.__hexversion__
else:
    print('Could not find __hexversion__ attribute')

__license__ = """
QuantLibEx ...
"""
```
