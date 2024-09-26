# iOS-Interview for Junior Enginee
1. 精通OC
2. 精通UIKit等Cocoa Framework 自定义UI控件
3. 熟悉网络通信、常用的数据传输协议的理解 数据格式的解析 HTTP HTTPS 对称加密算法、非对称 数据格式json和xml如何解析
4. 主流开源框架的使用经验

# OC语言
1. 分类：什么是分类，分类的实现机制、原理，能否为分类添加实例变量
2. 关联对象
3. 扩展：分类和扩展的区别
4. 代理：（针对初级工程师）
5. 通知：实现机制和原理 NS开头 考察对逻辑的理解
6. KVO：实现机制 
7. KVC：原理
8. 属性关键字：weak和assign的区别 如何使用copy关键字
   
## 分类 Category
Q1: 你用分类做了哪些事？

A：声明私有方法，分解体积庞大的类文件，把Framework的私有方法公开。。。

Q2: 分类的特点（和扩展的区别）

A：运行时决议（在编写分类文件之后，并没有把分类中对应添加的内容附加到相应的宿主类上，而是在运行时通过runtime把分类中添加的内容添加在对应的宿主类上），这是和扩展最大的区别；可以为系统类添加分类

Q3: 分类中都可以添加哪些内容？

A：实例方法；类方法；协议；属性。（分类不能添加实例变量，通过关联对象才可以添加实例变量）

from https://opensource.apple.com/source/objc4/objc4-680/runtime/objc-runtime-new.h

struct category_t {

    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    
    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }
    property_list_t *propertiesForMeta(bool isMeta) {
        if (isMeta) return nil; // classProperties;
        else return instanceProperties;
    }
};


Q4: 加载调用栈

_objc_init -> map_2_images（镜像不是图片） -> map_images_nolock -> _read_images -> remethodizeClass

Q5: 源码分析

from https://opensource.apple.com/source/objc4/objc4-723/runtime/objc-runtime-new.mm.auto.html

static void remethodizeClass(Class cls)

{
    category_list *cats;
    bool isMeta;

    runtimeLock.assertWriting();
/* 我们只分析分类中实例方法添加的逻辑，因此在这里我们假设 isMeta == NO，代表实例方法（否则元类方法） */
    isMeta = cls->isMetaClass();

    // Re-methodizing: check for more categories
    // 获取cls中未完成整合的所有分类
    if ((cats = unattachedCategoriesForClass(cls, false/*not realizing*/))) {
        if (PrintConnecting) {
            _objc_inform("CLASS: attaching categories to class '%s' %s", 
                         cls->nameForLogging(), isMeta ? "(meta)" : "");
        }
        
        attachCategories(cls, cats, true /*flush caches*/);        
        free(cats);
    }
}


// Attach method lists and properties and protocols from categories to a class.
// Assumes the categories in cats are all loaded and sorted by load order, 
// oldest categories first.

static void attachCategories(Class cls, category_list *cats, bool flush_caches)

{
    if (!cats) return;
    
    /* if (PrintReplacedMethods) printReplacements(cls, cats);*/

    bool isMeta = cls->isMetaClass();

    // fixme rearrange to remove these intermediate allocations
    method_list_t **mlists = (method_list_t **) /*方法列表*/
        malloc(cats->count * sizeof(*mlists));
    property_list_t **proplists = (property_list_t **) /*属性列表*/
        malloc(cats->count * sizeof(*proplists));
    protocol_list_t **protolists = (protocol_list_t **) /*协议列表*/
        malloc(cats->count * sizeof(*protolists));

    // Count backwards through cats to get newest categories first
    int mcount = 0;
    int propcount = 0;
    int protocount = 0;
    int i = cats->count;
    bool fromBundle = NO;
    while (i--) {
        auto& entry = cats->list[i];

        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            mlists[mcount++] = mlist;
            fromBundle |= entry.hi->isBundle();
        }

        property_list_t *proplist = 
            entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            proplists[propcount++] = proplist;
        }

        protocol_list_t *protolist = entry.cat->protocols;
        if (protolist) {
            protolists[protocount++] = protolist;
        }
    }

    auto rw = cls->data();

    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
    rw->methods.attachLists(mlists, mcount);
    free(mlists);
    if (flush_caches  &&  mcount > 0) flushCaches(cls);

    rw->properties.attachLists(proplists, propcount);
    free(proplists);

    rw->protocols.attachLists(protolists, protocount);
    free(protolists);
}




# UI视图 
## UITableView
1. 重用机制
   
   cell = [tableView dequeueReusableCellWithIdentifier:identifier]

   自定义UI控件：字母索引条
   
3. 数据源同步

## 事件传递 视图响应 *****
## 图像显示原理
## 卡顿 掉帧 *****
## 绘制原理 异步绘制
## 离屏渲染



