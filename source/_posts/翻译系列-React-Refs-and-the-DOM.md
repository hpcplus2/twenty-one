---
title: '翻译系列: React - Refs and the DOM'
date: 2018-04-09 21:33:40
categories:
  - Javascript
tags:
  - Javascript
  - React
  - React翻译系列
---

> 通过Refs，用户可以直接访问DOM节点或者React.render()创建的组件实例.

在典型的React数据流中(单向数据流)，`props`是父子组件交互的唯一途径。通过传递新的`props`, 可以修改子组件。
但是，在某些情况下可能需要在数据流之外去强制修改子组件。要修改的对象可能是DOM节点也可能是React组件实例。
对于这两种情况，React提供了相应的方法。

## When to Use Refs - 什么时候使用Refs

这里有几个很好的用例：

- 管理focus事件、文本选择或者媒体播放

- 触发命令式动画

- 集成第三方DOM库

如果可以通过声明式方式去解决，请避免使用refs。

举个例子: Dialog组件不要暴露open(), close()等方法。而是使用isOpen属性去控制。

## Don't Overuse Refs - 不要过度使用Refs

当你首先想到通过refs去做些什么的时候，请花点时间思考一下是否能在组件中使用state来达到相应的功能。
通常使用更高层次的state会使结构更清晰。([状态提升](https://reactjs.org/docs/lifting-state-up.html))

## Creating Refs - 创建Refs

Refs可以通过`React.createRef()`创建，同时也可以通过ref属性直接附加到React元素上。
在组件被构造的时候，refs会被赋予一个实例属性，以便他们可以在整个组件中被引用。

```
    class MyComponent extends React.Component {
      constructor(props) {
        super(props);
        this.myRef = React.createRef();
      }
      render() {
        return <div ref={this.myRef} />;
      }
    }
    
```

<!-- more -->

## Accessing Refs - 访问Refs

当ref被传递给render中的元素后，就可以通过ref的`current`属性去访问该节点的引用了。

```
    const node = this.myRef.current;

```

ref的值根据节点的类型决定:

- 当ref属性作用在HTML标签上，在构造函数中通过React.createRef()创建的ref接受基础的DOM元素作为current的值。

- 当ref属性作用在自定义的组件上，ref的current等于当前已挂载的组件实例。

- 不能在无状态组件上使用ref，因为他们没有实例。


下面的例子说明了不同之处：

### Adding a Ref to a DOM Element

这个例子用ref去存储DOM节点

```
    class CustomTextInput extends React.Component {
      constructor(props) {
        super(props);
        // create a ref to store the textInput DOM element
        this.textInput = React.createRef();
        this.focusTextInput = this.focusTextInput.bind(this);
      }

      focusTextInput() {
        // Explicitly focus the text input using the raw DOM API
        // Note: we're accessing "current" to get the DOM node
        this.textInput.current.focus();
      }

      render() {
        // tell React that we want the associate the <input> ref
        // with the `textInput` that we created in the constructor
        return (
          <div>
            <input
              type="text"
              ref={this.textInput} 
            />

            <input
              type="button"
              value="Focus the text input"
              onClick={this.focusTextInput}
            />
          </div>
        );
      }
    }

```

React会在组件装载后，将DOM节点元素赋予ref的`current`属性，并且在卸载后将其置空。
ref的更新发生在`componentDidMount`和`componentDidUpdate`之前

### Adding a Ref to a Class Component

如果想封装CustomTextInput来实现装载结束后立即点击一次的效果，我们可以通过ref去访问CustomTextInput并调用focusTextInput方法。

```
   class AutoFocusTextInput extends React.Component {
      constructor(props) {
        super(props);
        this.textInput = React.createRef();
      }

      componentDidMount() {
        this.textInput.current.focusTextInput();
      }

      render() {
        return (
          <CustomTextInput ref={this.textInput} />
        );
      }
    }

```

请注意，仅当CustomTextInput被声明为类的情况下才有效。


### Refs and Functional Components

*你不能在无状态组件上添加ref属性*，因为他们没有实例。

```
    function MyFunctionalComponent() {
      return <input />;
    }

    class Parent extends React.Component {
      constructor(props) {
        super(props);
        this.textInput = React.createRef();
      }
      render() {
        // This will *not* work!
        return (
          <MyFunctionalComponent ref={this.textInput} />
        );
      }
    }

```

当你需要ref的时候，你应该将组件转换成类组件，就像需要生命周期和state一样。

然而，我们可以在无状态组件内部使用ref来引用一个DOM节点或者类组件。

```
    function CustomTextInput(props) {
      // textInput must be declared here so the ref can refer to it
      let textInput = React.createRef();

      function handleClick() {
        textInput.current.focus();
      }

      return (
        <div>
          <input
            type="text"
            ref={textInput} />

          <input
            type="button"
            value="Focus the text input"
            onClick={handleClick}
          />
        </div>
      );
    }

```

## Exposing DOM Refs to Parent Components - 将Refs暴露给父组件

在某些情况下，你可能想通过父组件去访问子组件的DOM节点。
通常不鼓励这种行为，这样会破坏掉组件的封装性。
但是偶尔会用这种方法来触发焦点或者测量子节点的大小或者位置。

虽然你可以给子组件添加ref，但是这不是一个理想的方法，因为你获得的是子组件的实例而不是想要的DOM节点。
另外，对于无状态组件这样是无效的。

在这种情况下，我们推荐暴露子组件的特殊prop。这个prop可以不叫ref(不如InputRef)。子组件将该属性作为对应DOM节点的ref属性。
这样父组件就可以通过中间件来传递ref到子组件上。

这个方式使用类组件和无状态组件。

```
    function CustomTextInput(props) {
      return (
        <div>
          <input ref={props.inputRef} />
        </div>
      );
    }

    class Parent extends React.Component {
      constructor(props) {
        super(props);
        this.inputElement = React.createRef();
      }
      render() {
        return (
          <CustomTextInput inputRef={this.inputElement} />
        );
      }
    }

```

在上面的例子中，Parent传递自身属性`this.inputElement`到`CustomTextInput`的`inputRef`prop，
`CustomTextInput`将其作为ref属性赋予`input`。结果，Parent中的`this.inputElement.current`等同于Input的DOM节点。

注意`inputRef`在上面的例子中没有什么特殊的意义，仅仅只是一个component的prop。
然而，对于`<input>`的`ref`是很重要的，他告诉React附加ref到它的DOM节点上。

即使`CustomTextInput`是一个无状态组件，依然可以工作。不像ref只能作用在DOM节点和类组件上，对于像`inputRef`这样的组件属性没有限制。

这种模式的另一个好处是即使组件嵌套的层级很深依然有效。示例如下：

```
    function CustomTextInput(props) {
      return (
        <div>
          <input ref={props.inputRef} />
        </div>
      );
    }

    function Parent(props) {
      return (
        <div>
          My input: <CustomTextInput inputRef={props.inputRef} />
        </div>
      );
    }

    class Grandparent extends React.Component {
      constructor(props) {
        super(props);
        this.inputElement = React.createRef();
      }
      render() {
        return (
          <Parent inputRef={this.inputElement} />
        );
      }
    }

```

参照上面的例子，`GrandParent`指定了`this.inputElement`。
然后通过`inputRef`这个prop传递给`Parent`，然后`Parent`将其通过`inputRef`传递给`CustomTextInput`。
最终，`CustomTextInput`将`inputRef`加到`<input>`的`ref`属性上。
结果就是`GrandParent`的`this.inputElement.current`等于`CustomTextInput`的`<input>`。

如果可能，我们不推荐暴露DOM节点，但是这的确是一种有用的解决方法。
注意，这种方式需要你添加代码到子组件上。如果你无法控制子组件，你可以通过`findDOMNode()`，但是不推荐这样做。 


## Callback Refs - Refs回调

React支持另一种方式去设置refs，refs回调，这给refs的设置与取消设置提供了更好的控制。

通过function来代替createRef()，将其传递给`ref`属性。
function接受HTML的DOM节点或者React组件实例作为参数，这样就可以在其他地方存储或者访问他。

下面的例子实现了一种普通的模式: 使用`ref`回调去存储DOM节点的引用。

```
    class CustomTextInput extends React.Component {
      constructor(props) {
        super(props);

        this.textInput = null;

        this.setTextInputRef = element => {
          this.textInput = element;
        };

        this.focusTextInput = () => {
          // Focus the text input using the raw DOM API
          if (this.textInput) this.textInput.focus();
        };
      }

      componentDidMount() {
        // autofocus the input on mount
        this.focusTextInput();
      }

      render() {
        // Use the `ref` callback to store a reference to the text input DOM
        // element in an instance field (for example, this.textInput).
        return (
          <div>
            <input
              type="text"
              ref={this.setTextInputRef}
            />
            <input
              type="button"
              value="Focus the text input"
              onClick={this.focusTextInput}
            />
          </div>
        );
      }
    }

```

React会在组件挂载时调用`ref`回调，同时当组件卸载的时候，`ref`会变成null。
`ref`回调在`componentDidMount`或者`componentDidUpdate`之前调用。

你可以在组件间传递ref回调，就像上面传递React.createRef()创建的refs对象一样。

```
    function CustomTextInput(props) {
      return (
        <div>
          <input ref={props.inputRef} />
        </div>
      );
    }

    class Parent extends React.Component {
      render() {
        return (
          <CustomTextInput
            inputRef={el => this.inputElement = el}
          />
        );
      }
    }

```

在上面的例子中，`Parent`通过`inputRef`属性传递`ref`回调给`CustomTextInput`，
然后`CustomTextInput`将inputRef赋予`<input>`的ref属性。
这样，`Parent`的`this.inputElement`就设置成`CustomTextInput`中`<input>`元素的DOM节点了。

## Legacy API: String Refs - 旧版API：refs字符串

如果你之前用过React, 你可能熟悉旧的API中字符串类型的`ref`属性，像`textInput`，你可以通过`this.refs.textInput`访问DOM节点。
我们不建议用这种方式，因为它存在很多问题，同时可能在未来的版本中移除。

> Note

> 如果你要使用`this.refs.textInput`去访问refs，我们推荐你使用`回调模式`或者`createRef API`。


## Caveats with callback refs - refs回调注意事项

如果ref回调函数被定义为内联函数，在更新期间会被调用两次。
第一次的参数是null，然后才是DOM元素。
这是因为每次重新渲染都会产生一个新的函数实例，所以React会清理旧的，并生成新的ref。
你可以通过将他设置为类的绑定方法来避免这种情况，但是在大多数情况下，他不应该存在。


> 文章原文：
> 
> [React官网 - Refs-And-The-DOM](https://reactjs.org/docs/refs-and-the-dom.html)
