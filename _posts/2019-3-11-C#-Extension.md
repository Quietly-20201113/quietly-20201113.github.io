---
layout: post
title:  "C#-Extension-合集"
date:   2019-03-17 22:39
categories: C# Extension
tags: C#
---

------

一、去重扩展**Distinct**  

```c#
 /// <summary>
    /// Distinct扩展
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <typeparam name="V"></typeparam>
    public class CommonEqualityComparer<T, V> : IEqualityComparer<T>
    {
        private Func<T, V> keySelector;
        private IEqualityComparer<V> comparer;

        public CommonEqualityComparer(Func<T, V> keySelector, IEqualityComparer<V> comparer)
        {
            this.keySelector = keySelector;
            this.comparer = comparer;
        }

        public CommonEqualityComparer(Func<T, V> keySelector): this(keySelector, EqualityComparer<V>.Default)
        { }

        public bool Equals(T x, T y)
        {
            return comparer.Equals(keySelector(x), keySelector(y));
        }

        public int GetHashCode(T obj)
        {
            return comparer.GetHashCode(keySelector(obj));
        }
    }
```

1、引用  

```c#
   /// <summary>
   /// 根据条件去重(去重复之前先select，不然会以类的Key来去重)
   /// </summary>
   /// <typeparam name="T"></typeparam>
   /// <typeparam name="V"></typeparam>
   /// <param name="source">去重匿名对象</param>
   /// <param name="keySelector"></param>
   /// <returns></returns>      
public static IEnumerable<T> TyDistinct<T, V>(this IEnumerable<T> source, Func<T, V> keySelector)
        {
            return source.Distinct(new CommonEqualityComparer<T, V>(keySelector));
        }
```

> 使用时，可根据多字段去重  
>
> 问题：排序时，使用到了默认的**Distinct**，会默认根据id去重，需要根据别的字段来去重需重新写个匿名类型。

***

二、if条件扩展  

```C# 
        /// <summary>
        /// if条件扩展
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="obj"></param>
        /// <param name="values"></param>
        /// <returns></returns>
        public static bool In<T>(this T obj, params T[] values)
        {
            return values.Contains(obj);
        }
```

***

三、字符串匹配算法  (不是很准，写在这以后看看)

```c#
 /// <summary>
        /// 字符串匹配算法扩展
        /// </summary>
        /// <param name="s">待匹配字符串</param>
        /// <param name="t">匹配字符串</param>
        /// <returns></returns>
        public static int KmpIndexOf(this string s, string t)
        {
            int i = 0, j = 0, v;
            int[] nextVal = GetNextVal(t);

            while (i < s.Length && j < t.Length)
            {
                if (j == -1 || s[i] == t[j])
                {
                    i++;
                    j++;
                }
                else
                {
                    j = nextVal[j];
                }
            }

            if (j >= t.Length)
                v = i - t.Length;
            else
                v = -1;

            return v;
        }

        /// <summary>
        /// 字符串拆分匹配
        /// </summary>
        /// <param name="t"></param>
        /// <returns></returns>
        private static int[] GetNextVal(string t)
        {
            int j = 0, k = -1;
            int[] nextVal = new int[t.Length];

            nextVal[0] = -1;

            while (j < t.Length - 1)
            {
                if (k == -1 || t[j] == t[k])
                {
                    j++;
                    k++;
                    if (t[j] != t[k])
                    {
                        nextVal[j] = k;
                    }
                    else
                    {
                        nextVal[j] = nextVal[k];
                    }
                }
                else
                {
                    k = nextVal[k];
                }
            }

            return nextVal;
        }
```

***

