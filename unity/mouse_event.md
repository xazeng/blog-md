---
title: unity 鼠标事件
categories: unity
date: 2019-08-01 15:59:55
tags: [unity, ide]
---

刚开始学习 unity，鼠标事件处理或是入门会遇到的一个难点。其实说难不难，只是刚开始学，很多概念没有，所以比较容易疑惑。
<!--more-->

## 方法一： Input.GetMouseButton

每一帧主动去获取鼠标状态，可以在 Update 函数中也可以在 Coroutine 中。

这个方法比较底层，没有其他的要求，但是各种状态需要自己去判断，比如如何在一次点击中只触发一次等等。

跟脚本绑定在哪个 GameObject 上无关，需要判断是否直接在绑定对象上触发会用到 EventSystem.current.IsPointerOverGameObject() 。

## 方法二：Event System

这个方法最常用，详细内容可以查看 <https://docs.unity3d.com/Manual/EventSystem.html>。

这个方法需要配合 Raycaster 使用。
关于 UGUI 下，Canvas 创建时默认就有带 Graphic Raycaster。
2D 下需要给 Camera 绑定 Physics 2D Raycaster。
3D 下则是绑定 Physics Raycaster。

除了 UGUI ，如果其他 GameObject 想要触发事件就必须绑定 Collider 组件。

事件不会穿透，比如被 UGUI 触发了就不会传递给下面的 GameObject，GameObject 之间同理。
但是 UGUI 在 Inspector 面板中有个叫 Raycast Target 的属性，如果勾掉，则表示不触发事件，这时事件会穿过 UGUI 传递给下面的对象。

unity 定义了很多事件处理接口，比如 IPointerDownHandler, IBeginDragHandler 等等，实现这些接口就可以处理相应事件。
还可以用 EventTrigger ，这个类实现了 unity 自带的 Event System 中的所有的事件接口，可以直接在编辑器中做关联。但是需要注意，关联的处理函数必须带一个 BaseEventData 的参数，还有就是不要和 OnMouseDown 之类的函数同名。

## 方法三：OnMouseXXX

继承自 MonoBehaviour 的类，都可以定义这些函数来处理事件。

UGUI 对象不会触发，想要触发的 GameObject 也必须绑定 Collider 组件。如果 GameObject 在 UGUI 下方，也会被触发。

这些函数并非是重写了某个接口，而是 unity 框架通过放射进行调用，所以参数不对的情况只会在运行期抛出异常。