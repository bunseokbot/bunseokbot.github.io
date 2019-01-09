---
title: 파이썬이 거짓말을 했다
description: 파이썬에서 작은 정수를 처리할 때 신기한 현상
categories:
 - python
tags: python, programming, tricks
---

## 이상하다

어느 날 코드를 짜던 중 이상하게 오류가 나는 것을 발견했다.

```python
>>> a = 1
>>> b = 1
>>> id(a) == id(b)
True
>>> a = 1000
>>> b = 1000
>>> id(a) == id(b)
False
```

엥? 왜 같은 정수인데.. 이상하다.. 이상해 (파이썬이 거짓말을 하고 있는 것 같다)



## 과연 진짜 거짓말을 하고 있는 것일까

답은 생각보다 [간단한 곳](https://docs.python.org/3/c-api/long.html#c.PyLong_FromLong)에서(?) 찾을 수 있었다.

```r
The current implementation keeps an array of integer objects for all integers between -5 and 256, when you create an int in that range you actually just get back a reference to the existing object. So it should be possible to change the value of 1. I suspect the behaviour of Python in this case is undefined. :-)
```

간단하게 말하면 -5부터 256 사이에 있는 모든 정수는 미리 선언되어 있고, 이 범위 안에서 사용자가 열심히 선언해도 파이썬은 그냥 선언되어 있는 것을 가져다 쓴다는 개념이다.

그래서 우리는 이런 과정을 통해서 아래와 같은 결과를 얻을 수 있다.

```python
>>> a = 1
>>> b = 1
>>> c = 1
>>> d = 1
>>> e = 1
>>> f = 1
>>> a == b == c == d == e == f
True
```

일종의 파이썬은 캐싱을 하고 있는 것이다. 실제 선언하는 것은 쓸데없는 메모리를 많이 사용하게 되니깐..

일단 이렇게 일단락..



## 안에서는 어떻게 구현되어 있을까?

짓기에는 너무 흥미로웠다. 그냥 내부를 좀 까보고 싶은 생각이 더 컸지만..

그래서 만물의 탄생 근원인 [cpython repository](https://github.com/python/cpython) 에 가서 해당 함수의 구현부를 검색해 보았다.

(여기 아래에 있는 모든 코드는 [해당 링크](https://github.com/python/cpython/blob/b509d52083e156f97d6bd36f2f894a052e960f03/Objects/longobject.c)를 참고하였습니다.)

일단 PyObject의 longobject.c 라는 파일에서 `PyLong_FromLong` 이라는 함수를 살펴봅시다. (이 친구가 시작이니깐요)

```c
/* Create a new int object from a C long int */

PyObject *
PyLong_FromLong(long ival)
{
    PyLongObject *v;
    unsigned long abs_ival;
    unsigned long t;  /* unsigned so >> doesn't propagate sign bit */
    int ndigits = 0;
    int sign;

    CHECK_SMALL_INT(ival);

    if (ival < 0) {
        /* negate: can't write this as abs_ival = -ival since that
           invokes undefined behaviour when ival is LONG_MIN */
        abs_ival = 0U-(unsigned long)ival;
        sign = -1;
    }
  ...
}
```

여기 보면 변수 선언하고 일을 하기 전에 `CHECK_SMALL_INT` 이라는 함수를 호출합니다.

자, 여기서 CHECK_SMALL_INT 이라는 함수(정확히는 매크로 함수입니다)로 가보면 이러한 것을 찾을 수 있습니다.

```c
#define CHECK_SMALL_INT(ival)
    do if (-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS) {
        return get_small_int((sdigit)ival);
    } while(0)
```

매개변수로 받은 ival의 값이 -NSMALLNEGINTS보다 크거나 같고, NSMALLPOSINTS보다 작은 경우 `get_small_int` 함수를 다시 호출합니다.

여기서 NSMALLNEGINTS와 NSMALLPOSINTS는 이미 정의되어 있습니다. 어디에?

```c
#ifndef NSMALLPOSINTS
#define NSMALLPOSINTS           257
#endif
#ifndef NSMALLNEGINTS
#define NSMALLNEGINTS           5
#endif
```

즉, 이를 통해서 -5부터 256까지(257은 포함하지 않습니다) 의 값이 들어오게 된다면 `get_small_int` 라는 함수를 호출하게 됩니다.

그럼.. 이 함수는 뭘 하냐..?

```c
static PyObject *
get_small_int(sdigit ival)
{
    PyObject *v;
    assert(-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS);
    v = (PyObject *)&small_ints[ival + NSMALLNEGINTS];
    Py_INCREF(v);
#ifdef COUNT_ALLOCS
    if (ival >= 0)
        _Py_quick_int_allocs++;
    else
        _Py_quick_neg_int_allocs++;
#endif
    return v;
}
```

이미 선언되어 있는 `small_ints` 이라는 배열에 있는 값을 불러와서 반한해주는 아주 심플한 함수입니다.

근데 그럼 이 `small_ints` 이라는 배열의 값은 언제 세팅될까요..? (찾아보면 됩니다)

```c
int
_PyLong_Init(void)
{
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    int ival, size;
    PyLongObject *v = small_ints;

    for (ival = -NSMALLNEGINTS; ival <  NSMALLPOSINTS; ival++, v++) {
        size = (ival < 0) ? -1 : ((ival == 0) ? 0 : 1);
        if (Py_TYPE(v) == &PyLong_Type) {
            /* The element is already initialized, most likely
             * the Python interpreter was initialized before.
             */
            Py_ssize_t refcnt;
            PyObject* op = (PyObject*)v;

            refcnt = Py_REFCNT(op) < 0 ? 0 : Py_REFCNT(op);
            _Py_NewReference(op);
            /* _Py_NewReference sets the ref count to 1 but
             * the ref count might be larger. Set the refcnt
             * to the original refcnt + 1 */
            Py_REFCNT(op) = refcnt + 1;
            assert(Py_SIZE(op) == size);
            assert(v->ob_digit[0] == (digit)abs(ival));
        } else {
            (void)PyObject_INIT(v, &PyLong_Type);
        }
        Py_SIZE(v) = size;
        v->ob_digit[0] = (digit)abs(ival);
    }
    ...
}
```

바로 `_PyLong_Init` 이 함수, 초기화할 때 세팅을 뚝딱 해줍니다.

-5부터 256까지 값을 돌면서 object를 initializing 해줍니다. 이를 통해서 우리는 -5부터 256까지의 id값을 가진 `small_ints` 를 만들게 되는 것 이지요.

결국, 이를 통해서 정리하자면 아래와 같은 과정을 통해서 이렇게 미리 Integer 값을 caching 하는 것을 알 수 있습니다.

1. PyLong 타입이 Initialing될 때 -5 ~ 256까지의 id값을 미리 `small_ints` 라는 배열에 저장한다.
2. 만약 새롭게 사용자가 Integer object를 선언하거나 수정된 값이 해당 범위 내라면 새로 할당하는게 아니라 기존에 할당된 id값을 준다.
3. 이러면 사람들은 작은 정수의 변수가 선언 되더라도 실제로 새로 할당되지 않으므로 메모리를 넓게 쓸 수 있다(?)



## 컴퓨터는 거짓말을 하지 않는다

이것은 과학이였다.

컴퓨터는 거짓말을 하지 않았다.

느낀점: 이상하다 싶으면 Reference를 뒤져보자. 그럼 fact가 나올 것이다.