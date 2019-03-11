---
layout: post
title:  "C#-分页PagedList源码"
date:   2019-03-05 20:56
categories: C#
tags: C# 分页PagedList
---
* content
{:toc}
------

**剽窃技术总监框架**  

C#的一个分页，排序集合类  

1.方法扩展（扩展类中）

```java
        /// <summary>
        /// 返回IQueryble分页数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="linq"></param>
        /// <param name="pageIndex">页号</param>
        /// <param name="pageSize">每页行数</param>
        /// <returns></returns>
        public static PagedList<T> ToPagedList<T>(this IQueryable<T> linq, int pageIndex, int pageSize)
        {
            return new PagedList<T>(linq, pageIndex, pageSize);
        }


        /// <summary>
        /// 返回Iqueryble分页数据（默认每页18条数据）
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="linq">IQueryble对像</param>
        /// <param name="pageIndex">页号</param>
        /// <returns></returns>
        public static PagedList<T> ToPagedList<T>(this IQueryable<T> linq, int pageIndex)
        {
            int pageSize = 30;
            return new PagedList<T>(linq, pageIndex, pageSize);
        }

        public static List<T> ToPagedList<T>(this IQueryable<T> linq, PageCtl pcl)
        {
            return new PagedList<T>(linq, pcl).ToList();
        }

        public static List<T> ToPagedList<T>(this IEnumerable<T> linq, PageCtl pcl)
        {
            return new PagedList<T>(linq, pcl).ToList();
        }

        public static IQueryable ToPagedList(this IQueryable source, PageCtl pageCtl)
        {
            if (pageCtl == null)
            {
                return source;
            }

            if (pageCtl.PageSize == 0) pageCtl.PageSize = 30;
            if (pageCtl.PageIndex == 0) pageCtl.PageIndex = 1;


            int total = source.Count();
            pageCtl.TotalCount = total;
            pageCtl.TotalPages = total / pageCtl.PageSize;


            if (total % pageCtl.PageSize > 0)
                pageCtl.TotalPages++;

            string OrderByField = pageCtl.OrderByField;
            bool Ascending = !pageCtl.Dscending;

            Type type = source.ElementType;
            if (string.IsNullOrEmpty(OrderByField))
            {
                PropertyInfo[] propertys = type.GetProperties();
                if (propertys.Length == 0)
                    throw new Exception("当前实体未包含属性！");
                var pIs = from c in propertys where c.GetCustomAttributes(false).Count() > 0 select c;
                var pI = from c in pIs where c.CustomAttributes.Where(w => w.AttributeType == typeof(System.ComponentModel.DataAnnotations.KeyAttribute)).Count() > 0 select c;
                if (pI.Count() > 0)
                {
                    OrderByField = pI.First().Name;
                }
                else
                {
                    OrderByField = propertys[0].Name;
                }
            }

            PropertyInfo property = type.GetProperty(OrderByField);
            if (property == null)
                throw new Exception("错误：\r\n当前实体中不存在指定的排序字段：" + OrderByField);

            ParameterExpression param = Expression.Parameter(type, "p");
            Expression propertyAccessExpression = Expression.MakeMemberAccess(param, property);
            LambdaExpression orderByExpression = Expression.Lambda(propertyAccessExpression, param);

            string methodName = Ascending ? "OrderBy" : "OrderByDescending";

            MethodCallExpression resultExp = Expression.Call(
                typeof(Queryable), methodName, new Type[] { type, property.PropertyType },
            source.Expression,
            Expression.Quote(orderByExpression)
            );

            var query = source.Provider.CreateQuery(resultExp);

            var PageSize = pageCtl.PageSize;
            if (pageCtl.PageIndex > pageCtl.TotalPages)
            {
                pageCtl.PageIndex = pageCtl.TotalPages;
            }
            if (pageCtl.PageIndex < 1)
            {
                pageCtl.PageIndex = 1;
            }
            //  this.PageIndex = pageCtl.PageIndex;
            //pageCtl.TotalPages = this.TotalPages;
            //pageCtl.TotalCount = this.TotalCount;

            query = query.Skip((pageCtl.PageIndex - 1) * pageCtl.PageSize).Take(pageCtl.PageSize);
            return query;
        }
```

2.页面类  

