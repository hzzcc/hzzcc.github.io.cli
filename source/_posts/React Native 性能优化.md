---
title: React Native 性能优化
date: 2016-06-04 20:24:33
tags: ["React Native"]

---

React native 是facebook提供的可媲美原生应用的js框架，但是做一些性能优化还是有必要的。参考官方文档，列出了以下优化点。

<!-- more -->

<!-- toc -->

### 1. Navigator页面切换动画优化

使用InteractionManager，navigator切换完成会触发使用InteractionManager.runAfterInteractions的回调函数，你的组件可以写成这样：

    class ExpensiveScene extends React.Component {
      constructor(props, context) {
        super(props, context);
        this.state = {renderPlaceholderOnly: true};
      }

      componentDidMount() {
        InteractionManager.runAfterInteractions(() => {
          this.setState({renderPlaceholderOnly: false});
        });
      }

      render() {
        if (this.state.renderPlaceholderOnly) {
          return this._renderPlaceholderView();
        }

        return (
          <View>
            <Text>Your full view goes here</Text>
          </View>
        );
      }


      _renderPlaceholderView() {
        return (
          <View>
            <Text>Loading...</Text>
          </View>
        );
      }
    };

### 2. LayoutAnimation 布局动画

在你使用setState来改变页面布局时，不妨使用LayoutAnimation.easeInEaseOut()，来增加动画效果，使页面改变的更平滑。LayoutAnimation 还有提供其他的动画效果，也可以自己配制。

	LayoutAnimation.easeInEaseOut();
    this.setState({width: 120});

当你在js线程上做了大量的事情时，最通常的表现是"页面切换动画缓慢"。用InteractionManager是个很好的方式，但是如果来在动画过程中延迟其他动作，而使用户体验代价过高，你就可能会考虑到LayoutAnimation。
当前Animated api 会根据需求在js线程上计算每一帧画面，然而LayoutAnimation是利用核心动画，不会被js线程和主线程丢帧的影响。
**警告** LayoutAnimation 不能被打断。如果你想阻止动画，你需要使用Animated

### 3. shouldComponentUpdate 减少页面渲染次数

可以使用shouldComponentUpdate生命周期方法，此方法作用是在props或 者state改变且接收到新的值时，在render方法之前调用。此方法在第一次渲染的时候不会调用，在使用forceUpdate方法的时候也不会。所以在这个方法中我们可以增加些判断规则来避免当state或者props没有改变时所造成的重新render。

    shouldComponentUpdate(nextProps, nextState) {
    	return nextProps.value != this.state.value;
    }

但是如果state或props是对象时，shouldComponentUpdate一直返回true, 导致还是会执行render。所以必须对对象所有的键值进行比较才能确认是否相等。这时推荐使用immutablejs。

### 4. ListView数据加载的优化

- initialListSize
这个属性会规定在第一次render时有多少行单元需要render，如果我们想尽快显示数据，我们可以设置initialListSize为1，我们就可以很快的看到其他的行会依次填充。而一屏界面有多少行取决于pageSize属性

- pageSize
ListView 根据 pageSize 决定每屏显示多少行，默认值为1

- scrollRenderAheadDistance
在行单元显示出来时多早之前开始渲染。
如果我们有2000行单元，如果把它们都立即渲染出来会消耗大量内存和运算资源。也有可能导致非常糟糕的丢帧。所以scrollRenderAheadDistance允许我们指定在当前窗口需要继续渲染多远的行单元。

- removeClippedSubviews
当值为true时，不在界面显示的视图（那些overflow为true的）都被原生的父级组件移除了。这可以改善长列表的滚动性能。默认值是true。（在 0.14-rc之前的版本上是false）。
这是在大列表上的非常大的优化点。android上overflow属性的值一直是hidden，所以你不用担心去设值。但是在iOS上你需要保证你给每行的容器设置了overflow: hidden。

- rowHasChanged
你必须提供rowHasChanged来决定每一行是否需要重新render。如果你使用了immutable 数据结构，直接使用全等检查就可以了。

### 5. 移动view（滚动，转换，旋转）导致FPS下降

你可以使用shouldRasterizeIOS或renderToHardwareTextureAndroid来显著优化渲染。但是不能过度使用，否则你的内存会消耗殆尽。使用这些属性时，记录下性能和内存使用情况。如果你不打算再移动视图，请关闭此属性。

### 6. 点按视图并不能很快的反应

有时候，我们在点击的响应里调整透明度或高亮组件，如果我们在同一时候做了某个动作，我们在onPress返回前是不会看到任何变化的。onPress中setState可能会导致大量计算和少量帧丢失。一个解决办法是在你的onPress处理包裹在requestAnimationFrame中。

    handleOnPress() {
      // Always use TimerMixin with requestAnimationFrame, setTimeout and
      // setInterval
      this.requestAnimationFrame(() => {
        this.doExpensiveAction();
      });
    }

