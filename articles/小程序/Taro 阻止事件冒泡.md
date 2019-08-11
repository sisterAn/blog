在 Taro 中另一个不同是你不能使用 `catchEvent` 的方式阻止事件冒泡。你必须明确的使用 `stopPropagation`。例如，阻止事件冒泡你可以这样写：

```js
class Toggle extends Component {
  constructor (props) {
    super(props)
    this.state = {isToggleOn: true}
  }

  onClick = (e) => {
    e.stopPropagation() // 阻止事件冒泡
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }))
  }

  render () {
    return (
      <button onClick={this.onClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    )
  }
}
```