```java
'VB代码
Public Class PageCtl
    ''' <summary>
    ''' 当前页
    ''' </summary>
    Public Property PageIndex() As Integer
    ''' <summary>
    ''' 每页数据大小
    ''' </summary>
    Public Property PageSize() As Integer
    ''' <summary>
    ''' 总页数
    ''' </summary>
    Public Property TotalPages() As Integer
    ''' <summary>
    ''' 总记录数
    ''' </summary>
    Public Property TotalCount() As Integer
    ''' <summary>
    ''' 排序字段
    ''' </summary>
    Public Property OrderByField() As String
    ''' <summary>
    ''' 是否倒序
    ''' </summary>
    Public Property Dscending() As Boolean

End Class

Public Class PageCtl(Of T)
    Inherits PageCtl
    Property list As List(Of T)
End Class
///上面代码等同于C#代码
public class PageCtl<T>where T : class
{
     /// <summary>
    /// 当前页
    /// </summary>
    public int PageIndex { get; set; }
    /// <summary>
    /// 每页数据大小
    /// </summary>
    public int PageSize { get; set; }
    /// <summary>
    /// 总页数
    /// </summary>
    public int TotalPages { get; set; }
    /// <summary>
    /// 总记录数
    /// </summary>
    public int TotalCount { get; set; }
    /// <summary>
    /// 排序字段
    /// </summary>
    public string OrderByField { get; set; }
    /// <summary>
    /// 是否倒序
    /// </summary>
    public bool? Dscending { get; set; }
}

```

3.分页通用类  

