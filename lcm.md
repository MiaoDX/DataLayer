# Notes on LCM

### Set subscribe queue size to 1

First things first, when using LCM to subscribe messages, great chances are that the subscribe routine is slower than data source generator.

If producer emit every 20ms and consumer eats every 100ms, we will definitely have some lag here, and the [default receiver queue size is 30](https://github.com/lcm-proj/lcm/blob/v1.4.0/NEWS#L392) as [shown in code](https://github.com/lcm-proj/lcm/blob/v1.4.0/lcm/lcm.c#L135). So, the queue will be full quickly, we will have:

`30x100ms=3s` lag always, that's surely suboptimal as most of the time, we want the latest message and do not care too much about the possible messages lost (which by the way, even with 30 queue size data drop will also happen), so, set the queue size to 1 suits well:

``` cpp
auto sub = lcm.subscribe(channel, &Handler::handleMessage, &recv_obj);                                                                                                            
// sub->setQueueCapacity(0); // 0 means keeps all msgs, should not do this  
sub->setQueueCapacity(1); // keeps latest, nice choice for pseudo real-time applications
```
Also note that capacity 0 will try to keeps all the incomming messages and increase the message queue, please do not that.

See [lcm_demo/buffer_demo.cpp](https://github.com/MiaoDX/code_snippets/blob/master/lcm_demo/buffer_demo.cpp) for the working demo.

### Add debug info to lcm messages

When define LCM message, one extra item to specify the version can be used to ease debugging.
