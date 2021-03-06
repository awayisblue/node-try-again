# node-just-retry
通过node-just-retry来配置重试策略，管理需要重试的程序。

# 安装
`yarn add just-retry`

# 使用
```
const Retry = require('just-retry')

let retry = new Retry({
  delays: ['1000', '2000x2', '1000x~'],
  steadyTime: 3000
})
retry.on('all', function (event, data) {
  console.log(event)
})
function runner (retry) {
  retry.success()
  console.log('runner')
  setTimeout(() => retry.error(), 1000)
}
retry.monitor(runner)

```
上述代码中，使用just-retry,来管理runner里的代码。当代码执行了retry.error()时，just-retry会根据配置的delay进行重试。基本流程为：
1. 程序运行success(), 说明已经成功。
2. 间隔1000ms后，程序执行了error(), 说明出现了错误。
3. just-retry根据delays进行错误重试，第一次错误，间隔1000ms重试，第2次及第3次错误，间隔2000ms重试，第4次及以后，都只间隔1000进行重试。
4. steadyTime代表代码稳定下来的时间，当程序执行了success后，假如经过了steadyTime，还没有出现错误，就会对错误的状态进行重置，即下一次错误就变成第一次错误了。

# 配置说明
### delays
每次重试的时间间隔。支持的格式为`/^\d+(x[\d+|~])?$/`, 下面是例子：
1. 1000, 说明以下1次重试，间隔1秒，跟1000x1相同
2. 2000x2, 说明以下2次重试，都是间隔2秒，
3. 1000x~, 说明以下所有重试，都是间隔1秒

假如delays里面，没有一个配置是包含以下所有重试的，那么在just-retry走完所有delay后，会emit一个fail事件，可以通过监听该事件来做错误处理。

### steadyTime
代码稳定的时间，代码执行成功时，需要调用success()函数，来通知just-retry调用成功。这时，假如代码在steadyTime的时间间隔内，都没有调用error(), 则just-retry会重置错误状态，错误的次数被重置为0。
steadyTime默认为5000

# api
## monitor(runner)
开始监控代码。执行后，runner代码会被执行。runner需要是一个函数。

## succuss()
成功，需要在代码执行成功的时候进行调用。

## error(err)
出错，需要在代码出错的时间进行调用，调用后，just-retry会进行重试。
## stop()
暂停重试功能，调用后，just-retry在往后的error里，不会再进行重试。
## resume()
启用重试功能，调用后，假如在stop期间，有出现过错误，会进行重试。

## on(event, callback(event, data))
事件监听，支持的事件有：
### on('all', callback(event, data))
所有的事件都能监听到, 通过判断event来区分不同的事件。
### on('reset', callback(event))
重置事件，当系统在steadyTime内都没有出错时，则会进行reset

### on('error', callback(event, err))
出错事件，当runner调用error(err)时，会触发该事件。

### on('success', callback(event))
成功事件，当runner调用success()时，会触发该事件。

### on('retry', callback(event))
重试事件，当just-retry进行重试时，会触发该事件。

### on('fail', callback(event)) 
失败事件，当just-retry跑完所有delay后，若还是出错，会触发该事件。