```java
using CommonClass;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Linq.Expressions;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;

namespace ServiceCommon
{
    /// <summary>
    /// 分页通用类
    /// </summary>
    /// <typeparam name="T"></typeparam>
    public class PagedList<T> : List<T>, IPagedList
    {
        /// <summary>
        /// 数据源为IQueryable的范型
        /// </summary>
        /// <param name="source">数据源</param>
        /// <param name="index">当前页</param>
        /// <param name="pageSize">每页显示多少条记录</param>
        public PagedList(IQueryable<T> source, int index, int pageSize)
        {
            if (source != null) //判断传过来的实体集是否为空
            {
                int total = source.Count();
                this.TotalCount = total;
                this.TotalPages = total / pageSize;

                if (total % pageSize > 0)
                    TotalPages++;

                this.PageSize = pageSize;
                if (index > this.TotalPages)
                {
                    index = this.TotalPages;
                }
                if (index < 1)
                {
                    index = 1;
                }
                this.PageIndex = index;
                this.AddRange(source.Skip((index - 1) * pageSize).Take(pageSize).ToList()); //Skip是跳到第几页，Take返回多少条
            }
        }


        public PagedList(IQueryable<T> source, PageCtl pageCtl)
        {
            if (pageCtl == null)
            {
                this.AddRange(source.ToList());
                this.ToList();
                return;
            }

            if (pageCtl.PageSize == 0) pageCtl.PageSize = 30;
            if (pageCtl.PageIndex == 0) pageCtl.PageIndex = 1;


            int total = source.Count();
            this.TotalCount = total;
            this.TotalPages = total / pageCtl.PageSize;

            if (total % pageCtl.PageSize > 0)
                TotalPages++;

            string OrderByField = pageCtl.OrderByField;
            bool Ascending = !pageCtl.Dscending;

            Type type = typeof(T);
            if (string.IsNullOrEmpty(OrderByField))
            {
                PropertyInfo[] propertys = type.GetProperties();
                if (propertys.Length == 0)
                    throw new Exception("当前实体未包含属性！");
                //OrderByField = propertys[0].Name;
                foreach (var item in propertys.Where(x => x.GetCustomAttributes(false).Count() > 0))
                {
                    if (item.CustomAttributes.Where(x => x.AttributeType == new System.ComponentModel.DataAnnotations.KeyAttribute().GetType()).FirstOrDefault() != null)
                    {
                        OrderByField = item.Name;
                        break;
                    }
                }
            }

            PropertyInfo property = type.GetProperty(OrderByField);
            if (property == null)
                throw new Exception("错误：\r\n当前实体中不存在指定的排序字段：" + OrderByField);


            ParameterExpression param = Expression.Parameter(type, "p");
            Expression propertyAccessExpression = Expression.MakeMemberAccess(param, property);
            LambdaExpression orderByExpression = Expression.Lambda(propertyAccessExpression, param);

            string methodName = Ascending ? "OrderBy" : "OrderByDescending";

            MethodCallExpression resultExp = Expression.Call(
                typeof(Queryable), methodName, new Type[] { type, property.PropertyType },
            source.Expression,
            Expression.Quote(orderByExpression)
            );

            var query = source.Provider.CreateQuery<T>(resultExp);



            this.PageSize = pageCtl.PageSize;
            if (pageCtl.PageIndex > this.TotalPages)
            {
                pageCtl.PageIndex = this.TotalPages;
            }
            if (pageCtl.PageIndex < 1)
            {
                pageCtl.PageIndex = 1;
            }
            this.PageIndex = pageCtl.PageIndex;
            pageCtl.TotalPages = this.TotalPages;
            pageCtl.TotalCount = this.TotalCount;

            var list = query.Skip((pageCtl.PageIndex - 1) * pageCtl.PageSize).Take(pageCtl.PageSize).ToList();
            this.AddRange(list);
            this.ToList();
        }


        public PagedList(IEnumerable<T> source, PageCtl pageCtl)
        {
            if (pageCtl == null)
            {
                this.AddRange(source.ToList());
                this.ToList();
                return;
            }
            var query = source.AsQueryable();

            if (pageCtl.PageSize == 0) pageCtl.PageSize = 30;
            if (pageCtl.PageIndex == 0) pageCtl.PageIndex = 1;


            int total = source.Count();
            this.TotalCount = total;
            this.TotalPages = total / pageCtl.PageSize;

            if (total % pageCtl.PageSize > 0)
                TotalPages++;

            string OrderByField = pageCtl.OrderByField;
            bool Ascending = !pageCtl.Dscending;

            Type type = typeof(T);
            if (string.IsNullOrEmpty(OrderByField))
            {
                PropertyInfo[] propertys = type.GetProperties();
                if (propertys.Length == 0)
                    throw new Exception("当前实体未包含属性！");
                //OrderByField = propertys[0].Name;
                foreach (var item in propertys.Where(x => x.GetCustomAttributes(false).Count() > 0))
                {
                    if (item.CustomAttributes.Where(x => x.AttributeType == new System.ComponentModel.DataAnnotations.KeyAttribute().GetType()).FirstOrDefault() != null)
                    {
                        OrderByField = item.Name;
                        break;
                    }
                }
            }

            PropertyInfo property = type.GetProperty(OrderByField);
            if (property == null)
                throw new Exception("错误：\r\n当前实体中不存在指定的排序字段：" + OrderByField);

            ParameterExpression param = Expression.Parameter(type, "p");
            Expression propertyAccessExpression = Expression.MakeMemberAccess(param, property);
            LambdaExpression orderByExpression = Expression.Lambda(propertyAccessExpression, param);

            string methodName = Ascending ? "OrderBy" : "OrderByDescending";


            MethodCallExpression resultExp = Expression.Call(
                typeof(Queryable), methodName, new Type[] { type, property.PropertyType },
            query.Expression,
            Expression.Quote(orderByExpression)
            );

            query = query.Provider.CreateQuery<T>(resultExp);

            this.PageSize = pageCtl.PageSize;
            if (pageCtl.PageIndex > this.TotalPages)
            {
                pageCtl.PageIndex = this.TotalPages;
            }
            if (pageCtl.PageIndex < 1)
            {
                pageCtl.PageIndex = 1;
            }
            this.PageIndex = pageCtl.PageIndex;
            pageCtl.TotalPages = this.TotalPages;
            pageCtl.TotalCount = this.TotalCount;

            var list = query.Skip((pageCtl.PageIndex - 1) * pageCtl.PageSize).Take(pageCtl.PageSize).ToList();
            this.AddRange(list);
            this.ToList();
        }

        #region 属性

        /// <summary>
        /// 总页数
        /// </summary>
        public int TotalPages { get; set; }
        /// <summary>
        /// 总记录数
        /// </summary>
        public int TotalCount { get; set; }
        /// <summary>
        /// 当前页
        /// </summary>
        public int PageIndex { get; set; }
        /// <summary>
        /// 每页显示多少条记录
        /// </summary>
        public int PageSize { get; set; }
        /// <summary>
        /// 是否有上一页
        /// </summary>
        public bool IsPreviousPage { get { return (PageIndex > 0); } }
        /// <summary>
        /// 是否有下一页
        /// </summary>
        public bool IsNextPage { get { return (PageIndex * PageSize) <= TotalCount; } }


        #endregion


    }
}

```

4.接口

```java
 public interface IPagedList
    {
        /// <summary>
        /// 记录数
        /// </summary>
        int TotalCount { get; set; }
        /// <summary>
        /// 页数
        /// </summary>
        int TotalPages { get; set; }
        /// <summary>
        /// 当前页
        /// </summary>
        int PageIndex { get; set; }
        /// <summary>
        /// 页面大小
        /// </summary>
        int PageSize { get; set; }
        /// <summary>
        /// 是否上一页
        /// </summary>
        bool IsPreviousPage { get; }
        /// <summary>
        /// 是否下一页
        /// </summary>
        bool IsNextPage { get; }
    }
```



5.具体调用  (VB)

```java
 Dim page As New PageCtl With {.PageIndex = pageSize, .PageSize = pageNum, .OrderByField = "CityID"}'定义，赋页码，行数，排序字段
 res.datas = model.ToPagedList(page)'扩展方式调用
```

6.具体调用 (C# )  

```java
var page = new PageCtl{PageIndex = pageSize, PageSize = pageNum, OrderByField = "CityID"}
res.datas = model.ToPagedList(page)'扩展方式调用
```
