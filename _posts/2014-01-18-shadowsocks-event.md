---
layout: post
title:  "记开发ruby版shadowsocks的一个bug"
date:   2014-01-18 17:55:54
categories: shadowsocks
---

ruby版shadowsocks的开发过程其实是蛮快乐的，本来开发这个工具的目的就是just for fun，这也是我编程的初衷。

这个工具使用了eventmachine，简称EM。EM底层是C++，说实话，我个人并不是很喜欢这样的方式，因为对于debug来说并不是很友好。之后如果得空，考虑改写为线程的方式。

在最初的版本使用稳定几天后，我在用tsocks去下载一个apple tailers的预告片时，遇到了一个服务器内存爆掉的问题。

想了一下，其实很简单，原因是服务器的带宽比我下载的带宽大很多。怎么说呢？EM是unblock的，所有的请求照单全收。所有下载的请求全部接收到服务器，然后服务器再发送回本地。发送回本地的带宽将根据本地的带宽决定，如果服务器的带宽远大于本地带宽的时候，所有未来得及发送的数据全部缓存起来。所以如果这时候下载一个大文件时，很大一部分会被cache起来，结果导致内存爆掉。为了找这个问题的解答，我还特意跑去EM的mailing list问，得到的答案为使用pause的api。然后，我就有了以下的代码:

{% highlight ruby %}
class Connection < EventMachine::Connection
  BackpressureLevel = 524288 # 512k

  private

  def over_pressure?
    remote.get_outbound_data_size > BackpressureLevel
  end

  def outbound_scheduler
    if over_pressure?
      pause unless paused?
      EM.add_timer(0.2) { outbound_scheduler }
    else
      resume if paused?
    end
  end
end
{% endhighlight %}

这里代码的意思为，我设置了一个压力值，是512k，如果发送的缓存超过512k的时候，将停止接收新数据。
bug算是解决了，但是我觉得并不是很优雅。考虑之后重写一下，用erlang的方式。

erlang的方式又是什么样呢？

erlang的`gen_tcp`中使用了`{active}`参数，`active`为false时，为阻塞的方式工作。`active`为true时，将类似eventmachine的方式。但是erlang还提供了一个`once`的参数。即只接收一次数据，然后阻塞起来，之后再用setopts去打开，这种方式成为半阻塞。我在考虑，EM用pause去实现这种方式应该也没问题，而且比用timer去检查更为合理和高效。考虑之后得空后实现一个半阻塞的版本。

以上代码可以从我改的shadowsocks repo中找到: <https://github.com/Sen/shadowsocks-ruby>，欢迎指正。
