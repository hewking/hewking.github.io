---
layout: post
title:  "EventBus事件总线"
crawlertitle: "EventBus事件总线"
summary: "EventBus事件总线"
date:   2016-07-12 23:09:47 +0700
categories: posts
tags: 'Android源码分析'
author: hewking
---
> Android 常用事件传递有：BroadcastReceiver ,Interface回调，Handler,事件总线，这几种方式各有使用场景及优点和不足。本篇分析EventBus


### 基本使用

```
public void onCreate(){
    EventBus bus = EventBus.getDefault();
    bus.register(this);
    bus.post("post message");
}

@Subscribe
public void onEvent(String message){
        Log.i("TAG","receive message");
}

public void onDestory(){
        EventBus.unregister(this);
}

```
从以上示例可知基本使用主要有四个步骤
- 获取EventBus 实例
- 订阅 register
- 发送消息 post
- 取消订阅

所以以下分析也是从四个步骤开始，先大致分析整体结构。

```
    public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
```
从getDefault方法看，是懒汉式单例模式。主要代码给defaultInstance 赋值对象，进入到EventBus构造方法看

```
   public EventBus() {
        this(DEFAULT_BUILDER);
    }

    EventBus(EventBusBuilder builder) {
        subscriptionsByEventType = new HashMap<>();
        typesBySubscriber = new HashMap<>();
        stickyEvents = new ConcurrentHashMap<>();
        mainThreadPoster = new HandlerPoster(this, Looper.getMainLooper(), 10);
        backgroundPoster = new BackgroundPoster(this);
        asyncPoster = new AsyncPoster(this);
        indexCount = builder.subscriberInfoIndexes != null ? builder.subscriberInfoIndexes.size() : 0;
        subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
                builder.strictMethodVerification, builder.ignoreGeneratedIndex);
        logSubscriberExceptions = builder.logSubscriberExceptions;
        logNoSubscriberMessages = builder.logNoSubscriberMessages;
        sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
        sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
        throwSubscriberException = builder.throwSubscriberException;
        eventInheritance = builder.eventInheritance;
        executorService = builder.executorService;
    }

```
在构造方法中，对EventBus 各个成员赋值。通过创建者模式。包括两个重要得Map
- subscriptionsByEventType
- typesBySubscriber

前者通过eventType 保存所订阅对象与订阅方法组成得对象newSubscription，后者保存所订阅对象与所有得已订阅方法。这两个重要的 map 再post 方法调用，有重要作用。

### 订阅

```
    public void register(Object subscriber) {
        Class<?> subscriberClass = subscriber.getClass();
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }

 // Must be called in synchronized block
    private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
        Class<?> eventType = subscriberMethod.eventType;
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }

        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        if (subscriberMethod.sticky) {
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }
```
这段代码有点长，其实主要就做了两件事，更新了两个最重要的Map：subscriptionsByEventType和typesBySubscriber，已备我们在后面post事件时使用。其中，前者保存了已eventType为键的所有相关的subscription。

整体流程图如下：
![eventbus_register.png](http://upload-images.jianshu.io/upload_images/1394860-d427ec93d34eb5f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 发送事件

```
 /** Posts the given event to the event bus. */
    public void post(Object event) {
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);

        if (!postingState.isPosting) {
            postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                while (!eventQueue.isEmpty()) {
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
```
首先在发送事件所在的线程创建一个PostingThreadState对象，这个对象有一个事件队列，将新的事件添加到队列中，然后开始分发并将发送过的事件移除队列，直到将队列中所有的事件发送完毕。

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
        if (eventInheritance) {
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                Log.d(TAG, "No subscribers registered for event " + eventClass);
            }
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }

private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
        CopyOnWriteArrayList<Subscription> subscriptions;
        synchronized (this) {
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
        if (subscriptions != null && !subscriptions.isEmpty()) {
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted = false;
                try {
                    postToSubscription(subscription, event, postingState.isMainThread);
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                if (aborted) {
                    break;
                }
            }
            return true;
        }
        return false;
    }

```
在这里，终于将发送的时间和subscriber联系了起来，根据发送事件的类型找到订阅了这个事件的订阅列表，通知列表中每个订阅者：


```
   private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }

    void invokeSubscriber(Subscription subscription, Object event) {
        try {
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
        } catch (InvocationTargetException e) {
            handleSubscriberException(subscription, event, e.getCause());
        } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
        }
    }
```
最后，看到了subscriber的subscriberMethod最终通过反射的方式被执行，整个事件分发的过程也就到此结束了。

![eventbus_post.png](http://upload-images.jianshu.io/upload_images/1394860-a16b0a038b1e008b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)\

### 取消订阅
通过typesBySubscriber这个HashMap找到订阅者订阅的所有事件，然后在subscriptionsByEventType的各个事件订阅列表中将这个订阅者对应的订阅事件移除，最后再将订阅者从typesBySubscriber移除。

### 总结
到这里，简单的EventBus事件流就算解释清楚了，主要就是两个方法，一个是register，一个是post，分别用于注册保存subscriber和消息通知subscriber， 在注册的过程中，通过subscriberMethodFinder将subscriber中所有的订阅方法都找出来，并根据两种不同的规则分别按照eventType和subscriber将订阅方法缓存在Map中，以供后续的步骤使用；在post事件中，是以线程为单位的，将事件放入当前线程的消息队列中，然后依次循环取出发送，直至队列为空，按照事件的类型找到当前事件类型相关的Subscription列表，如果当前bus还支持事件继承的话，还会找到当前事件类型的父类和父类接口对应的Subscription列表，找到这样的订阅列表之后，顺序取出每一个Subscription，然后通过反射调用subscriber的订阅方法，至此，整个事件的传递就结束了。

以下为类图：
![eventbus_class.png](http://upload-images.jianshu.io/upload_images/1394860-7cd8048c0b9e43bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
依照这个图中，再回忆我们分析的整个过程:首先EventBus的实例都是依据EventBusBuilder创建的，我们可以自定义EventBusBuilder来达到定制EventBus的目的，SubscriberMethodFinder是用于找出订阅者中的所有的订阅方法，然后订阅方法和订阅者组成一个叫做Subscription的订阅事件，EventBus保存的就是Subscription的列表




