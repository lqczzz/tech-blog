---
title: 模版模式在golang的使用
date: 2018-10-27 18:05:57
tags: golang
---

## 从需求说起

还是timeline（微博／朋友圈etc）的业务场景。
timeline业务从步骤上考虑，无非就是几个步骤：
1. 获取不同的数据队列（通常是meta信息)
2. 根据一定的算法合并和截取数据
3. 根据合并后的meta数据获取详细的信息（content信息)
4. 返回


*如果第一步直接获取meta信息和content信息，则信息太大，tcp耗时和内存消耗也很多*

另外，timeline可能会分成很多类型：
1. 用户未登录看到的
2. 用户登录看到的
3. 推荐列表
4. etc

这种场景最适合用模版模式实现了
在java里面：

    public asbtract class BaseTimeline {
        public void Do() {
            this.doRetrieve()
            this.doMerge()
            this.doGetContent()
        }

        abstract void doRetrieve()
        abstract void doMerge()
        abstract void doGetContent()
    }

    class LoginedTimeline extends BaseTimeline {
        public void doRetrieve() {}
        public void doMerge() {}
        public void doGetContent() {}
    }

    class UnLoginedTimeline extends BaseTimeline {
        public void doRetrieve() {}
        public void doMerge() {}
        public void doGetContent() {}
    }


    // 场景类
    public class Server {
        private BaseTimeline timeline 
        public static void main([]string args) {
            if args[0] == "" {
                timeline = new LoginedTimeline()
            } else {
                timeline = new UnLoginedTimeline()
            }

            timeline.Do()
        }
    }

通过模版模式可以很方便的实现对timeline的扩展，新增不同的展示方式直接新增timeline类。在java servlet编程和图形界面开发(android view, html5 vue .etc)中是很常见的设计模式


可是golang的组合的方式没办法支持抽象方法

    type ITimeline interface {
        doRetrieve()   // 获取不同的队列
        doMerge()      // 合并
        doGetContent() // 获取详情
        Do()
    }

    type BaseTimeline struct{}

    func (bt *BaseTimeline) doRetrieve()   { fmt.Println("base retrieve") }
    func (bt *BaseTimeline) doMerge()      { fmt.Println("base merge") }
    func (bt *BaseTimeline) doGetContent() { fmt.Println("base content") }
    func (bt *BaseTimeline) Do() {
        bt.doRetrieve()
        bt.doMerge()
        bt.doGetContent()
    }

    type LoginedTimeline struct {
        BaseTimeline
    }

    func (bt *LoginedTimeline) doRetrieve()   { fmt.Println("LoginedTimeline retrieve") }
    func (bt *LoginedTimeline) doMerge()      { fmt.Println("LoginedTimeline merge") }
    func (bt *LoginedTimeline) doGetContent() { fmt.Println("LoginedTimeline content") }

    type UnLoginedTimeline struct {
        BaseTimeline
    }

    func (bt *UnLoginedTimeline) doRetrieve()   { fmt.Println("UnLoginedTimeline retrieve") }
    func (bt *UnLoginedTimeline) doMerge()      { fmt.Println("UnLoginedTimeline merge") }
    func (bt *UnLoginedTimeline) doGetContent() { fmt.Println("UnLoginedTimeline content") }

    // 使用
    type GetTimelineRequest struct {
        UserID uint64
    }

    func server(request, response interface{}) {
        // GetTimelineRequest is a struct
        var timeline ITimeline
        switch UserID := request.(*GetTimelineRequest).UserID; {
        case UserID != uint64(0):
            timeline = &LoginedTimeline{}
        default:
            timeline = &UnLoginedTimeline{}
        }

        timeline.Do()
    }

    func main() {
        req := &GetTimelineRequest{}
        server(req, nil)
    }

    // 输出：
    // base retrieve
    // base merge
    // base content


原因是因为组合方式不支持方法覆盖
可以把Do单独出来：

    type ITimeline interface {
        doRetrieve()   // 获取不同的队列
        doMerge()      // 合并
        doGetContent() // 获取详情
    }

    type BaseTimeline struct{}

    func (bt *BaseTimeline) doRetrieve()   { fmt.Println("base retrieve") }
    func (bt *BaseTimeline) doMerge()      { fmt.Println("base merge") }
    func (bt *BaseTimeline) doGetContent() { fmt.Println("base content") }
    
    // !!! 这里Do不再是具有接收者的方法了，调用方式也会不一样
    func Do(bt ITimeline) {
        bt.doRetrieve()
        bt.doMerge()
        bt.doGetContent()
    }

    type LoginedTimeline struct {
        BaseTimeline
    }

    func (bt *LoginedTimeline) doRetrieve()   { fmt.Println("LoginedTimeline retrieve") }
    func (bt *LoginedTimeline) doMerge()      { fmt.Println("LoginedTimeline merge") }
    func (bt *LoginedTimeline) doGetContent() { fmt.Println("LoginedTimeline content") }

    type UnLoginedTimeline struct {
        BaseTimeline
    }

    func (bt *UnLoginedTimeline) doRetrieve()   { fmt.Println("UnLoginedTimeline retrieve") }
    func (bt *UnLoginedTimeline) doMerge()      { fmt.Println("UnLoginedTimeline merge") }
    func (bt *UnLoginedTimeline) doGetContent() { fmt.Println("UnLoginedTimeline content") }

    // 使用
    type GetTimelineRequest struct {
        UserID uint64
    }

    func server(request, response interface{}) {
        // GetTimelineRequest is a struct
        var timeline ITimeline
        switch UserID := request.(*GetTimelineRequest).UserID; {
        case UserID != uint64(0):
            timeline = &LoginedTimeline{}
        default:
            timeline = &UnLoginedTimeline{}
        }

        // 这里的调用方式也就变化了
        Do(timeline)
    }

    func main() {
        req := &GetTimelineRequest{}
        server(req, nil)
    }